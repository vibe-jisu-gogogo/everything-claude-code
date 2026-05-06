---
description: Comprehensive Python code review for PEP 8 compliance, type hints, security, and Pythonic idioms. Invokes the python-reviewer agent.
---

# Python 代码审查

此命令调用 **python-reviewer** agent 进行全面的 Python 特定代码审查。

## 此命令的功能

1. **识别 Python 变更**：通过 `git diff` 查找修改的 `.py` 文件
2. **运行静态分析**：执行 `ruff`、`mypy`、`pylint`、`black --check`
3. **安全扫描**：检查 SQL injection、command injection、unsafe deserialization
4. **类型安全审查**：分析 type hints 和 mypy 错误
5. **Pythonic 代码检查**：验证代码遵循 PEP 8 和 Python 最佳实践
6. **生成报告**：按严重程度对问题分类

## 何时使用

使用 `/python-review` 当：
- 编写或修改 Python 代码后
- 提交 Python 变更之前
- 审查包含 Python 代码的 pull requests
- 上手新的 Python 代码库
- 学习 Pythonic 模式和惯用法

## 审查分类

### CRITICAL（必须修复）
- SQL/Command injection 漏洞
- 不安全的 eval/exec 使用
- Pickle unsafe deserialization
- 硬编码凭证
- YAML unsafe load
- 隐藏错误的 bare except 子句

### HIGH（应该修复）
- 公共函数缺少 type hints
- Mutable default arguments
- 静默吞掉异常
- 资源未使用 context managers
- C 风格循环而非 comprehensions
- 使用 type() 而非 isinstance()
- 无锁的竞态条件

### MEDIUM（考虑修复）
- PEP 8 格式违规
- 公共函数缺少 docstrings
- Print 语句而非 logging
- 低效的字符串操作
- 无命名常量的魔术数字
- 不使用 f-strings 进行格式化
- 不必要的列表创建

## 运行的自动化检查

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
# Python 代码审查报告

## 已审查文件
- app/routes/user.py (已修改)
- app/services/auth.py (已修改)

## 静态分析结果
✓ ruff: 无问题
✓ mypy: 无错误
WARNING: black: 2 个文件需要重新格式化
✓ bandit: 无安全问题

## 发现的问题

[CRITICAL] SQL Injection 漏洞
文件: app/routes/user.py:42
问题: 用户输入直接插入 SQL 查询
```python
query = f"SELECT * FROM users WHERE id = {user_id}"  # 错误
```
修复: 使用参数化查询
```python
query = "SELECT * FROM users WHERE id = %s"  # 正确
cursor.execute(query, (user_id,))
```

[HIGH] Mutable default argument
文件: app/services/auth.py:18
问题: Mutable default argument 导致共享状态
```python
def process_items(items=[]):  # 错误
    items.append("new")
    return items
```
修复: 使用 None 作为默认值
```python
def process_items(items=None):  # 正确
    if items is None:
        items = []
    items.append("new")
    return items
```

[MEDIUM] 缺少 type hints
文件: app/services/auth.py:25
问题: 公共函数无 type annotations
```python
def get_user(user_id):  # 错误
    return db.find(user_id)
```
修复: 添加 type hints
```python
def get_user(user_id: str) -> Optional[User]:  # 正确
    return db.find(user_id)
```

[MEDIUM] 未使用 context manager
文件: app/routes/user.py:55
问题: 异常时文件未关闭
```python
f = open("config.json")  # 错误
data = f.read()
f.close()
```
修复: 使用 context manager
```python
with open("config.json") as f:  # 正确
    data = f.read()
```

## 摘要
- CRITICAL: 1
- HIGH: 1
- MEDIUM: 2

建议: FAIL: 阻止合并直到 CRITICAL 问题修复

## 需要格式化
运行: `black app/routes/user.py app/services/auth.py`
```

## 批准标准

| 状态 | 条件 |
|--------|-----------|
| PASS: 批准 | 无 CRITICAL 或 HIGH 问题 |
| WARNING: 警告 | 仅有 MEDIUM 问题（谨慎合并） |
| FAIL: 阻止 | 发现 CRITICAL 或 HIGH 问题 |

## 与其他命令集成

- 首先使用 `tdd-workflow` skill 确保测试通过
- 使用 `/code-review` 处理非 Python 特定问题
- 提交前使用 `/python-review`
- 如果静态分析工具失败，使用 `/build-fix`

## 框架特定审查

### Django 项目
审查者检查：
- N+1 查询问题（使用 `select_related` 和 `prefetch_related`）
- 模型变更缺少 migrations
- 可使用 ORM 时使用 raw SQL
- 多步操作缺少 `transaction.atomic()`

### FastAPI 项目
审查者检查：
- CORS 错误配置
- 用于请求验证的 Pydantic 模型
- Response 模型正确性
- 正确的 async/await 使用
- Dependency injection 模式

### Flask 项目
审查者检查：
- Context management（app context、request context）
- 正确的错误处理
- Blueprint 组织
- 配置管理

## 相关

- 代理: `agents/python-reviewer.md`
- Skills: `skills/python-patterns/`、`skills/python-testing/`

## 常见修复

### 添加 Type Hints
```python
# 之前
def calculate(x, y):
    return x + y

# 之后
from typing import Union

def calculate(x: Union[int, float], y: Union[int, float]) -> Union[int, float]:
    return x + y
```

### 使用 Context Managers
```python
# 之前
f = open("file.txt")
data = f.read()
f.close()

# 之后
with open("file.txt") as f:
    data = f.read()
```

### 使用 List Comprehensions
```python
# 之前
result = []
for item in items:
    if item.active:
        result.append(item.name)

# 之后
result = [item.name for item in items if item.active]
```

### 修复 Mutable Defaults
```python
# 之前
def append(value, items=[]):
    items.append(value)
    return items

# 之后
def append(value, items=None):
    if items is None:
        items = []
    items.append(value)
    return items
```

### 使用 f-strings (Python 3.6+)
```python
# 之前
name = "Alice"
greeting = "Hello, " + name + "!"
greeting2 = "Hello, {}".format(name)

# 之后
greeting = f"Hello, {name}!"
```

### 修复循环中的字符串拼接
```python
# 之前
result = ""
for item in items:
    result += str(item)

# 之后
result = "".join(str(item) for item in items)
```

## Python 版本兼容性

审查者会标注代码何时使用了较新 Python 版本的特性：

| 特性 | 最低 Python 版本 |
|---------|----------------|
| Type hints | 3.5+ |
| f-strings | 3.6+ |
| Walrus operator (`:=`) | 3.8+ |
| Position-only parameters | 3.8+ |
| Match statements | 3.10+ |
| Type unions (`x | None`) | 3.10+ |

确保项目的 `pyproject.toml` 或 `setup.py` 指定了正确的最低 Python 版本。
