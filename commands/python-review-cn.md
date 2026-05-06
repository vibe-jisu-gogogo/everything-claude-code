---
description: 针对 PEP 8 compliance、type hints、security 和 Pythonic idioms 的综合 Python code review。调用 python-reviewer agent。
---

# Python Code Review

此命令调用 **python-reviewer** agent 进行综合的 Python 特定代码审查。

## 此命令的功能

1. **识别 Python 更改**：通过 `git diff` 查找修改的 `.py` 文件
2. **运行静态分析**：执行 `ruff`、`mypy`、`pylint`、`black --check`
3. **Security Scan**：检查 SQL injection、command injection、unsafe deserialization
4. **Type Safety Review**：分析 type hints 和 mypy errors
5. **Pythonic Code Check**：验证代码遵循 PEP 8 和 Python 最佳实践
6. **生成报告**：按 severity 分类问题

## 何时使用

在以下情况下使用 `/python-review`：
- 编写或修改 Python 代码后
- 提交 Python 更改之前
- 审查包含 Python 代码的 pull request
- 加入新的 Python codebase 时
- 学习 Pythonic patterns 和 idioms 时

## 审查类别

### CRITICAL (必须修复)
- SQL/Command injection vulnerabilities
- 不安全的 eval/exec 使用
- Pickle unsafe deserialization
- 硬编码的 credentials
- YAML unsafe load
- Bare except clauses 隐藏错误

### HIGH (应该修复)
- Public functions 缺少 type hints
- Mutable default arguments
- 静默吞掉 exceptions
- 资源未使用 context managers
- C-style looping 而非 comprehensions
- 使用 type() 而非 isinstance()
- Race conditions without locks

### MEDIUM (考虑修复)
- PEP 8 formatting violations
- Public functions 缺少 docstrings
- 使用 print statements 而非 logging
- 低效的 string operations
- 未命名常量的 magic numbers
- 格式化未使用 f-strings
- 不必要的 list creation

## 运行的 Automated Checks

```bash
# Type checking
mypy .

# Linting and formatting
ruff check .
black --check .
isort --check-only .

# Security scanning
bandit -r .

# Dependency audit
pip-audit
safety check

# Testing
pytest --cov=app --cov-report=term-missing
```

## 使用示例

```text
User: /python-review

Agent:
# Python Code Review Report

## 审查的文件
- app/routes/user.py (modified)
- app/services/auth.py (modified)

## 静态分析结果
✓ ruff: 无问题
✓ mypy: 无 errors
WARNING: black: 2 个文件需要 reformatting
✓ bandit: 无 security issues

## 发现的问题

[CRITICAL] SQL Injection vulnerability
文件: app/routes/user.py:42
问题: 用户 input 直接 interpolated 到 SQL query 中
```python
query = f"SELECT * FROM users WHERE id = {user_id}"  # Bad
```
修复: 使用 parameterized query
```python
query = "SELECT * FROM users WHERE id = %s"  # Good
cursor.execute(query, (user_id,))
```

[HIGH] Mutable default argument
文件: app/services/auth.py:18
问题: Mutable default argument 导致 shared state
```python
def process_items(items=[]):  # Bad
    items.append("new")
    return items
```
修复: 使用 None 作为 default
```python
def process_items(items=None):  # Good
    if items is None:
        items = []
    items.append("new")
    return items
```

[MEDIUM] Missing type hints
文件: app/services/auth.py:25
问题: Public function 没有 type annotations
```python
def get_user(user_id):  # Bad
    return db.find(user_id)
```
修复: 添加 type hints
```python
def get_user(user_id: str) -> Optional[User]:  # Good
    return db.find(user_id)
```

[MEDIUM] Not using context manager
文件: app/routes/user.py:55
问题: File 在 exception 时未关闭
```python
f = open("config.json")  # Bad
data = f.read()
f.close()
```
修复: 使用 context manager
```python
with open("config.json") as f:  # Good
    data = f.read()
```

## Summary
- CRITICAL: 1
- HIGH: 1
- MEDIUM: 2

建议: FAIL: 在修复 CRITICAL issue 前阻止 merge

## 需要 Formatting
运行: `black app/routes/user.py app/services/auth.py`
```

## Approval Criteria

| Status | Condition |
|--------|-----------|
| PASS: 批准 | 无 CRITICAL 或 HIGH issues |
| WARNING: 警告 | 仅有 MEDIUM issues (谨慎 merge) |
| FAIL: 阻止 | 发现 CRITICAL 或 HIGH issues |

## 与其他命令的集成

- 首先使用 `tdd-workflow` skill 确保 tests 通过
- 使用 `/code-review` 处理非 Python 特定 concerns
- 在 commit 前使用 `/python-review`
- 如果静态分析 tools 失败，使用 `/build-fix`

## Framework-Specific Reviews

### Django Projects
Reviewer 检查：
- N+1 query issues (使用 `select_related` 和 `prefetch_related`)
- Model 更改缺少 migrations
- 可使用 ORM 时使用 raw SQL
- 多步操作缺少 `transaction.atomic()`

### FastAPI Projects
Reviewer 检查：
- CORS misconfiguration
- Pydantic models 用于 request validation
- Response models 正确性
- 正确的 async/await 使用
- Dependency injection patterns

### Flask Projects
Reviewer 检查：
- Context management (app context, request context)
- 正确的 error handling
- Blueprint organization
- Configuration management

## 相关内容

- Agent: `agents/python-reviewer.md`
- Skills: `skills/python-patterns/`, `skills/python-testing/`

## Common Fixes

### 添加 Type Hints
```python
# Before
def calculate(x, y):
    return x + y

# After
from typing import Union

def calculate(x: Union[int, float], y: Union[int, float]) -> Union[int, float]:
    return x + y
```

### 使用 Context Managers
```python
# Before
f = open("file.txt")
data = f.read()
f.close()

# After
with open("file.txt") as f:
    data = f.read()
```

### 使用 List Comprehensions
```python
# Before
result = []
for item in items:
    if item.active:
        result.append(item.name)

# After
result = [item.name for item in items if item.active]
```

### 修复 Mutable Defaults
```python
# Before
def append(value, items=[]):
    items.append(value)
    return items

# After
def append(value, items=None):
    if items is None:
        items = []
    items.append(value)
    return items
```

### 使用 f-strings (Python 3.6+)
```python
# Before
name = "Alice"
greeting = "Hello, " + name + "!"
greeting2 = "Hello, {}".format(name)

# After
greeting = f"Hello, {name}!"
```

### 修复 Loops 中的 String Concatenation
```python
# Before
result = ""
for item in items:
    result += str(item)

# After
result = "".join(str(item) for item in items)
```

## Python Version Compatibility

当代码使用较新 Python versions 的 features 时，reviewer 会指出：

| Feature | Minimum Python |
|---------|----------------|
| Type hints | 3.5+ |
| f-strings | 3.6+ |
| Walrus operator (`:=`) | 3.8+ |
| Position-only parameters | 3.8+ |
| Match statements | 3.10+ |
| Type unions (`x | None`) | 3.10+ |

确保项目的 `pyproject.toml` 或 `setup.py` 指定了正确的 minimum Python version。
