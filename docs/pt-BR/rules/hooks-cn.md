# Hooks 系统

## Hook 类型

- **PreToolUse**: 工具执行前（验证、参数修改）
- **PostToolUse**: 工具执行后（自动格式化、检查）
- **Stop**: 会话结束时（最终检查）

## 自动接受权限

谨慎使用：
- 为可靠且定义良好的计划启用
- 为探索性工作禁用
- 永远不要使用 dangerously-skip-permissions 标志
- 改为在 `~/.claude.json` 中配置 `allowedTools`

## TodoWrite 最佳实践

使用 TodoWrite 工具来：
- 跟踪多步骤任务的进度
- 验证对指令的理解
- 启用实时指导
- 显示细粒度的实施步骤

任务列表会揭示：
- 顺序错误的步骤
- 缺失的项目
- 不必要的额外项目
- 不正确的粒度
- 被误解的需求
