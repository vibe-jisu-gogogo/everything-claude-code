# 故障排除

社区报告的当前 Claude Code bugs 的解决方案，这些问题可能影响 ECC 用户。

这些是上游 Claude Code 的行为，不是 ECC 的 bugs。以下条目总结了在 Claude Code `v2.1.79`（macOS，大量 hooks 使用，启用 MCP connectors）上收集的、经过生产测试的变通方法，详见 [issue #644](https://github.com/affaan-m/everything-claude-code/issues/644)。在上游修复发布前，将这些作为实用的临时解决方案。

## Claude Code 已知 bugs 的社区解决方案

### hooks 执行成功但显示"Hook Error"标签

**症状：** Hook 执行成功，但 Claude Code 仍在对话记录中显示 `Hook Error`。

**解决方案：**

- 在 hook 开始时消费 stdin（shell hooks 中使用 `input=$(cat)`），这样父进程就不会看到未消费的管道。
- 对于简单的 allow/block hooks，将人类可读的诊断信息发送到 stderr，并保持 stdout 静默，除非你的 hook 实现明确需要结构化 stdout。
- 当子进程的 stderr 输出没有实际意义时，重定向它。
- 使用正确的退出代码：`0` 表示允许，`2` 表示阻止，其他非零退出视为错误。

**示例：**

```bash
# 好的做法：用 stderr 消息和 exit 2 阻止
input=$(cat)
echo "[BLOCKED] Reason here" >&2
exit 2
```

### `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` 导致的提前 compaction

**症状：** 降低 `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` 导致 compaction 更早发生，而不是更晚。

**解决方案：**

- 在当前的某些 Claude Code 构建中，较低的值可能会降低 compaction 阈值而不是延长它。
- 如果你想要更多工作空间，移除 `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE`，并在逻辑任务边界使用手动 `/compact`。
- 使用 ECC 的 `strategic-compact` 指南，而不是强制降低自动 compaction 阈值。

### MCP connectors 看起来已连接，但在 compaction 后失败

**症状：** Gmail 或 Google Drive MCP tools 在 compaction 后失败，即使 connector 在 UI 中看起来仍然已认证。

**解决方案：**

- Compaction 后，将受影响的 connector 关闭后再重新打开。
- 如果你的 Claude Code 构建支持，添加一个 `PostCompact` 提醒 hook，警告你在 compaction 后重新检查 connector 认证状态。
- 将此视为认证状态恢复步骤，而不是永久修复。

### Hook 编辑不会热重载

**症状：** 对 `settings.json` hooks 的更改不会生效，直到会话重启。

**解决方案：**

- 更改 hooks 后重启 Claude Code 会话。
- 高级用户有时会围绕 `kill -HUP $PPID` 编写一个本地 `/reload` 命令脚本，但 ECC 不提供该功能，因为它依赖于 shell，并非普遍可靠。

### 重复的 `529 Overloaded` 响应

**症状：** 在高 hook/tool/context 压力下，Claude Code 开始失败。

**解决方案：**

- 如果你的设置支持，使用 `ENABLE_TOOL_SEARCH=auto:5` 减少 tool 定义的压力。
- 对于日常工作，降低 `MAX_THINKING_TOKENS`。
- 如果你的设置暴露该选项，将 subagent 工作路由到更便宜的模型，例如 `CLAUDE_CODE_SUBAGENT_MODEL=haiku`。
- 每个项目禁用未使用的 MCP servers。
- 在自然断点手动 compact，而不是等待自动 compaction。

## 相关 ECC 文档

- [hook-bug-workarounds.md](./hook-bug-workarounds.md) 用于更简短的 hook/compaction/MCP 恢复检查清单。
- [hooks/README.md](../hooks/README.md) 用于 ECC 记录的 hook 生命周期和退出代码行为。
- [token-optimization.md](./token-optimization.md) 用于成本和上下文管理设置。
- [issue #644](https://github.com/affaan-m/everything-claude-code/issues/644) 用于原始报告和测试环境。