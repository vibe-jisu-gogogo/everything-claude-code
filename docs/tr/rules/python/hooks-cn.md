---
paths:
  - "**/*.py"
  - "**/*.pyi"
---
# Python Hooks

> 本文档扩展了 [common/hooks.md](../common/hooks.md)，添加了 Python 特有的内容。

## PostToolUse Hooks

在 `~/.claude/settings.json` 中配置：

- **black/ruff**: 每次 Edit 后自动格式化 `.py` 文件
- **mypy/pyright**: 编辑 `.py` 文件后运行类型检查

## 警告

- 当编辑的文件中存在 `print()` 语句时发出警告（应使用 `logging` 模块替代）
