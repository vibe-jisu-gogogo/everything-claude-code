---
name: promote
description: 把项目范围的instincts提升到全局范围
command: true
---

# Promote 命令

将continuous-learning-v2中的instincts从项目范围提升到全局范围。

## 实现

使用插件根路径运行instinct CLI：

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/skills/continuous-learning-v2/scripts/instinct-cli.py" promote [instinct-id] [--force] [--dry-run]
```

或者如果`CLAUDE_PLUGIN_ROOT`未设置（手动安装）：

```bash
python3 ~/.claude/skills/continuous-learning-v2/scripts/instinct-cli.py promote [instinct-id] [--force] [--dry-run]
```

## 使用方法

```bash
/promote                      # 自动检测可提升的候选
/promote --dry-run            # 预览自动提升的候选
/promote --force              # 无需提示直接提升所有符合条件的候选
/promote grep-before-edit     # 从当前项目提升指定的instinct
```

## 执行步骤

1. 检测当前项目
2. 如果提供了`instinct-id`，仅提升该instinct（如果当前项目中存在）
3. 否则，寻找符合以下条件的跨项目候选：
   - 至少在2个项目中出现
   - 满足置信度阈值
4. 将已提升的instincts写入`~/.claude/homunculus/instincts/personal/`，并设置`scope: global`
