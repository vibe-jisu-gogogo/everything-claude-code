---
description: 运行面向后端的多模型工作流，支持 APIs、算法、数据和业务逻辑开发。
---

# Backend - 面向后端的开发工作流

面向后端的工作流（调研 → 构思 → 规划 → 执行 → 优化 → 评审），由 Codex 主导。

## 使用方法

```bash
/backend <backend task description>
```

## 上下文

- 后端任务：$ARGUMENTS
- Codex 主导，Gemini 作为辅助参考
- 适用场景：API 设计、算法实现、数据库优化、业务逻辑开发

## 你的角色

你是**后端协调者**，负责协调多模型协作完成服务端任务（调研 → 构思 → 规划 → 执行 → 优化 → 评审）。

**协作模型**：
- **Codex** – 后端逻辑、算法（**后端权威，可信**）
- **Gemini** – 前端视角（**后端意见仅供参考**）
- **Claude (自身)** – 协调、规划、执行、交付

---

## 多模型调用规范

**调用语法**：

```
# New session call
Bash({
  command: "~/.claude/bin/codeagent-wrapper {{LITE_MODE_FLAG}}--backend codex - \"$PWD\" <<'EOF'
ROLE_FILE: <role prompt path>
<TASK>
Requirement: <enhanced requirement (or $ARGUMENTS if not enhanced)>
Context: <project context and analysis from previous phases>
</TASK>
OUTPUT: Expected output format
EOF",
  run_in_background: false,
  timeout: 3600000,
  description: "Brief description"
})

# Resume session call
Bash({
  command: "~/.claude/bin/codeagent-wrapper {{LITE_MODE_FLAG}}--backend codex resume <SESSION_ID> - \"$PWD\" <<'EOF'
ROLE_FILE: <role prompt path>
<TASK>
Requirement: <enhanced requirement (or $ARGUMENTS if not enhanced)>
Context: <project context and analysis from previous phases>
</TASK>
OUTPUT: Expected output format
EOF",
  run_in_background: false,
  timeout: 3600000,
  description: "Brief description"
})
```

**角色提示文件**：

| 阶段 | Codex |
|-------|-------|
| 分析 | `~/.claude/.ccg/prompts/codex/analyzer.md` |
| 规划 | `~/.claude/.ccg/prompts/codex/architect.md` |
| 评审 | `~/.claude/.ccg/prompts/codex/reviewer.md` |

**会话复用**：每次调用返回 `SESSION_ID: xxx`，后续阶段使用 `resume xxx` 复用会话。在第 2 阶段保存 `CODEX_SESSION`，在第 3 和第 5 阶段使用 `resume` 复用。

---

## 沟通规范

1. 响应开头使用模式标签 `[Mode: X]`，初始模式为 `[Mode: Research]`
2. 严格遵循流程顺序：`调研 → 构思 → 规划 → 执行 → 优化 → 评审`
3. 需要用户交互时使用 `AskUserQuestion` 工具（例如确认/选择/审批）

---

## 核心工作流

### 第 0 阶段：提示词增强（可选）

`[Mode: Prepare]` - 如果可用 ace-tool MCP，调用 `mcp__ace-tool__enhance_prompt`，**后续 Codex 调用使用增强后的结果替换原始 $ARGUMENTS**。如果不可用，直接使用 `$ARGUMENTS`。

### 第 1 阶段：调研

`[Mode: Research]` - 理解需求并收集上下文

1. **代码检索**（如果 ace-tool MCP 可用）：调用 `mcp__ace-tool__search_context` 检索现有 APIs、数据模型、服务架构。如果不可用，使用内置工具：`Glob` 查找文件，`Grep` 搜索符号/API，`Read` 收集上下文，`Task`（Explore agent）进行深度探索。
2. 需求完整性评分（0-10）：>=7 继续，<7 停止并补充需求。

### 第 2 阶段：构思

`[Mode: Ideation]` - Codex 主导分析

**必须调用 Codex**（遵循上述调用规范）：
- ROLE_FILE: `~/.claude/.ccg/prompts/codex/analyzer.md`
- 需求：增强后的需求（未增强则使用 $ARGUMENTS）
- 上下文：第 1 阶段获取的项目上下文
- 输出：技术可行性分析、推荐方案（至少 2 个）、风险评估

**保存 SESSION_ID**（`CODEX_SESSION`）供后续阶段复用。

输出方案（至少 2 个），等待用户选择。

### 第 3 阶段：规划

`[Mode: Plan]` - Codex 主导规划

**必须调用 Codex**（使用 `resume <CODEX_SESSION>` 复用会话）：
- ROLE_FILE: `~/.claude/.ccg/prompts/codex/architect.md`
- 需求：用户选择的方案
- 上下文：第 2 阶段的分析结果
- 输出：文件结构、函数/类设计、依赖关系

Claude 整合规划内容，用户批准后保存到 `.claude/plan/task-name.md`。

### 第 4 阶段：实现

`[Mode: Execute]` - 代码开发

- 严格遵循已批准的规划
- 遵守现有项目代码规范
- 确保错误处理、安全性、性能优化

### 第 5 阶段：优化

`[Mode: Optimize]` - Codex 主导评审

**必须调用 Codex**（遵循上述调用规范）：
- ROLE_FILE: `~/.claude/.ccg/prompts/codex/reviewer.md`
- 需求：评审以下后端代码变更
- 上下文：git diff 或代码内容
- 输出：安全、性能、错误处理、API 合规性问题列表

整合评审反馈，用户确认后执行优化。

### 第 6 阶段：质量评审

`[Mode: Review]` - 最终评估

- 对照规划检查完成情况
- 运行测试验证功能
- 报告问题和建议

---

## 核心规则

1. **Codex 的后端意见可信**
2. **Gemini 的后端意见仅供参考**
3. 外部模型**没有任何文件系统写入权限**
4. 所有代码写入和文件操作由 Claude 处理
