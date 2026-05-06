# Hooks 系统

## Hook 类型

- **PreToolUse**: Tool 执行前 (验证、参数修改)
- **PostToolUse**: Tool 执行后 (自动格式化、检查)
- **Stop**: Session 结束时 (最终验证)

## Auto-Accept 权限

谨慎使用：
- 对可靠、定义良好的计划启用
- 对探索性工作禁用
- 永远不要使用 dangerously-skip-permissions flag
- 而是在 `~/.claude.json` 中配置 `allowedTools`

## TodoWrite 最佳实践

将 TodoWrite tool 用于：
- 跟踪多步骤任务的进度
- 确认指令已被理解
- 启用实时指导
- 展示详细的实现步骤

Todo 列表会揭示：
- 异常步骤
- 缺失项
- 多余的不必要项
- 错误的细节级别
- 被误解的需求
