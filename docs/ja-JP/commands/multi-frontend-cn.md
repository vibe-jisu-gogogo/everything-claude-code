# Frontend - 前端中心开发

前端中心工作流程（调研 → 创意生成 → 计划 → 实现 → 优化 → 评审），Gemini主导。

## 使用方法

```bash
/frontend <UI任务说明>
```

## 上下文

- 前端任务: $ARGUMENTS
- Gemini主导，Codex仅用于辅助参考
- 适用范围: 组件设计、响应式布局、UI动画、样式优化

## 角色

你作为**前端协调器**，为UI/UX任务协调多模型协作（调研 → 创意生成 → 计划 → 实现 → 优化 → 评审）。

**协作模型**:
- **Gemini** – 前端UI/UX（**前端权威、可信**）
- **Codex** – 后端视角（**前端意见仅供参考**）
- **Claude(自身)** – 协调、计划、实现、交付

---

## 多模型调用规范

**调用语法**:

```
# 新会话调用
Bash({
  command: "~/.claude/bin/codeagent-wrapper {{LITE_MODE_FLAG}}--backend gemini --gemini-model gemini-3-pro-preview - \"$PWD\" <<'EOF'
ROLE_FILE: <角色提示词路径>
<TASK>
Requirement: <增强后的要求（如未增强则使用$ARGUMENTS）>
Context: <来自上一阶段的项目上下文与分析>
</TASK>
OUTPUT: 期望的输出格式
EOF",
  run_in_background: false,
  timeout: 3600000,
  description: "简要说明"
})

# 会话恢复调用
Bash({
  command: "~/.claude/bin/codeagent-wrapper {{LITE_MODE_FLAG}}--backend gemini --gemini-model gemini-3-pro-preview resume <SESSION_ID> - \"$PWD\" <<'EOF'
ROLE_FILE: <角色提示词路径>
<TASK>
Requirement: <增强后的要求（如未增强则使用$ARGUMENTS）>
Context: <来自上一阶段的项目上下文与分析>
</TASK>
OUTPUT: 期望的输出格式
EOF",
  run_in_background: false,
  timeout: 3600000,
  description: "简要说明"
})
```

**角色提示词**:

| 阶段 | Gemini |
|-------|--------|
| 分析 | `~/.claude/.ccg/prompts/gemini/analyzer.md` |
| 计划 | `~/.claude/.ccg/prompts/gemini/architect.md` |
| 评审 | `~/.claude/.ccg/prompts/gemini/reviewer.md` |

**会话复用**: 每次调用返回`SESSION_ID: xxx`。在后续阶段使用`resume xxx`。在阶段2保存`GEMINI_SESSION`，在阶段3和5使用`resume`。

---

## 沟通指南

1. 在响应开头添加模式标签`[Mode: X]`，初始为`[Mode: Research]`
2. 严格遵循顺序: `Research → Ideation → Plan → Execute → Optimize → Review`
3. 必要时使用`AskUserQuestion`工具与用户交互（例如：确认/选择/批准）

---

## 核心工作流

### 阶段 0: 提示词增强（可选）

`[Mode: Prepare]` - 如果ace-tool MCP可用，调用`mcp__ace-tool__enhance_prompt`，**将原始$ARGUMENTS替换为增强结果用于后续Gemini调用**。如不可用则直接使用`$ARGUMENTS`。

### 阶段 1: 调研

`[Mode: Research]` - 理解需求并收集上下文

1. **代码获取**（如果ace-tool MCP可用）: 调用`mcp__ace-tool__search_context`获取现有组件、样式、设计系统。如不可用则使用内置工具: `Glob`搜索文件，`Grep`搜索组件/样式，`Read`收集上下文，`Task`（Explore代理）进行深度探索。
2. 需求完整性评分（0-10）: >=7继续，<7停止并补充

### 阶段 2: 创意生成

`[Mode: Ideation]` - Gemini主导的分析

**需要调用Gemini**（遵循上述调用规范）:
- ROLE_FILE: `~/.claude/.ccg/prompts/gemini/analyzer.md`
- Requirement: 增强后的要求（如未增强则使用$ARGUMENTS）
- Context: 来自阶段1的项目上下文
- OUTPUT: UI可行性分析、推荐解决方案（至少2个）、UX评估

保存**SESSION_ID**（`GEMINI_SESSION`）用于后续阶段复用。

输出解决方案（至少2个），等待用户选择。

### 阶段 3: 计划

`[Mode: Plan]` - Gemini主导的计划

**需要调用Gemini**（使用`resume <GEMINI_SESSION>`复用会话）:
- ROLE_FILE: `~/.claude/.ccg/prompts/gemini/architect.md`
- Requirement: 用户选择的解决方案
- Context: 来自阶段2的分析结果
- OUTPUT: 组件结构、UI流程、样式方法

Claude整合计划，经用户批准后保存至`.claude/plan/task-name.md`。

### 阶段 4: 实现

`[Mode: Execute]` - 代码开发

- 严格遵循已批准的计划
- 遵循现有项目的设计系统和代码标准
- 确保响应式和可访问性

### 阶段 5: 优化

`[Mode: Optimize]` - Gemini主导的评审

**需要调用Gemini**（遵循上述调用规范）:
- ROLE_FILE: `~/.claude/.ccg/prompts/gemini/reviewer.md`
- Requirement: 评审以下前端代码变更
- Context: git diff或代码内容
- OUTPUT: 可访问性、响应式、性能、设计一致性问题列表

整合评审反馈，经用户确认后执行优化。

### 阶段 6: 质量评审

`[Mode: Review]` - 最终评估

- 检查相对于计划的完成度
- 验证响应式和可访问性
- 报告问题和建议

---

## 重要规则

1. **Gemini的前端意见可信**
2. **Codex的前端意见仅供参考**
3. 外部模型**对文件系统零写入权限**
4. Claude处理所有代码写入和文件操作
