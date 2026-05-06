# Orchestrate 命令

用于复杂任务的顺序代理工作流程。

## 用法

`/orchestrate [workflow-type] [task-description]`

## 工作流程类型

### feature
完整功能实现工作流程：
```
planner -> tdd-guide -> code-reviewer -> security-reviewer
```

### bugfix
bug 调查与修复工作流程：
```
planner -> tdd-guide -> code-reviewer
```

### refactor
安全重构工作流程：
```
architect -> code-reviewer -> tdd-guide
```

### security
以安全为中心的审查：
```
security-reviewer -> code-reviewer -> architect
```

## 执行模式

对于工作流程中的每个代理：

1. 使用上一个代理的上下文**调用代理**
2. 通过结构化交接文档**收集输出**
3. 传递给链中的**下一个代理**
4. **综合结果**并编写最终报告

## 交接文档格式

在代理之间生成交接文档：

```markdown
## HANDOFF: [前一代理] -> [下一代理]

### Context
[已执行工作摘要]

### Findings
[主要发现或决定事项]

### Files Modified
[修改的文件列表]

### Open Questions
[为下一个代理准备的待解决项目]

### Recommendations
[建议的下一步]
```

## 示例：Feature 工作流程

```
/orchestrate feature "Add user authentication"
```

执行顺序：

1. **Planner 代理**
   - 需求分析
   - 制定实现计划
   - 识别依赖
   - 输出：`HANDOFF: planner -> tdd-guide`

2. **TDD Guide 代理**
   - 读取 planner 交接
   - 先写测试
   - 实现以通过测试
   - 输出：`HANDOFF: tdd-guide -> code-reviewer`

3. **Code Reviewer 代理**
   - 实现审查
   - 确认问题
   - 提出改进建议
   - 输出：`HANDOFF: code-reviewer -> security-reviewer`

4. **Security Reviewer 代理**
   - 安全审计
   - 检查漏洞
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
[所有修改的文件列表]

TEST RESULTS
------------
[测试通过/失败摘要]

SECURITY STATUS
---------------
[安全发现事项]

RECOMMENDATION
--------------
[SHIP / NEEDS WORK / BLOCKED]
```

## 并行执行

对于独立检查，将并行执行代理：

```markdown
### Parallel Phase
同时执行：
- code-reviewer（质量）
- security-reviewer（安全）
- architect（设计）

### Merge Results
将输出合并为单个报告
```

## 参数

$ARGUMENTS:
- `feature <description>` - 完整功能工作流程
- `bugfix <description>` - bug 修复工作流程
- `refactor <description>` - 重构工作流程
- `security <description>` - 安全审查工作流程
- `custom <agents> <description>` - 自定义代理顺序

## 自定义工作流程示例

```
/orchestrate custom "architect,tdd-guide,code-reviewer" "Redesign caching layer"
```

## 提示

1. 对于复杂功能，**从 planner 开始**
2. merge 前**始终包含 code-reviewer**
3. 对于认证/支付/个人信息处理，**使用 security-reviewer**
4. **保持交接简洁** - 专注于下一个代理需要的内容
5. 必要时在代理之间**执行验证**
