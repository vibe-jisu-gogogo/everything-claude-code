# Workflow - 多模型协同开发

多模型协同开发工作流(调研 → 创意生成 → 计划 → 实施 → 优化 → 评审)，智能路由: 前端 → Gemini、后端 → Codex。

具备质量门禁、MCP 服务、多模型联动的结构化开发工作流。

## 使用方法

```bash
/workflow <任务说明>
```

## 上下文

- 开发任务: $ARGUMENTS
- 具备质量门禁的结构化 6 阶段工作流
- 多模型联动: Codex(后端) + Gemini(前端) + Claude(编排)
- MCP 服务集成(ace-tool、可选)实现功能增强

## 职责

你作为**编排器**，协调多模型协同系统(调研 → 创意生成 → 计划 → 实施 → 优化 → 评审)。面向经验丰富的开发者进行简洁专业的沟通。

**协作模型**:
- **ace-tool MCP**(可选) – 代码获取 + Prompt 增强
- **Codex** – 后端逻辑、算法、调试(**后端权威、可信**)
- **Gemini** – 前端 UI/UX、可视化设计(**前端专家、后端意见仅供参考**)
- **Claude(自身)** – 编排、计划、实施、交付

---

## 多模型调用规范

**调用语法**(并行: `run_in_background: true`、串行: `false`):

```
# 新会话调用
Bash({
  command: "~/.claude/bin/codeagent-wrapper {{LITE_MODE_FLAG}}--backend <codex|gemini> {{GEMINI_MODEL_FLAG}}- \"$PWD\" <<'EOF'
ROLE_FILE: <角色提示路径>
<TASK>
Requirement: <增强后的要求(如未增强则为$ARGUMENTS)>
Context: <来自前一阶段的项目上下文和分析>
</TASK>
OUTPUT: 期望的输出格式
EOF",
  run_in_background: true,
  timeout: 3600000,
  description: "简明说明"
})

# 会话恢复调用
Bash({
  command: "~/.claude/bin/codeagent-wrapper {{LITE_MODE_FLAG}}--backend <codex|gemini> {{GEMINI_MODEL_FLAG}}resume <SESSION_ID> - \"$PWD\" <<'EOF'
ROLE_FILE: <角色提示路径>
<TASK>
Requirement: <增强后的要求(如未增强则为$ARGUMENTS)>
Context: <来自前一阶段的项目上下文和分析>
</TASK>
OUTPUT: 期望的输出格式
EOF",
  run_in_background: true,
  timeout: 3600000,
  description: "简明说明"
})
```

**模型参数注意事项**:
- `{{GEMINI_MODEL_FLAG}}`: 使用`--backend gemini`时替换为`--gemini-model gemini-3-pro-preview`(注意末尾空格); codex 时使用空字符串

**角色提示**:

| 阶段 | Codex | Gemini |
|-------|-------|--------|
| 分析 | `~/.claude/.ccg/prompts/codex/analyzer.md` | `~/.claude/.ccg/prompts/gemini/analyzer.md` |
| 计划 | `~/.claude/.ccg/prompts/codex/architect.md` | `~/.claude/.ccg/prompts/gemini/architect.md` |
| 评审 | `~/.claude/.ccg/prompts/codex/reviewer.md` | `~/.claude/.ccg/prompts/gemini/reviewer.md` |

**会话复用**: 每次调用返回`SESSION_ID: xxx`，后续阶段使用`resume xxx`子命令(注意: 是`resume`不是`--resume`)。

**并行调用**: 使用`run_in_background: true`启动，`TaskOutput`等待结果。**进入下一阶段前必须等待所有模型返回结果**。

**后台任务等待**(使用最大超时 600000ms = 10 分钟):

```
TaskOutput({ task_id: "<task_id>", block: true, timeout: 600000 })
```

