---
description: 使用安全默认值和明确的停止条件启动托管式自主循环模式。
---

# Loop Start 命令

使用安全默认值启动托管式自主循环模式。

## 使用方法

`/loop-start [pattern] [--mode safe|fast]`

- `pattern`: `sequential`, `continuous-pr`, `rfc-dag`, `infinite`
- `--mode`:
  - `safe` (默认): 严格的质量关卡和检查点
  - `fast`: 减少关卡以提升速度

## 流程

1. 确认仓库状态和分支策略。
2. 选择循环模式和模型层级策略。
3. 为选定的模式启用所需的hooks/profile。
4. 创建循环计划并将运行手册写入`.claude/plans/`目录下。
5. 打印启动和监控循环的命令。

## 必需的安全检查

- 在第一次循环迭代前验证测试通过。
- 确保`ECC_HOOK_PROFILE`没有被全局禁用。
- 确保循环有明确的停止条件。

## 参数

$ARGUMENTS:
- `<pattern>` 可选 (`sequential|continuous-pr|rfc-dag|infinite`)
- `--mode safe|fast` 可选
