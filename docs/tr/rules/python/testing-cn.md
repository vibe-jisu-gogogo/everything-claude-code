---
paths:
  - "**/*.py"
  - "**/*.pyi"
---
# Python Testing

> 此文件使用 Python 特定内容扩展了 [common/testing.md](../common/testing.md) 文件。

## Framework

使用 **pytest** 作为测试框架。

## Coverage

```bash
pytest --cov=src --cov-report=term-missing
```

## Test Organization

使用 `pytest.mark` 进行测试分类：

```python
import pytest

@pytest.mark.unit
def test_calculate_total():
    ...

@pytest.mark.integration
def test_database_connection():
    ...
```

## Reference

详细的 pytest patterns 和 fixtures 请查看 skill: `python-testing` 文件。
