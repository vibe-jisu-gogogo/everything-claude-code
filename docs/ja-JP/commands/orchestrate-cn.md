# Orchestrate 命令

针对复杂任务的连续 agent 工作流。

## 使用方法

`/orchestrate [工作流类型] [任务说明]`

## 工作流类型

### feature
完整的功能实现工作流:
```
planner -> tdd-guide -> code-reviewer -> security-reviewer
```

### bugfix
bug 调查与修复工作流:
```
explorer -> tdd-guide -> code-reviewer
```

### refactor
安全的重构工作流:
```
architect -> code-reviewer -> tdd-guide
```

### security
以安全为重点的审查:
```
security-reviewer -> code-reviewer -> architect
```

## 执行模式

对于工作流中的每个 agent:

1. 使用来自前一个 agent 的上下文**调用 agent**
2. 将输出**收集**为结构化的交接文档
3. **传递**给链中的**下一个 agent**
4. 将结果**汇总**到最终报告

## 交接文档格式

在 agent 之间创建交接文档:

```markdown
## HANDOFF: [前一个 agent] -> [下一个 agent]

### 上下文
[执行内容的摘要]

### 发现事项
[重要发现或决策]

### 已修改文件
[已修改文件的列表]

### 未解决问题
[为下一个 agent 准备的未解决项目]

### 建议事项
[建议的下一步]
```

## 示例: 功能工作流

```
/orchestrate feature "Add user authentication"
```

执行以下步骤:

1. **Planner agent**
   - 分析需求
   - 创建实现计划
   - 识别依赖关系
   - 输出: `HANDOFF: planner -> tdd-guide`

2. **TDD Guide agent**
   - 读取 planner 的交接
   - 先编写测试
   - 实现以通过测试
   - 输出: `HANDOFF: tdd-guide -> code-reviewer`

3. **Code Reviewer agent**
   - 审查实现
   - 检查问题
   - 建议改进
   - 输出: `HANDOFF: code-reviewer -> security-reviewer`

4. **Security Reviewer agent**
   - 安全审计
   - 漏洞检查
   - 最终批准
   - 输出: 最终报告

## 最终报告格式

```
编排报告
====================
工作流: feature
任务: 添加用户认证
Agents: planner -> tdd-guide -> code-reviewer -> security-reviewer

摘要
-------
[1段摘要]

Agent 输出
-------------
Planner: [摘要]
TDD Guide: [摘要]
Code Reviewer: [摘要]
Security Reviewer: [摘要]

已修改文件
-------------
[列出所有已修改文件]

测试结果
------------
[测试通过/失败摘要]

安全状态
---------------
[安全发现事项]

建议事项
--------------
[可发布 / 需要修改 / 阻塞中]
```

## 并行执行

对于独立检查，并行执行 agents:

```markdown
### 并行阶段
同时执行:
- code-reviewer (质量)
- security-reviewer (安全)
- architect (设计)

### 结果合并
将输出合并为单个报告
```

## 参数

$ARGUMENTS:
- `feature <说明>` - 完整功能工作流
- `bugfix <说明>` - bug 修复工作流
- `refactor <说明>` - 重构工作流
- `security <说明>` - 安全审查工作流
- `custom <agent> <说明>` - 自定义 agent 序列

## 自定义工作流示例

```
/orchestrate custom "architect,tdd-guide,code-reviewer" "Redesign caching layer"
```

## 提示

1. 对于复杂功能**从 planner 开始**
2. 合并前**始终包含 code-reviewer**
3. 对于认证/支付/个人信息**使用 security-reviewer**
4. **保持交接简洁** - 聚焦于下一个 agent 需要的内容
5. 根据需要**在 agent 之间执行验证**
