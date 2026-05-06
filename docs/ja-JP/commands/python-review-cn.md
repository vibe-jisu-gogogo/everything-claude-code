---
description: 基于 PEP 8 规范、类型提示、安全性和 Pythonic 惯用法的全面 Python 代码审查。调用 python-reviewer 代理。
---

# Python Code Review

此命令调用 **python-reviewer** 代理进行全面的 Python 特定代码审查。

## 此命令的功能

1. **识别 Python 变更**: 用 `git diff` 检测已修改的 `.py` 文件
2. **执行静态分析**: 运行 `ruff`、`mypy`、`pylint`、`black --check`
3. **安全扫描**: 检查 SQL 注入、命令注入和不安全反序列化
4. **类型安全审查**: 分析类型提示和 mypy 错误
5. **Pythonic 代码检查**: 确保代码遵循 PEP 8 和 Python 最佳实践
6. **生成报告**: 按严重程度分类问题

## 使用时机

在以下情况下使用 `/python-review`:
- 创建或修改 Python 代码后
- 提交 Python 变更之前
- 审查包含 Python 代码的 PR 时
- 上手新 Python 代码库时
- 学习 Pythonic 模式和惯用法时

## 审查类别

### CRITICAL (必须修正)
- SQL/命令注入漏洞
- 不安全使用 eval/exec
- Pickle 不安全反序列化
- 硬编码凭据
- YAML 不安全加载
- 隐藏错误的裸 except 子句

### HIGH (推荐修正)
- 公共函数缺少类型提示
- 可变默认参数
- 静默吞掉异常
- 资源未使用上下文管理器
- 用 C 风格循环代替推导式
- 使用 type() 而非 isinstance()
- 无锁的竞态条件

### MEDIUM (考虑修正)
- PEP 8 格式违规
- 公共函数缺少 docstring
- 使用 print 语句而非 logging
- 低效字符串操作
- 无命名常量的魔法数字
- 格式化未使用 f-strings
- 不必要的列表创建

## 执行的自动检查

```bash
# 类型检查
mypy .

# 代码检查和格式化
ruff check .
black --check .
isort --check-only .

# 安全扫描
bandit -r .

# 依赖审计
pip-audit
safety check

# 测试
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

[CRITICAL] SQL 注入漏洞
文件: app/routes/user.py:42
问题: 用户输入直接插入到 SQL 查询中
```python
query = f"SELECT * FROM users WHERE id = {user_id}"  # 错误
```
修正: 使用参数化查询
```python
query = "SELECT * FROM users WHERE id = %s"  # 正确
cursor.execute(query, (user_id,))
```

[HIGH] 可变默认参数
文件: app/services/auth.py:18
问题: 可变默认参数导致共享状态
```python
def process_items(items=[]):  # 错误
    items.append("new")
    return items
```
修正: 使用 None 作为默认值
```python
def process_items(items=None):  # 正确
    if items is None:
        items = []
    items.append("new")
    return items
```

[MEDIUM] 缺少类型提示
文件: app/services/auth.py:25
问题: 公共函数没有类型注解
```python
def get_user(user_id):  # 错误
    return db.find(user_id)
```
修正: 添加类型提示
```python
def get_user(user_id: str) -> Optional[User]:  # 正确
    return db.find(user_id)
```

[MEDIUM] 未使用上下文管理器
文件: app/routes/user.py:55
问题: 异常时文件不会关闭
```python
f = open("config.json")  # 错误
data = f.read()
f.close()
```
修正: 使用上下文管理器
```python
with open("config.json") as f:  # 正确
    data = f.read()
```

## 摘要
- CRITICAL: 1
- HIGH: 1
- MEDIUM: 2

建议: FAIL: 在 CRITICAL 问题修复之前阻止合并

## 需要格式化
执行: `black app/routes/user.py app/services/auth.py`
```

## 批准标准

| 状态 | 条件 |
|--------|-----------|
| PASS: 批准 | 无 CRITICAL 或 HIGH 问题 |
| WARNING: 警告 | 仅有 MEDIUM 问题 (谨慎合并) |
| FAIL: 阻止 | 发现 CRITICAL 或 HIGH 问题 |

## 与其他命令集成

- 首先使用 `/python-test` 确保测试通过
- 使用 `/code-review` 处理非 Python 特定问题
- 提交前使用 `/python-review`
- 静态分析工具失败时使用 `/build-fix`

## 框架特定审查

### Django 项目
审查者会检查:
- N+1 查询问题 (使用 `select_related` 和 `prefetch_related`)
- 模型变更缺少迁移
- 在可以使用 ORM 的地方使用原生 SQL
- 多步骤操作缺少 `transaction.atomic()`

### FastAPI 项目
审查者会检查:
- CORS 错误配置
- 请求验证的 Pydantic 模型
- 响应模型准确性
- 正确的 async/await 使用
- 依赖注入模式

### Flask 项目
审查者会检查:
- 上下文管理 (app 上下文、request 上下文)
- 正确的错误处理
- Blueprint 结构
- 配置管理

## 相关内容

- 代理: `agents/python-reviewer.md`
- 技能: `skills/python-patterns/`、`skills/python-testing/`

## 常见修正

### 添加类型提示
```python
# 修改前
def calculate(x, y):
    return x + y

# 修改后
from typing import Union

def calculate(x: Union[int, float], y: Union[int, float]) -> Union[int, float]:
    return x + y
```

### 使用上下文管理器
```python
# 修改前
f = open("file.txt")
data = f.read()
f.close()

# 修改后
with open("file.txt") as f:
    data = f.read()
```

### 使用列表推导式
```python
# 修改前
result = []
for item in items:
    if item.active:
        result.append(item.name)

# 修改后
result = [item.name for item in items if item.active]
```

### 修正可变默认值
```python
# 修改前
def append(value, items=[]):
    items.append(value)
    return items

# 修改后
def append(value, items=None):
    if items is None:
        items = []
    items.append(value)
    return items
```

### 使用 f-strings (Python 3.6+)
```python
# 修改前
name = "Alice"
greeting = "Hello, " + name + "!"
greeting2 = "Hello, {}".format(name)

# 修改后
greeting = f"Hello, {name}!"
```

### 修正循环中的字符串连接
```python
# 修改前
result = ""
for item in items:
    result += str(item)

# 修改后
result = "".join(str(item) for item in items)
```

## Python 版本兼容性

如果代码使用新版 Python 功能，审查者会发出通知:

| 功能 | 最低 Python 版本 |
|---------|----------------|
| 类型提示 | 3.5+ |
| f-strings | 3.6+ |
| 海象运算符 (`:=`) | 3.8+ |
| 位置专用参数 | 3.8+ |
| Match 语句 | 3.10+ |
| 类型联合 (`x | None`) | 3.10+ |

请确保项目的 `pyproject.toml` 或 `setup.py` 指定了正确的最低 Python 版本。
