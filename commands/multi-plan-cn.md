---
description: 创建多模型协同实现计划，不修改生产代码。
---

# 计划 - 多模型协同规划

多模型协同规划 - 上下文检索 + 双模型分析 → 生成分步实现计划。

$ARGUMENTS

---

## 核心协议

- **语言协议**: 与工具/模型交互时使用**English**，与用户通信时使用用户语言
- **强制并行**: Codex/Gemini 调用必须使用 `run_in_background: true`（包括单模型调用，避免阻塞主线程）
- **代码主权**: 外部模型**零文件系统写入权限**，所有修改由 Claude 执行
- **止损机制**: 当前阶段输出未验证前不进入下一阶段
- **仅规划**: 此命令允许读取上下文和写入 `.claude/plan/*` 计划文件，但**绝不修改生产代码**

---

## 多模型调用规范

**调用语法**（并行：使用 `run_in_background: true`）：

```
Bash({
  command: "~/.claude/bin/codeagent-wrapper {{LITE_MODE_FLAG}}--backend <codex|gemini> {{GEMINI_MODEL_FLAG}}- \"$PWD\" <<'EOF'
ROLE_FILE: <role prompt path>
<TASK>
Requirement: <enhanced requirement>
Context: <retrieved project context>
</TASK>
OUTPUT: Step-by-step implementation plan with pseudo-code. DO NOT modify any files.
EOF",
  run_in_background: true,
  timeout: 3600000,
  description: "Brief description"
})
```

**模型参数说明**:
- `{{GEMINI_MODEL_FLAG}}`: 使用 `--backend gemini` 时替换为 `--gemini-model gemini-3-pro-preview`（注意末尾空格）；codex 使用空字符串

**角色提示词**:

| 阶段 | Codex | Gemini |
|-------|-------|--------|
| 分析 | `~/.claude/.ccg/prompts/codex/analyzer.md` | `~/.claude/.ccg/prompts/gemini/analyzer.md` |
| 规划 | `~/.claude/.ccg/prompts/codex/architect.md` | `~/.claude/.ccg/prompts/gemini/architect.md` |

**会话复用**: 每次调用返回 `SESSION_ID: xxx`（通常由 wrapper 输出），**必须保存**供后续 `/ccg:execute` 使用。

**等待后台任务**（最大超时 600000ms = 10 分钟）：

```
TaskOutput({ task_id: "<task_id>", block: true, timeout: 600000 })
```

**重要**:
- 必须指定 `timeout: 600000`，否则默认 30 秒会导致提前超时
- 10 分钟后仍未完成，继续使用 `TaskOutput` 轮询，**绝不杀死进程**
- 如因超时跳过等待，**必须**调用 `AskUserQuestion` 询问用户是继续等待还是杀死任务

---

## 执行工作流

**规划任务**: $ARGUMENTS

### 阶段 1：完整上下文检索

`[模式: 研究]`

#### 1.1 提示词增强（必须首先执行）

**如果 ace-tool MCP 可用**，调用 `mcp__ace-tool__enhance_prompt` 工具：

```
mcp__ace-tool__enhance_prompt({
  prompt: "$ARGUMENTS",
  conversation_history: "<last 5-10 conversation turns>",
  project_root_path: "$PWD"
})
```

等待增强后的提示词，**将原始 $ARGUMENTS 替换为增强结果**用于所有后续阶段。

**如果 ace-tool MCP 不可用**：跳过此步骤，对所有后续阶段使用原始 `$ARGUMENTS`。

#### 1.2 上下文检索

**如果 ace-tool MCP 可用**，调用 `mcp__ace-tool__search_context` 工具：

```
mcp__ace-tool__search_context({
  query: "<semantic query based on enhanced requirement>",
  project_root_path: "$PWD"
})
```

- 使用自然语言构建语义查询（Where/What/How）
- **绝不基于假设回答**

**如果 ace-tool MCP 不可用**，使用 Claude Code 内置工具作为后备：
1. **Glob**: 按模式查找相关文件（例如 `Glob("**/*.ts")`、`Glob("src/**/*.py")`
2. **Grep**: 搜索关键符号、函数名、类定义（例如 `Grep("className|functionName")`）
3. **Read**: 读取发现的文件以收集完整上下文
4. **Task (Explore agent)**: 深度探索时，使用 `Task` 配合 `subagent_type: "Explore"` 在代码库中搜索

#### 1.3 完整性检查

- 必须获取相关类、函数、变量的**完整定义和签名**
- 如上下文不足，触发**递归检索**
- 优先输出：入口文件 + 行号 + 关键符号名；仅在必要时添加最小代码片段以消除歧义

#### 1.4 需求对齐

- 如需求仍有歧义，**必须**输出引导性问题给用户
- 直到需求边界清晰（无遗漏、无冗余）

### 阶段 2：多模型协同分析

`[模式: 分析]`

#### 2.1 分发输入

**并行调用** Codex 和 Gemini (`run_in_background: true`)：

将**原始需求**（无预设意见）分发给两个模型：

