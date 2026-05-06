# Hook Bug 解决方案

针对当前 Claude Code 中可能影响 ECC 重度 Hook 配置的 Bug，社区验证过的解决方案。

本文档目的明确：从更广泛的故障排查范围中收集最有效的操作修复方案，不重复推测性或不受支持的配置建议。这些是上游 Claude Code 的行为问题，不是 ECC 的 Bug。

## 何时使用本文档

当你专门调试以下问题时使用本文档：

- Hook 运行成功但显示错误的 `Hook Error` 标签
- 提前发生的上下文压缩（compaction）
- MCP 连接器看似已认证但在压缩后失败
- Hook 编辑后没有热重载
- 在重度 Hook/Tool 压力下重复出现 `529 Overloaded` 响应

完整的 ECC 故障排查请参阅 [TROUBLESHOOTING.md](./TROUBLESHOOTING-cn.md)。

## 高效解决方案

### 错误的 `Hook Error` 标签

有效方法：

- 在 Shell Hook 开始时读取 stdin（`input=$(cat)`）。
- 对于简单的允许/阻止 Hook，保持 stdout 安静，除非你的 Hook 明确需要结构化 stdout。
- 将人类可读的诊断信息发送到 stderr。
- 使用正确的退出码：`0` 允许，`2` 阻止，其他非零值视为错误。

```bash
input=$(cat)
echo "[BLOCKED] Reason here" >&2
exit 2
```

### 提前发生的上下文压缩（Compaction）

有效方法：

- 如果降低 `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` 导致构建中提前压缩，请移除该设置。
- 优先在自然任务边界使用手动 `/compact`。
- 使用 ECC 的 `strategic-compact` 指导，而不是强制降低阈值。

### MCP 认证看似有效但压缩后失败

有效方法：

- 压缩后关闭并重新打开受影响的连接器。
- 如果你的 Claude Code 构建支持，添加一个轻量级的 `PostCompact` 提醒 Hook，告诉你重新检查连接器认证。
- 将此视为恢复提醒，而不是永久修复。

### Hook 编辑后没有热重载

有效方法：

- 更改 Hook 后重启 Claude Code 会话。
- 高级用户有时使用 Shell 本地重载助手，但 ECC 不提供此类工具，因为这些方法依赖于特定 Shell 和平台。

### 重复的 `529 Overloaded`

有效方法：

- 如果你的配置支持，使用 `ENABLE_TOOL_SEARCH=auto:5` 减少工具定义压力。
- 降低常规工作的 `MAX_THINKING_TOKENS`。
- 如果你的配置暴露该选项，将子代理工作路由到更便宜的模型，例如 `CLAUDE_CODE_SUBAGENT_MODEL=haiku`。
- 按项目禁用未使用的 MCP 服务器。
- 在自然断点处手动压缩，而不是等待自动压缩。

## 相关 ECC 文档

- [TROUBLESHOOTING.md](./TROUBLESHOOTING-cn.md)
- [token-optimization.md](./token-optimization-cn.md)
- [hooks/README.md](../hooks/README-cn.md)
- [issue #644](https://github.com/affaan-m/everything-claude-code/issues/644)
