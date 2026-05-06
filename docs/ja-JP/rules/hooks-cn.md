# Hooks 系统

## Hook 类型

- **PreToolUse**: 工具执行前（验证、参数修改）
- **PostToolUse**: 工具执行后（自动格式化、检查）
- **Stop**: 会话结束时（最终验证）

## 自动批准权限

谨慎使用：
- 针对可靠、明确定义的计划启用
- 在探索性工作中禁用
- 绝对不要使用 dangerously-skip-permissions 标志
- 改为在 `~/.claude.json` 中设置 `allowedTools`

## TodoWrite 最佳实践

使用 TodoWrite 工具：
- 跟踪多步骤任务的进度
- 验证对指令的理解
- 允许实时调整
- 显示详细的实现步骤

Todo 列表可以揭示：
- 顺序错误的步骤
- 缺失的项目
- 不必要的多余项目
- 粒度错误
- 被误解的需求
