---
paths:
  - "**/*.py"
  - "**/*.pyi"
---
# Python 模式

> 本文件扩展了 [common/patterns.md](../common/patterns.md)，添加了 Python 特定内容。

## Protocol (Duck Typing)

```python
from typing import Protocol

class Repository(Protocol):
    def find_by_id(self, id: str) -> dict | None: ...
    def save(self, entity: dict) -> dict: ...
```

## Dataclass 作为 DTO

```python
from dataclasses import dataclass

@dataclass
class CreateUserRequest:
    name: str
    email: str
    age: int | None = None
```

## Context Managers & Generators

- 使用 context managers (`with` 语句) 进行资源管理
- 使用 generators 进行惰性求值和内存高效迭代

## Reference

有关完整的模式（包括 decorators、concurrency 和包组织），请参阅 skill: `python-patterns` 文件。
