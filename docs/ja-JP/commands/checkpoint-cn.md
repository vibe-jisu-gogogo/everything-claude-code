# Checkpoint 命令

在工作流程中创建或验证检查点。

## 使用方法

`/checkpoint [create|verify|list] [name]`

## 创建 Checkpoint

创建检查点时：

1. 执行 `/verify quick` 确认当前状态为 clean
2. 使用检查点名称创建 git stash 或 commit
3. 将检查点记录到 `.claude/checkpoints.log`：

```bash
echo "$(date +%Y-%m-%d-%H:%M) | $CHECKPOINT_NAME | $(git rev-parse --short HEAD)" >> .claude/checkpoints.log
```

4. 报告检查点创建完成

## 验证 Checkpoint

针对检查点进行验证时：

1. 从日志中读取检查点

2. 将当前状态与检查点比较：
   * 检查点之后新增的文件
   * 检查点之后修改的文件
   * 当前测试成功率与当时的比较
   * 当前覆盖率与当时的比较

3. 报告：

```
Checkpoint 比较: $NAME
============================
变更文件: X
测试: +Y 合格 / -Z 失败
覆盖率: +X% / -Y%
构建: [PASS/FAIL]
```

## Checkpoint 列表显示

显示所有检查点，包括：

* 名称
* 时间戳
* Git SHA
* 状态（current、behind、ahead）

## 工作流程

典型的检查点流程：

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

* `create <name>` - 用指定名称创建检查点
* `verify <name>` - 验证指定名称的检查点
* `list` - 显示所有检查点
* `clear` - 删除旧检查点（保留最新 5 个）
