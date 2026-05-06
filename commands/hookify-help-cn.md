---
description: 获取 hookify 系统的帮助
---

显示完整的 hookify 文档。

## Hook 系统概述

Hookify 创建规则文件，与 Claude Code 的 hook 系统集成以防止不必要的行为。

### 事件类型

- `bash`：在使用 Bash 工具时触发，并匹配命令模式
- `file`：在使用 Write/Edit 工具时触发，并匹配文件路径
- `stop`：在会话结束时触发
- `prompt`：在用户提交消息时触发，并匹配输入模式
- `all`：在所有事件上触发

### 规则文件格式

文件存储为 `.claude/hookify.{name}.local.md`：

```yaml
---
name: descriptive-name
enabled: true
event: bash|file|stop|prompt|all
action: block|warn
pattern: "regex pattern to match"
---
规则触发时显示的消息。
支持多行。
```

### 命令

- `/hookify [description]` 创建新规则，在未提供描述时自动分析会话
- `/hookify-list` 列出已配置的规则
- `/hookify-configure` 启用或禁用规则

### 模式技巧

- 使用 regex 语法
- 对于 `bash`，匹配完整命令字符串
- 对于 `file`，匹配文件路径
- 部署前测试模式