1. **Codex 后端分析**:
   - ROLE_FILE: `~/.claude/.ccg/prompts/codex/analyzer.md`
   - 重点：技术可行性、架构影响、性能考虑、潜在风险
   - 输出：多角度解决方案 + 优缺点分析

2. **Gemini 前端分析**:
   - ROLE_FILE: `~/.claude/.ccg/prompts/gemini/analyzer.md`
   - 重点：UI/UX 影响、用户体验、视觉设计
   - 输出：多角度解决方案 + 优缺点分析

使用 `TaskOutput` 等待两个模型的完整结果。**保存 SESSION_ID** (`CODEX_SESSION` 和 `GEMINI_SESSION`)。

#### 2.2 交叉验证

整合观点并迭代优化：

1. **识别共识**（强信号）
2. **识别分歧**（需要权衡）
3. **优势互补**：后端逻辑遵循 Codex，前端设计遵循 Gemini
4. **逻辑推理**：消除解决方案中的逻辑漏洞

#### 2.3（可选但推荐）双模型计划草案

为降低 Claude 综合计划的遗漏风险，可并行让两个模型输出"计划草案"（仍**不允许**修改文件）：

1. **Codex 计划草案**（后端权威）:
   - ROLE_FILE: `~/.claude/.ccg/prompts/codex/architect.md`
   - 输出：分步计划 + 伪代码（重点：数据流/边界情况/错误处理/测试策略）

2. **Gemini 计划草案**（前端权威）:
   - ROLE_FILE: `~/.claude/.ccg/prompts/gemini/architect.md`
   - 输出：分步计划 + 伪代码（重点：信息架构/交互/可访问性/视觉一致性）

使用 `TaskOutput` 等待两个模型的完整结果，记录其建议中的关键差异。

#### 2.4 生成实现计划（Claude 最终版）

综合两份分析，生成**分步实现计划**:

```markdown
## 实现计划: <任务名称>

### 任务类型
- [ ] 前端 (→ Gemini)
- [ ] 后端 (→ Codex)
- [ ] 全栈 (→ 并行)

### 技术方案
<从 Codex + Gemini 分析综合得出的最优方案>

### 实现步骤
1. <步骤 1> - 预期交付物
2. <步骤 2> - 预期交付物
...

### 关键文件
| 文件 | 操作 | 描述 |
|------|-----------|-------------|
| path/to/file.ts:L10-L50 | 修改 | 描述 |

### 风险与缓解
| 风险 | 缓解措施 |
|------|------------|

### SESSION_ID（供 /ccg:execute 使用）
- CODEX_SESSION: <session_id>
- GEMINI_SESSION: <session_id>
```

### 阶段 2 结束：计划交付（非执行）

**`/ccg:plan` 责任在此结束，必须执行以下操作**:

1. 向用户呈现完整实现计划（包括伪代码）
2. 保存计划到 `.claude/plan/<feature-name>.md`（从需求中提取功能名称，例如 `user-auth`、`payment-module`）
3. **粗体文本**输出提示（必须使用实际保存的文件路径）：

---
**计划已生成并保存到 `.claude/plan/actual-feature-name.md`**

**请审阅上述计划。您可以:**
- **修改计划**: 告诉我需要调整的内容，我将更新计划
- **执行计划**: 将以下命令复制到新会话

```
/ccg:execute .claude/plan/actual-feature-name.md
```
---

**注意**: 上述 `actual-feature-name.md` 必须替换为实际保存的文件名！

4. **立即终止当前响应**（到此为止。不再有工具调用。）

**绝对禁止**:
- 询问用户"是/否"然后自动执行（执行是 `/ccg:execute` 的责任）
- 对生产代码的任何写入操作
- 自动调用 `/ccg:execute` 或任何实现操作
- 用户未明确请求修改时继续触发模型调用

---

## 计划保存

规划完成后，保存计划到：

- **首次规划**: `.claude/plan/<feature-name>.md`
- **迭代版本**: `.claude/plan/<feature-name>-v2.md`、`.claude/plan/<feature-name>-v3.md`...

计划文件写入应在向用户呈现计划之前完成。

---

## 计划修改流程

如用户请求计划修改：

1. 根据用户反馈调整计划内容
2. 更新 `.claude/plan/<feature-name>.md` 文件
3. 重新呈现修改后的计划
4. 提示用户再次审阅或执行

---

## 后续步骤

用户批准后，**手动**执行：

```bash
/ccg:execute .claude/plan/<feature-name>.md
```

---

## 关键规则

1. **仅规划，不执行** – 此命令不执行任何代码更改
2. **无是/否提示** – 仅呈现计划，让用户决定后续步骤
3. **信任规则** – 后端遵循 Codex，前端遵循 Gemini
4. 外部模型具有**零文件系统写入权限**
5. **SESSION_ID 交接** – 计划末尾必须包含 `CODEX_SESSION` / `GEMINI_SESSION`（供 `/ccg:execute resume <SESSION_ID>` 使用）
