---
description: Create hooks to prevent unwanted behaviors from conversation analysis or explicit instructions
---

创建hook规则，通过分析对话模式或用户明确指令来防止Claude Code出现不需要的行为。

## 用法

`/hookify [要防止的行为描述]`

如果未提供参数，则分析当前对话以找出值得防止的行为。

## 工作流程

### 步骤1：收集行为信息

- 带参数时：解析用户对不需要行为的描述
- 不带参数时：使用`conversation-analyzer`代理查找：
  - 明确的纠正
  - 对重复错误的沮丧反应
  - 回滚的变更
  - 重复出现的类似问题

### 步骤2：展示发现结果

向用户展示：

- 行为描述
- 提议的事件类型
- 提议的模式或匹配器
- 提议的操作

### 步骤3：生成规则文件

对于每个已批准的规则，在`.claude/hookify.{name}.local.md`创建文件：

```yaml
---
name: rule-name
enabled: true
event: bash|file|stop|prompt|all
action: block|warn
pattern: "regex pattern"
---
Message shown when rule triggers.
```

### 步骤4：确认

报告已创建的规则，以及如何使用`/hookify-list`和`/hookify-configure`管理它们。