**重要**:
- 必须指定`timeout: 600000`。如未指定将以默认 30 秒发生早期超时。
- 10 分钟后仍未完成时，继续使用`TaskOutput`进行轮询，**不要强制终止进程**。
- 因超时导致等待被跳过时，**必须调用`AskUserQuestion`询问用户是继续等待还是强制终止任务。不要直接强制终止。**

---

## 沟通指南

1. 响应开始时附加模式标签`[Mode: X]`，初始为`[Mode: Research]`。
2. 严格遵循顺序: `Research → Ideation → Plan → Execute → Optimize → Review`。
3. 各阶段完成后请求用户确认。
4. 评分 < 7 或用户不予认可时强制终止。
5. 根据需要使用`AskUserQuestion`工具与用户交互(如: 确认/选择/认可)。

---

## 执行工作流

**任务说明**: $ARGUMENTS

### 阶段 1: 调研与分析

`[Mode: Research]` - 理解需求并收集上下文:

1. **Prompt 增强**(ace-tool MCP 可用时): 调用`mcp__ace-tool__enhance_prompt`，**为后续所有 Codex/Gemini 调用使用增强结果替换原始的$ARGUMENTS**。不可用时直接使用`$ARGUMENTS`。
2. **上下文获取**(ace-tool MCP 可用时): 调用`mcp__ace-tool__search_context`。不可用时使用内置工具: `Glob`搜索文件、`Grep`搜索符号、`Read`收集上下文、`Task`(Explore 代理)进行更深入的探索。
3. **需求完整性评分**(0-10):
   - 目标清晰度(0-3)、预期结果(0-3)、范围边界(0-2)、制约条件(0-2)
   - ≥7: 继续 | <7: 停止、询问澄清问题

### 阶段 2: 解决方案创意生成

`[Mode: Ideation]` - 多模型并行分析:

**并行调用**(`run_in_background: true`):
- Codex: 使用分析器提示，输出技术可行性、解决方案、风险
- Gemini: 使用分析器提示，输出 UI 可行性、解决方案、UX 评估

使用`TaskOutput`等待结果。保存**SESSION_ID**(`CODEX_SESSION`与`GEMINI_SESSION`)。

**请严格遵循上述`多模型调用规范`中的`重要`指示**

整合双方分析，输出方案比较(至少 2 个选项)，并等待用户选择。

### 阶段 3: 详细计划

`[Mode: Plan]` - 多模型协同计划:

**并行调用**(使用`resume <SESSION_ID>`恢复会话):
- Codex: 使用架构师提示 + `resume $CODEX_SESSION`，输出后端架构
- Gemini: 使用架构师提示 + `resume $GEMINI_SESSION`，输出前端架构

使用`TaskOutput`等待结果。

**请严格遵循上述`多模型调用规范`中的`重要`指示**

**Claude 整合**: 采用 Codex 的后端计划 + Gemini 的前端计划，经用户认可后保存至`.claude/plan/task-name.md`。

### 阶段 4: 实施

`[Mode: Execute]` - 代码开发:

- 严格遵循已获批的计划
- 遵循现有项目的代码标准
- 在关键里程碑处请求反馈

### 阶段 5: 代码优化

`[Mode: Optimize]` - 多模型并行评审:

**并行调用**:
- Codex: 使用评审员提示，聚焦安全性、性能、错误处理
- Gemini: 使用评审员提示，聚焦可访问性、设计一致性

使用`TaskOutput`等待结果。整合评审反馈，经用户确认后执行优化。

**请严格遵循上述`多模型调用规范`中的`重要`指示**

### 阶段 6: 质量评审

`[Mode: Review]` - 最终评估:

- 检查相对于计划的完成度
- 执行测试以验证功能
- 报告问题和建议事项
- 请求最终用户确认

---

## 重要规则

1. 阶段顺序不可跳过(除非用户明确指示)
2. 外部模型**对文件系统的写入权限为零**，所有变更由 Claude 执行
3. 评分 < 7 或用户不予认可时**强制终止**
