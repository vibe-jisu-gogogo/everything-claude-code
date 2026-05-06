# Plan - 多模型协同计划

多模型协同计划 - 上下文获取 + 双模型分析 → 生成逐步实施计划。

$ARGUMENTS

---

## 核心协议

- **语言协议**: 与工具/模型交互时使用**英语**，与用户通信时使用用户的语言
- **强制并行**: Codex/Gemini 调用必须使用 `run_in_background: true`（包括单模型调用，以避免阻塞主线程）
- **代码主权**: 外部模型**完全没有文件系统写入权限**，所有变更由 Claude 执行
- **损失限制机制**: 验证当前阶段输出后再进入下一阶段
- **仅计划模式**: 此命令允许读取上下文并写入 `.claude/plan/*` 计划文件，但**不修改生产代码**

---

## 多模型调用规范

**调用语法**（并行：使用 `run_in_background: true`）:

```
Bash({
  command: "~/.claude/bin/codeagent-wrapper {{LITE_MODE_FLAG}}--backend <codex|gemini> {{GEMINI_MODEL_FLAG}}- \"$PWD\" <<'EOF'
ROLE_FILE: <角色提示词路径>
<TASK>
Requirement: <增强后的需求>
Context: <获取的项目上下文>
</TASK>
OUTPUT: 包含伪代码的逐步实施计划。不修改文件。
EOF",
  run_in_background: true,
  timeout: 3600000,
  description: "简洁描述"
})
```

**模型参数注意事项**:
- `{{GEMINI_MODEL_FLAG}}`: 使用 `--backend gemini` 时，替换为 `--gemini-model gemini-3-pro-preview`（注意末尾空格）；使用 codex 时使用空字符串

**角色提示词**:

| 阶段 | Codex | Gemini |
|-------|-------|--------|
| 分析 | `~/.claude/.ccg/prompts/codex/analyzer.md` | `~/.claude/.ccg/prompts/gemini/analyzer.md` |
| 计划 | `~/.claude/.ccg/prompts/codex/architect.md` | `~/.claude/.ccg/prompts/gemini/architect.md` |

**会话重用**: 每次调用会返回 `SESSION_ID: xxx`（通常由包装器输出），**必须保存**以供后续 `/ccg:execute` 使用。

**等待后台任务**（最大超时 600000ms = 10 分钟）:

```
TaskOutput({ task_id: "<task_id>", block: true, timeout: 600000 })
```

**重要**:
- 必须指定 `timeout: 600000`。否则默认 30 秒会导致提前超时
- 10 分钟后仍未完成时，继续使用 `TaskOutput` 轮询，**不要强制终止进程**
- 因超时跳过等待时，**必须调用 `AskUserQuestion` 询问用户是继续等待还是强制终止任务**

---

## 执行工作流程

**计划任务**: $ARGUMENTS

### 阶段 1: 完整上下文获取

`[Mode: Research]`

#### 1.1 提示词增强（必须首先执行）

**ace-tool MCP 可用时**，调用 `mcp__ace-tool__enhance_prompt` 工具:

```
mcp__ace-tool__enhance_prompt({
  prompt: "$ARGUMENTS",
  conversation_history: "<最近5-10轮对话>",
  project_root_path: "$PWD"
})
```

等待增强后的提示词，**将原始 $ARGUMENTS 替换为增强结果，用于所有后续阶段**。

**ace-tool MCP 不可用时**: 跳过此步骤，在所有后续阶段中直接使用原始 `$ARGUMENTS`。

#### 1.2 上下文获取

**ace-tool MCP 可用时**，调用 `mcp__ace-tool__search_context` 工具:

```
mcp__ace-tool__search_context({
  query: "<基于增强需求的语义查询>",
  project_root_path: "$PWD"
})
```

- 使用自然语言构建语义查询（Where/What/How）
- **不基于假设回答**

**ace-tool MCP 不可用时**，使用 Claude Code 内置工具回退:
1. **Glob**: 按模式搜索相关文件（例如: `Glob("**/*.ts")`, `Glob("src/**/*.py")`）
2. **Grep**: 搜索关键符号、函数名、类定义（例如: `Grep("className|functionName")`）
3. **Read**: 读取发现的文件，收集完整上下文
4. **Task (Explore 代理)**: 需要深度探索时，使用 `subagent_type: "Explore"` 的 `Task`

#### 1.3 完整性检查

- 必须获取相关类、函数、变量的**完整定义和签名**
- 上下文不足时触发**递归获取**
- 优先输出：入口文件 + 行号 + 关键符号名；仅在解决歧义需要时添加最少量代码片段

#### 1.4 需求一致性

- 需求仍有歧义时，**务必**向用户输出引导性问题
- 直到需求边界清晰为止（无缺失、无冗余）

### 阶段 2: 多模型协同分析

`[Mode: Analysis]`

#### 2.1 输入分配

**并行调用 Codex 和 Gemini**（`run_in_background: true`）:

将**原始需求**（无预设观点）分配给两个模型:

