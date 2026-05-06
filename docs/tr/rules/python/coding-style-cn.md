---
paths:
  - "**/*.py"
  - "**/*.pyi"
---
# Python 编码风格

> 本文件基于 [common/coding-style.md](../common/coding-style.md) 扩展了 Python 特定内容。

## 标准

- 遵循 **PEP 8** 规范
- 在所有函数签名中使用 **type annotations**

## Immutability

优先使用 Immutable 数据结构：

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class User:
    name: str
    email: str

from typing import NamedTuple

class Point(NamedTuple):
    x: float
    y: float
```

## 格式化

- 使用 **black** 进行代码格式化
- 使用 **isort** 进行导入排序
- 使用 **ruff** 进行 linting

## 参考

有关全面的 Python idioms 和 patterns，请参阅 skill: `python-patterns` 文件。