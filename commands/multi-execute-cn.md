---
description: 执行多模型实现计划，同时保证 Claude 是唯一的文件系统写入者。
---

# 执行 - 多模型协作执行

多模型协作执行 - 从计划获取原型 → Claude 重构并实现 → 多模型审计与交付。

$ARGUMENTS

---

## 核心协议

- **语言协议**：与工具/模型交互时使用**English**，与用户沟通时使用其母语
- **代码主权**：外部模型**没有任何文件系统写入权限**，所有修改由 Claude 执行
- **粗糙原型重构**：将 Codex/Gemini 统一差异视为“粗糙原型”，必须重构为生产级代码
- **止损机制**：当前阶段输出验证通过前，不得进入下一阶段
- **前提条件**：仅在用户对 `/ccg:plan` 输出明确回复“Y”后执行（如果缺失，必须先确认）

---

## 多模型调用规范

**调用语法**（并行：使用 `run_in_background: true`）：

```
# Resume session call (recommended) - Implementation Prototype
Bash({
  command: "~/.claude/bin/codeagent-wrapper {{LITE_MODE_FLAG}}--backend <codex|gemini> {{GEMINI_MODEL_FLAG}}resume <SESSION_ID> - \"$PWD\" <<'EOF'
ROLE_FILE: <role prompt path>
<TASK>
Requirement: <task description>
Context: <plan content + target files>
</TASK>
OUTPUT: Unified Diff Patch ONLY. Strictly prohibit any actual modifications.
EOF",
  run_in_background: true,
  timeout: 3600000,
  description: "Brief description"
})

# New session call - Implementation Prototype
Bash({
  command: "~/.claude/bin/codeagent-wrapper {{LITE_MODE_FLAG}}--backend <codex|gemini> {{GEMINI_MODEL_FLAG}}- \"$PWD\" <<'EOF'
ROLE_FILE: <role prompt path>
<TASK>
Requirement: <task description>
Context: <plan content + target files>
</TASK>
OUTPUT: Unified Diff Patch ONLY. Strictly prohibit any actual modifications.
EOF",
  run_in_background: true,
  timeout: 3600000,
  description: "Brief description"
})
```

**审计调用语法**（Code Review / 审计）：

```
Bash({
  command: "~/.claude/bin/codeagent-wrapper {{LITE_MODE_FLAG}}--backend <codex|gemini> {{GEMINI_MODEL_FLAG}}resume <SESSION_ID> - \"$PWD\" <<'EOF'
ROLE_FILE: <role prompt path>
<TASK>
Scope: Audit the final code changes.
Inputs:
- The applied patch (git diff / final unified diff)
- The touched files (relevant excerpts if needed)
Constraints:
- Do NOT modify any files.
- Do NOT output tool commands that assume filesystem access.
</TASK>
OUTPUT:
1) A prioritized list of issues (severity, file, rationale)
2) Concrete fixes; if code changes are needed, include a Unified Diff Patch in a fenced code block.
EOF",
  run_in_background: true,
  timeout: 3600000,
  description: "Brief description"
})
```

**模型参数说明**：
- `{{GEMINI_MODEL_FLAG}}`：使用 `--backend gemini` 时，替换为 `--gemini-model gemini-3-pro-preview`（注意末尾空格）；codex 使用空字符串

**角色提示词**：

| 阶段 | Codex | Gemini |
|-------|-------|--------|
| 实现 | `~/.claude/.ccg/prompts/codex/architect.md` | `~/.claude/.ccg/prompts/gemini/frontend.md` |
| 评审 | `~/.claude/.ccg/prompts/codex/reviewer.md` | `~/.claude/.ccg/prompts/gemini/reviewer.md` |

**会话复用**：如果 `/ccg:plan` 提供了 SESSION_ID，使用 `resume <SESSION_ID>` 复用上下文。

**等待后台任务**（最大超时 600000ms = 10 分钟）：

```
TaskOutput({ task_id: "<task_id>", block: true, timeout: 600000 })
```

