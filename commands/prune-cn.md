---
name: prune
description: 删除超过30天从未被提升的待处理本能记录
command: true
---

# 清理待处理本能记录

移除自动生成但从未被审核或提升的过期待处理本能记录。

## 实现方式

使用插件根路径运行instinct CLI:

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/skills/continuous-learning-v2/scripts/instinct-cli.py" prune
```

如果未设置`CLAUDE_PLUGIN_ROOT`(手动安装):

```bash
python3 ~/.claude/skills/continuous-learning-v2/scripts/instinct-cli.py prune
```

## 使用方法

```
/prune                    # 删除超过30天的本能记录
/prune --max-age 60      # 自定义时间阈值(天)
/prune --dry-run         # 预览删除操作而不实际执行
```