1. **Codex 后端分析**:
   - ROLE_FILE: `~/.claude/.ccg/prompts/codex/analyzer.md`
   - 重点：技术可行性、架构影响、性能考虑、潜在风险
   - OUTPUT: 多角度解决方案 + 优缺点分析

2. **Gemini 前端分析**:
   - ROLE_FILE: `~/.claude/.ccg/prompts/gemini/analyzer.md`
   - 重点：UI/UX 影响、用户体验、视觉设计
   - OUTPUT: 多角度解决方案 + 优缺点分析

使用 `TaskOutput` 等待两个模型的完整结果。保存 **SESSION_ID**（`CODEX_SESSION` 和 `GEMINI_SESSION`）。

#### 2.2 交叉验证

整合观点，迭代优化:

1. **识别共识**（强信号）
2. **识别差异**（需要加权）
3. **互补优势**: 后端逻辑遵循 Codex，前端设计遵循 Gemini
4. **逻辑推理**: 消除解决方案中的逻辑差距

#### 2.3（可选但推荐）双模型计划草稿

为减少 Claude 整合计划中的遗漏风险，可以让两个模型并行输出"计划草稿"（但**不允许**修改文件）:

1. **Codex 计划草稿**（后端权威）:
   - ROLE_FILE: `~/.claude/.ccg/prompts/codex/architect.md`
   - OUTPUT: 逐步计划 + 伪代码（重点：数据流/边界情况/错误处理/测试策略）

2. **Gemini 计划草稿**（前端权威）:
   - ROLE_FILE: `~/.claude/.ccg/prompts/gemini/architect.md`
   - OUTPUT: 逐步计划 + 伪代码（重点：信息架构/交互/可访问性/视觉一致性）

使用 `TaskOutput` 等待两个模型的完整结果，记录提案的主要差异。

#### 2.4 实施计划生成（Claude 最终版本）

整合两种分析，生成**逐步实施计划**:

```markdown
## 实施计划: <任务名>

### 任务类型
- [ ] 前端(→ Gemini)
- [ ] 后端(→ Codex)
- [ ] 全栈(→ 并行)

### 技术解决方案
<从 Codex + Gemini 分析整合的最优解决方案>

### 实施步骤
1. <步骤1> - 预期交付物
2. <步骤2> - 预期交付物
...

### 关键文件
| 文件 | 操作 | 说明 |
|------|-----------|-------------|
| path/to/file.ts:L10-L50 | 变更 | 说明 |

### 风险与缓解措施
| 风险 | 缓解措施 |
|------|------------|

### SESSION_ID(供 /ccg:execute 使用)
- CODEX_SESSION: <session_id>
- GEMINI_SESSION: <session_id>
```

### 阶段 2 结束：交付计划（非实施）

**`/ccg:plan` 的责任在此结束。需要执行以下操作**:

1. 向用户展示完整实施计划（包括伪代码）
2. 将计划保存到 `.claude/plan/<feature-name>.md`（从需求中提取功能名，例如: `user-auth`、`payment-module`）
3. 以**粗体文本**输出提示（**必须使用实际保存的文件路径**）:

   ---
**计划已生成并保存至 `.claude/plan/actual-feature-name.md`**

**请审核上述计划。您可以:**
- **修改计划**: 告知需要调整的内容，我将更新计划
- **执行计划**: 将以下命令复制到新会话

   ```
   /ccg:execute .claude/plan/actual-feature-name.md
   ```
   ---

**注意**: 上述 `actual-feature-name.md` 必须替换为实际保存的文件名！

4. **立即结束当前响应**（在此停止。不再进行工具调用。）

**严格禁止**:
- 询问用户"Y/N"后自动执行（执行是 `/ccg:execute` 的责任）
- 对生产代码执行写入操作
- 自动调用 `/ccg:execute` 或任何实施操作
- 用户未明确要求变更时继续触发模型调用

---

## 计划保存

计划完成后，将计划保存至:

- **初始计划**: `.claude/plan/<feature-name>.md`
- **迭代版本**: `.claude/plan/<feature-name>-v2.md`、`.claude/plan/<feature-name>-v3.md`...

计划文件写入必须在向用户展示计划之前完成。

---

## 计划变更流程

用户要求修改计划时:

1. 根据用户反馈调整计划内容
2. 更新 `.claude/plan/<feature-name>.md` 文件
3. 重新展示修改后的计划
4. 再次提示用户审核或执行

---

## 下一步

用户批准后，**手动**执行:

```bash
/ccg:execute .claude/plan/<feature-name>.md
```

---

## 重要规则

1. **仅计划，不实施** – 此命令不执行代码变更
2. **无 Y/N 提示** – 仅展示计划，由用户决定下一步
3. **信任规则** – 后端遵循 Codex，前端遵循 Gemini
4. 外部模型**完全没有文件系统写入权限**
5. **SESSION_ID 继承** – 计划必须最后包含 `CODEX_SESSION` / `GEMINI_SESSION`（供 `/ccg:execute resume <SESSION_ID>` 使用）