**重要提示**：
- 必须指定 `timeout: 600000`，否则默认 30 秒会导致提前超时
- 如果 10 分钟后仍未完成，继续使用 `TaskOutput` 轮询，**绝对不要终止进程**
- 如果因超时而跳过等待，**必须调用 `AskUserQuestion` 询问用户是继续等待还是终止任务**

---

## 执行工作流

**执行任务**：$ARGUMENTS

### 第 0 阶段：读取计划

`[模式：准备]`

1. **识别输入类型**：
   - 计划文件路径（例如 `.claude/plan/xxx.md`）
   - 直接任务描述

2. **读取计划内容**：
   - 如果提供了计划文件路径，读取并解析
   - 提取：任务类型、实现步骤、关键文件、SESSION_ID

3. **执行前确认**：
   - 如果输入是“直接任务描述”或计划缺少 `SESSION_ID` / 关键文件：先与用户确认
   - 如果无法确认用户已对计划回复“Y”：继续前必须再次确认

4. **任务类型路由**：

   | 任务类型 | 检测条件 | 路由 |
   |-----------|-----------|-------|
   | **前端** | 页面、组件、UI、样式、布局 | Gemini |
   | **后端** | API、接口、数据库、逻辑、算法 | Codex |
   | **全栈** | 同时包含前端和后端 | Codex ∥ Gemini 并行 |

---

### 第 1 阶段：快速上下文获取

`[模式：检索]`

**如果 ace-tool MCP 可用**，使用它进行快速上下文检索：

基于计划中的“关键文件”列表，调用 `mcp__ace-tool__search_context`：

```
mcp__ace-tool__search_context({
  query: "<semantic query based on plan content, including key files, modules, function names>",
  project_root_path: "$PWD"
})
```

**检索策略**：
- 从计划的“关键文件”表中提取目标路径
- 构建语义查询，包含：入口文件、依赖模块、相关类型定义
- 如果结果不足，增加 1-2 次递归检索

**如果 ace-tool MCP 不可用**，使用 Claude Code 内置工具作为备选：
1. **Glob**: 从计划的“关键文件”表中查找目标文件（例如 `Glob("src/components/**/*.tsx")`）
2. **Grep**: 在代码库中搜索关键符号、函数名、类型定义
3. **Read**: 读取发现的文件以收集完整上下文
4. **Task (Explore agent)**: 如需更广泛的探索，使用 `Task` 搭配 `subagent_type: "Explore"`

**检索完成后**：
- 整理检索到的代码片段
- 确认实现所需的上下文完整
- 进入第 3 阶段

---

### 第 3 阶段：获取原型

`[模式：原型]`

**根据任务类型路由**：

#### 路线 A：前端/UI/样式 → Gemini

**限制**：上下文 < 32k tokens

1. 调用 Gemini（使用 `~/.claude/.ccg/prompts/gemini/frontend.md`）
2. 输入：计划内容 + 检索到的上下文 + 目标文件
3. OUTPUT: `Unified Diff Patch ONLY. Strictly prohibit any actual modifications.`
4. **Gemini 是前端设计权威，其 CSS/React/Vue 原型是最终视觉基准**
5. **警告**：忽略 Gemini 的后端逻辑建议
6. 如果计划包含 `GEMINI_SESSION`：优先使用 `resume <GEMINI_SESSION>`

#### 路线 B：后端/逻辑/算法 → Codex

1. 调用 Codex（使用 `~/.claude/.ccg/prompts/codex/architect.md`）
2. 输入：计划内容 + 检索到的上下文 + 目标文件
3. OUTPUT: `Unified Diff Patch ONLY. Strictly prohibit any actual modifications.`
4. **Codex 是后端逻辑权威，充分利用其逻辑推理和调试能力**
5. 如果计划包含 `CODEX_SESSION`：优先使用 `resume <CODEX_SESSION>`

#### 路线 C：全栈 → 并行调用

