---
name: checkpoint
description: 在工作流中创建、验证、查询或清理 checkpoint。
---

# Checkpoint 命令

在工作流中创建或验证 checkpoint。

## 使用方法

`/checkpoint [create|verify|list|clear] [name]`

## 创建 Checkpoint

创建 Checkpoint 时：

1. 执行 `/verify quick` 确认当前状态干净
2. 使用 checkpoint 名称创建 git stash 或 commit
3. 在 `.claude/checkpoints.log` 中记录 checkpoint：

```bash
echo "$(date +%Y-%m-%d-%H:%M) | $CHECKPOINT_NAME | $(git rev-parse --short HEAD)" >> .claude/checkpoints.log
```

4. 报告 Checkpoint 创建完成

## 验证 Checkpoint

与 Checkpoint 对照验证时：

1. 从日志中读取 checkpoint
2. 比较当前状态与 checkpoint：
   - Checkpoint 之后新增的文件
   - Checkpoint 之后修改的文件
   - 当前与当时的测试通过率
   - 当前与当时的覆盖率

3. 报告：
```
CHECKPOINT COMPARISON: $NAME
============================
Files changed: X
Tests: +Y passed / -Z failed
Coverage: +X% / -Y%
Build: [PASS/FAIL]
```

## Checkpoint 列表

显示所有 checkpoint 及以下信息：
- 名称
- 时间戳
- Git SHA
- 状态 (current, behind, ahead)

## 工作流

典型的 checkpoint 流程：

```
[开始] --> /checkpoint create "feature-start"
   |
[实现] --> /checkpoint create "core-done"
   |
[测试] --> /checkpoint verify "core-done"
   |
[重构] --> /checkpoint create "refactor-done"
   |
[PR] --> /checkpoint verify "feature-start"
```

## 参数

$ARGUMENTS:
- `create <name>` - 创建指定名称的 checkpoint
- `verify <name>` - 验证指定名称的 checkpoint
- `list` - 显示所有 checkpoint
- `clear` - 清除旧 checkpoint（仅保留最近 5 个）
