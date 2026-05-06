# Orchestrate 命令

用于复杂任务的顺序代理工作流。

## 使用方法

`/orchestrate [workflow-type] [task-description]`

## 工作流类型

### feature
完整功能实现工作流：
```
planner -> tdd-guide -> code-reviewer -> security-reviewer
```

### bugfix
Bug 调查和修复工作流：
```
planner -> tdd-guide -> code-reviewer
```

### refactor
安全重构工作流：
```
architect -> code-reviewer -> tdd-guide
```

### security
以安全为中心的审查：
```
security-reviewer -> code-reviewer -> architect
```

## 执行模式

对于工作流中的每个代理：

1. 使用前一个代理的上下文 **调用代理**
2. 使用结构化交接文档 **收集输出**
3. **传递给链中的下一个代理**
4. **综合结果** 生成最终报告

## 交接文档格式

在代理之间生成交接文档：

```markdown
## HANDOFF: [previous-agent] -> [next-agent]

### Context
[已完成工作的摘要]

### Findings
[主要发现或决策]

### Files Modified
[已修改文件列表]

### Open Questions
[下一个代理需要解决的未解决问题]

### Recommendations
[建议的下一步]
```

## 示例：Feature 工作流

```
/orchestrate feature "Add user authentication"
```

执行顺序：

1. **Planner 代理**
   - 分析需求
   - 制定实施计划
   - 识别依赖项
   - 输出：`HANDOFF: planner -> tdd-guide`

2. **TDD Guide 代理**
   - 读取 planner 交接文档
   - 先编写测试
   - 实现以通过测试
   - 输出：`HANDOFF: tdd-guide -> code-reviewer`

3. **Code Reviewer 代理**
   - 审查实现
   - 识别问题
   - 建议改进
   - 输出：`HANDOFF: code-reviewer -> security-reviewer`

4. **Security Reviewer 代理**
   - 安全审计
   - 漏洞检查
   - 最终批准
   - 输出：最终报告

## 最终报告格式

```
ORCHESTRATION REPORT
====================
Workflow: feature
Task: Add user authentication
Agents: planner -> tdd-guide -> code-reviewer -> security-reviewer

SUMMARY
-------
[单段摘要]

AGENT OUTPUTS
-------------
Planner: [摘要]
TDD Guide: [摘要]
Code Reviewer: [摘要]
Security Reviewer: [摘要]

FILES CHANGED
-------------
[所有已修改文件列表]

TEST RESULTS
------------
[测试通过/失败摘要]

SECURITY STATUS
---------------
[安全发现]

RECOMMENDATION
--------------
[SHIP / NEEDS WORK / BLOCKED]
```

## 并行执行

对于独立的检查，代理可以并行运行：

```markdown
### Parallel Phase
同时运行：
- code-reviewer (质量)
- security-reviewer (安全)
- architect (设计)

### Merge Results
将输出合并为单一报告
```

## 参数

$ARGUMENTS:
- `feature <description>` - 完整功能工作流
- `bugfix <description>` - Bug 修复工作流
- `refactor <description>` - 重构工作流
- `security <description>` - 安全审查工作流
- `custom <agents> <description>` - 自定义代理顺序

## 自定义工作流示例

```
/orchestrate custom "architect,tdd-guide,code-reviewer" "Redesign caching layer"
```

## 提示

1. 对于复杂功能，**从 planner 开始**
2. 合并前 **始终包含 code-reviewer**
3. 对于认证/支付/个人数据处理，**使用 security-reviewer**
4. **保持交接简洁** - 专注于下一个代理需要的内容
5. 必要时在代理之间 **运行验证**
