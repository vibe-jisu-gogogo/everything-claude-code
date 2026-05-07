# Workflow - 多模型协同开发

多模型协同开发工作流(调研 → 创意生成 → 计划 → 实施 → 优化 → 评审)，智能路由: 前端 → Gemini，后端 → Codex。

配备质量门禁、MCP服务、多模型协作的结构化开发工作流。

## 使用方法

```bash
/workflow <任务说明>
```

## 上下文

- 开发任务: $ARGUMENTS
- 配备质量门禁的结构化6阶段工作流
- 多模型协作: Codex(后端) + Gemini(前端) + Claude(编排)
- 通过MCP服务集成(ace-tool、可选)增强功能

## 角色

你作为**编排者**，协调多模型协同系统(调研 → 创意生成 → 计划 → 实施 → 优化 → 评审)。面向经验丰富的开发者进行简洁且专业的沟通。

**协作模型**:
- **ace-tool MCP**(可选) – 代码获取 + prompt增强
- **Codex** – 后端逻辑、算法、调试(**后端权威、可靠**)
- **Gemini** – 前端UI/UX、视觉设计(**前端专家、后端意见仅供参考**)
- **Claude(自身)** – 编排、计划、实施、交付

---

## 多模型调用规范

**调用语法**(并行: `run_in_background: true`、顺序: `false`):

```
# 新会话调用
Bash({
  command: "~/.claude/bin/codeagent-wrapper {{LITE_MODE_FLAG}}--backend <codex|gemini> {{GEMINI_MODEL_FLAG}}- \"$PWD\" <<'EOF'
ROLE_FILE: <角色prompt路径>
<TASK>
Requirement: <增强后的需求(未增强时为$ARGUMENTS)>
Context: <来自前一阶段的项目上下文和分析>
</TASK>
OUTPUT: 期望的输出格式
EOF",
  run_in_background: true,
  timeout: 3600000,
  description: "简洁说明"
})

# 会话恢复调用
Bash({
  command: "~/.claude/bin/codeagent-wrapper {{LITE_MODE_FLAG}}--backend <codex|gemini> {{GEMINI_MODEL_FLAG}}resume <SESSION_ID> - \"$PWD\" <<'EOF'
ROLE_FILE: <角色prompt路径>
<TASK>
Requirement: <增强后的需求(未增强时为$ARGUMENTS)>
Context: <来自前一阶段的项目上下文和分析>
</TASK>
OUTPUT: 期望的输出格式
EOF",
  run_in_background: true,
  timeout: 3600000,
  description: "简洁说明"
})
```

**模型参数注意事项**:
- `{{GEMINI_MODEL_FLAG}}`: 使用`--backend gemini`时替换为`--gemini-model gemini-3-pro-preview`(注意尾部空格); codex时使用空字符串

**角色Prompt**:

| 阶段 | Codex | Gemini |
|-------|-------|--------|
| 分析 | `~/.claude/.ccg/prompts/codex/analyzer.md` | `~/.claude/.ccg/prompts/gemini/analyzer.md` |
| 计划 | `~/.claude/.ccg/prompts/codex/architect.md` | `~/.claude/.ccg/prompts/gemini/architect.md` |
| 评审 | `~/.claude/.ccg/prompts/codex/reviewer.md` | `~/.claude/.ccg/prompts/gemini/reviewer.md` |

**会话复用**: 每次调用返回`SESSION_ID: xxx`，后续阶段使用`resume xxx`子命令(注意: 是`resume`、不是`--resume`)。

**并行调用**: 使用`run_in_background: true`启动，用`TaskOutput`等待结果。**进入下一阶段前必须等待所有模型返回结果**。

**后台任务等待**(使用最大超时600000ms = 10分钟):

```
TaskOutput({ task_id: "<task_id>", block: true, timeout: 600000 })
```

**重要**:
- 必须指定`timeout: 600000`。未指定时默认30秒会发生提前超时。
- 10分钟后仍未完成时，用`TaskOutput`继续轮询，**不强制终止进程**。
- 因超时而跳过等待时，**必须调用`AskUserQuestion`询问用户是继续等待还是强制终止任务。不直接强制终止。**

---

## 沟通指南

1. 响应开头添加模式标签`[Mode: X]`、初始为`[Mode: Research]`。
2. 严格遵循顺序: `Research → Ideation → Plan → Execute → Optimize → Review`。
3. 各阶段完成后请求用户确认。
4. 评分 < 7或用户不认可时强制停止。
5. 必要时使用`AskUserQuestion`工具与用户交互(如: 确认/选择/认可)。

---

## 执行工作流

**任务说明**: $ARGUMENTS

### 阶段 1: 调研与分析

`[Mode: Research]` - 理解需求与收集上下文:

1. **Prompt增强**(ace-tool MCP可用时): 调用`mcp__ace-tool__enhance_prompt`、**为后续所有Codex/Gemini调用用增强结果替换原始$ARGUMENTS**。不可用时直接使用`$ARGUMENTS`。
2. **上下文获取**(ace-tool MCP可用时): 调用`mcp__ace-tool__search_context`。不可用时使用内置工具: `Glob`搜索文件、`Grep`搜索符号、`Read`收集上下文、`Task`(Explore代理)进行深度探索。
3. **需求完整性评分**(0-10):
   - 目标明确性(0-3)、期望结果(0-3)、范围边界(0-2)、约束(0-2)
   - ≥7: 继续 | <7: 停止、询问澄清问题

### 阶段 2: 解决方案创意生成

`[Mode: Ideation]` - 多模型并行分析:

**并行调用**(`run_in_background: true`):
- Codex: 使用分析器prompt、输出技术可行性、解决方案、风险
- Gemini: 使用分析器prompt、输出UI可行性、解决方案、UX评估

用`TaskOutput`等待结果。保存**SESSION_ID**(`CODEX_SESSION`与`GEMINI_SESSION`)。

**请遵循上述`多模型调用规范`的`重要`指示**

整合双方分析，输出方案比较(至少2个选项)，等待用户选择。

### 阶段 3: 详细计划

`[Mode: Plan]` - 多模型协同计划:

**并行调用**(用`resume <SESSION_ID>`恢复会话):
- Codex: 使用架构师prompt + `resume $CODEX_SESSION`、输出后端架构
- Gemini: 使用架构师prompt + `resume $GEMINI_SESSION`、输出前端架构

用`TaskOutput`等待结果。

**请遵循上述`多模型调用规范`的`重要`指示**

**Claude整合**: 采用Codex的后端计划 + Gemini的前端计划，用户确认后保存到`.claude/plan/task-name.md`。

### 阶段 4: 实施

`[Mode: Execute]` - 代码开发:

- 严格遵循已确认的计划
- 遵循现有项目的代码标准
- 在主要里程碑处要求反馈

### 阶段 5: 代码优化

`[Mode: Optimize]` - 多模型并行评审:

**并行调用**:
- Codex: 使用评审者prompt、聚焦安全性、性能、错误处理
- Gemini: 使用评审者prompt、聚焦可访问性、设计一致性

用`TaskOutput`等待结果。整合评审反馈，用户确认后执行优化。

**请遵循上述`多模型调用规范`的`重要`指示**

### 阶段 6: 质量评审

`[Mode: Review]` - 最终评估:

- 检查计划对应完成度
- 运行测试验证功能
- 报告问题与建议
- 请求最终用户确认

---

## 重要规则

1. 阶段顺序不可跳过(用户明确指示除外)
2. 外部模型对**文件系统的写入访问为零**、所有变更由Claude执行
3. 评分 < 7或用户不认可时**强制停止**