1. **并行调用** (`run_in_background: true`)：
   - Gemini：处理前端部分
   - Codex：处理后端部分
2. 使用 `TaskOutput` 等待两个模型返回完整结果
3. 各自使用计划中对应的 `SESSION_ID` 进行 `resume`（如果缺失则创建新会话）

**遵循上方“多模型调用规范”中的 `重要提示` 说明**

---

### 第 4 阶段：代码实现

`[模式：实现]`

**作为代码主权方的 Claude 执行以下步骤**：

1. **读取 Diff**：解析 Codex/Gemini 返回的 Unified Diff Patch

2. **思维沙箱**：
   - 模拟将 Diff 应用到目标文件
   - 检查逻辑一致性
   - 识别潜在冲突或副作用

3. **重构与清理**：
   - 将“粗糙原型”重构为**高可读性、可维护、企业级代码**
   - 移除冗余代码
   - 确保符合项目现有代码规范
   - **除非必要，不要生成注释/文档**，代码应自解释

4. **最小范围**：
   - 变更仅局限于需求范围
   - **必须审查**副作用
   - 进行针对性修正

5. **应用变更**：
   - 使用 Edit/Write 工具执行实际修改
   - **仅修改必要代码**，绝不影响用户其他现有功能

6. **自我验证**（强烈推荐）：
   - 运行项目现有 lint / 类型检查 / 测试（优先最小相关范围）
   - 如果失败：先修复回归问题，再进入第 5 阶段

---

### 第 5 阶段：审计与交付

`[模式：审计]`

#### 5.1 自动审计

**变更生效后，必须立即并行调用** Codex 和 Gemini 进行 Code Review：

1. **Codex 评审** (`run_in_background: true`)：
   - ROLE_FILE: `~/.claude/.ccg/prompts/codex/reviewer.md`
   - 输入：变更 Diff + 目标文件
   - 关注：安全性、性能、错误处理、逻辑正确性

2. **Gemini 评审** (`run_in_background: true`)：
   - ROLE_FILE: `~/.claude/.ccg/prompts/gemini/reviewer.md`
   - 输入：变更 Diff + 目标文件
   - 关注：可访问性、设计一致性、用户体验

使用 `TaskOutput` 等待两个模型返回完整评审结果。优先复用第 3 阶段的会话（`resume <SESSION_ID>`）以保持上下文一致性。

#### 5.2 整合与修复

1. 综合 Codex + Gemini 的评审反馈
2. 按信任规则权衡：后端遵循 Codex，前端遵循 Gemini
3. 执行必要的修复
4. 根据需要重复第 5.1 阶段（直到风险可接受）

#### 5.3 交付确认

审计通过后，向用户报告：

```markdown
## 执行完成

### 变更摘要
| 文件 | 操作 | 描述 |
|------|-----------|-------------|
| path/to/file.ts | 修改 | 描述 |

### 审计结果
- Codex: <通过/发现 N 个问题>
- Gemini: <通过/发现 N 个问题>

### 建议
1. [ ] <建议测试步骤>
2. [ ] <建议验证步骤>
```

---

## 关键规则

1. **代码主权** – 所有文件修改由 Claude 执行，外部模型无任何写入权限
2. **粗糙原型重构** – Codex/Gemini 输出视为草稿，必须重构
3. **信任规则** – 后端遵循 Codex，前端遵循 Gemini
4. **最小变更** – 仅修改必要代码，无副作用
5. **强制审计** – 变更后必须执行多模型 Code Review

---

## 使用方法

```bash
# 执行计划文件
/ccg:execute .claude/plan/feature-name.md

# 直接执行任务（适用于上下文已经讨论过的计划）
/ccg:execute implement user authentication based on previous plan
```

---

## 与 /ccg:plan 的关系

1. `/ccg:plan` 生成计划 + SESSION_ID
2. 用户回复“Y”确认
3. `/ccg:execute` 读取计划，复用 SESSION_ID，执行实现
