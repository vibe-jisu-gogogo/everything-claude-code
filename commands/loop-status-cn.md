---
description: 检查活跃的 loop 状态、进度、失败信号以及建议的干预措施。
---

# Loop Status 命令

检查活跃的 loop 状态、进度和失败信号。

此斜杠命令只能在当前会话将其出队后运行。如果你需要检查卡住的或并行的会话，请从另一个终端运行打包的 CLI：

```bash
npx --package ecc-universal ecc loop-status --json
```

该 CLI 会扫描 `~/.claude/projects/**` 下的本地 Claude 会话记录 JSONL 文件，并报告过时的 `ScheduleWakeup` 调用或没有匹配 `tool_result` 的 `Bash` 工具调用。

## 使用方法

`/loop-status [--watch]`

## 报告内容

- 活跃的 loop 模式
- 当前阶段和上一次成功的检查点
- 失败的检查（如果有）
- 预估的时间/成本偏移
- 建议的干预措施（继续/暂停/停止）

## 跨会话 CLI

- `ecc loop-status --json` 为近期的本地 Claude 会话记录输出机器可读的状态。
- `ecc loop-status --home <dir>` 在检查另一个本地配置文件或挂载的工作区时扫描不同的主目录。
- `ecc loop-status --transcript <session.jsonl>` 直接检查单个会话记录文件。
- `ecc loop-status --bash-timeout-seconds 1800` 调整 Bash 调用过时的阈值。
- `ecc loop-status --exit-code` 当发现过时的 loop 或工具信号时退出码为 `2`，当无法扫描会话记录时退出码为 `1`。
- `--exit-code` 与 `--watch` 一起使用时需要指定 `--watch-count`，这样看门狗脚本就不会无限等待进程退出。
- `ecc loop-status --watch` 持续刷新状态直到被中断。
- `ecc loop-status --watch --watch-count 3 --exit-code` 刷新指定次数，然后以出现过的最高状态码退出。
- `ecc loop-status --watch --watch-count 3` 为脚本和任务交接输出有限次数的监控流。
- `ecc loop-status --watch --write-dir ~/.claude/loops` 为并行终端或看门狗脚本维护 `index.json` 和每个会话的 JSON 快照。

## 监控模式

当指定 `--watch` 时，会定期刷新状态。配合 `--json` 使用时，每次刷新会输出一行 JSON 对象，这样其他终端或脚本可以消费这个流。

## 快照文件

当有独立进程需要检查 loop 状态而无需等待当前 Claude 会话将 `/loop-status` 出队时，请使用 `--write-dir <dir>` 参数。CLI 会写入：

- `index.json`，包含每个被检查会话的一行记录。
- `<session-id>.json`，包含该会话的完整状态数据。

这些文件是本地会话记录分析的快照。它们不会控制或设置 Claude Code 运行时工具调用的超时。

## 参数

$ARGUMENTS:
- `--watch` 可选