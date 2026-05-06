---
paths:
  - "**/*.py"
  - "**/*.pyi"
---
# Python 安全

> 本文档扩展了 [common/security.md](../common/security.md)，添加了 Python 特定的内容。

## Secret 管理

```python
import os
from dotenv import load_dotenv

load_dotenv()

api_key = os.environ["OPENAI_API_KEY"]  # 如果缺失会抛出 KeyError
```

## 安全扫描

- 使用 **bandit** 进行静态安全分析：
  ```bash
  bandit -r src/
  ```

## 参考

对于 Django 特定的安全规则（如果适用），请查看 skill: `django-security` 文件。
