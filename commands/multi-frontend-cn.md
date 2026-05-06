---
description: 运行专注于前端的多模型工作流，用于组件、布局、动画和UI优化。
---

# Frontend - 专注于前端的开发

专注于前端的工作流（研究 → 构思 → 规划 → 执行 → 优化 → 评审），由 Gemini 主导。

## 用法

```bash
/frontend <UI 任务描述>
```

## 上下文

- 前端任务: $ARGUMENTS
- Gemini 主导，Codex 作为辅助参考
- 适用场景: 组件设计、响应式布局、UI 动画、样式优化

## 你的角色

你是**前端协调器**，负责协调多模型协作完成 UI/UX 任务（研究 → 构思 → 规划 → 执行 → 优化 → 评审）。

**协作模型**:
- **Gemini** – 前端 UI/UX（**前端权威，可信赖**）
- **Codex** – 后端视角（**前端意见仅供参考**）
- **Claude (自身)** – 协调、规划、执行、交付

---

## 多模型调用规范

**调用语法**:

```
# 新会话调用
Bash({
  command: "~/.claude/bin/codeagent-wrapper {{LITE_MODE_FLAG}}--backend gemini --gemini-model gemini-3-pro-preview - \"$PWD\" <<'EOF'
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

# 恢复会话调用
Bash({
  command: "~/.claude/bin/codeagent-wrapper {{LITE_MODE_FLAG}}--backend gemini --gemini-model gemini-3-pro-preview resume <SESSION_ID> - \"$PWD\" <<'EOF'
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

**角色提示词**:

| 阶段 | Gemini |
|-------|--------|
| 分析 | `~/.claude/.ccg/prompts/gemini/analyzer.md` |
| 规划 | `~/.claude/.ccg/prompts/gemini/architect.md` |
| 评审 | `~/.claude/.ccg/prompts/gemini/reviewer.md` |

**会话复用**: 每次调用返回 `SESSION_ID: xxx`，后续阶段使用 `resume xxx`。在第2阶段保存 `GEMINI_SESSION`，在第3和第5阶段使用 `resume`。

---

## 沟通指南

1. 响应以模式标签 `[Mode: X]` 开头，初始为 `[Mode: Research]`
2. 严格遵循顺序: `研究 → 构思 → 规划 → 执行 → 优化 → 评审`
3. 需要时使用 `AskUserQuestion` 工具与用户交互（例如：确认/选择/批准）

---

## 核心工作流

### 阶段 0: 提示词增强（可选）

`[Mode: Prepare]` - 如果 ace-tool MCP 可用，调用 `mcp__ace-tool__enhance_prompt`，**将后续 Gemini 调用的原始 $ARGUMENTS 替换为增强后的结果**。如果不可用，直接使用 `$ARGUMENTS`。

### 阶段 1: 研究

`[Mode: Research]` - 理解需求并收集上下文

1. **代码检索**（如果 ace-tool MCP 可用）: 调用 `mcp__ace-tool__search_context` 检索现有组件、样式、设计系统。如果不可用，使用内置工具: `Glob` 用于文件发现，`Grep` 用于组件/样式搜索，`Read` 用于上下文收集，`Task`（Explore agent）用于更深入的探索。
2. 需求完整性评分（0-10）: >=7 继续，<7 停止并补充信息

### 阶段 2: 构思

`[Mode: Ideation]` - Gemini 主导分析

**必须调用 Gemini**（遵循上述调用规范）:
- ROLE_FILE: `~/.claude/.ccg/prompts/gemini/analyzer.md`
- Requirement: 增强后的需求（如果未增强则为 $ARGUMENTS）
- Context: 第1阶段的项目上下文
- OUTPUT: UI 可行性分析、推荐解决方案（至少2个）、UX 评估

**保存 SESSION_ID**（`GEMINI_SESSION`）供后续阶段复用。

输出解决方案（至少2个），等待用户选择。

### 阶段 3: 规划

`[Mode: Plan]` - Gemini 主导规划

**必须调用 Gemini**（使用 `resume <GEMINI_SESSION>` 复用会话）:
- ROLE_FILE: `~/.claude/.ccg/prompts/gemini/architect.md`
- Requirement: 用户选择的解决方案
- Context: 第2阶段的分析结果
- OUTPUT: 组件结构、UI 流程、样式方案

Claude 综合规划，用户批准后保存到 `.claude/plan/task-name.md`。

### 阶段 4: 实现

`[Mode: Execute]` - 代码开发

- 严格遵循已批准的计划
- 遵循现有项目设计系统和代码规范
- 确保响应式、可访问性

### 阶段 5: 优化

`[Mode: Optimize]` - Gemini 主导评审

**必须调用 Gemini**（遵循上述调用规范）:
- ROLE_FILE: `~/.claude/.ccg/prompts/gemini/reviewer.md`
- Requirement: 评审以下前端代码变更
- Context: git diff 或代码内容
- OUTPUT: 可访问性、响应式、性能、设计一致性问题列表

整合评审反馈，用户确认后执行优化。

### 阶段 6: 质量评审

`[Mode: Review]` - 最终评估

- 对照计划检查完成情况
- 验证响应式和可访问性
- 报告问题和建议

---

## 关键规则

1. **Gemini 的前端意见是可信赖的**
2. **Codex 的前端意见仅供参考**
3. 外部模型**没有文件系统写入权限**
4. Claude 处理所有代码写入和文件操作
