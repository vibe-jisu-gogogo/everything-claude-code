# Django REST API — 项目 CLAUDE.md

> 一个真实的 Django REST Framework API 示例，使用 PostgreSQL 和 Celery。
> 复制到您的项目根目录并根据您的服务进行自定义。

## 项目概述

**技术栈：** Python 3.12+, Django 5.x, Django REST Framework, PostgreSQL, Celery + Redis, pytest, Docker Compose

**架构：** 面向领域的设计，按业务领域划分 apps。DRF 用于 API 层，Celery 用于异步任务，pytest 用于测试。所有端点返回 JSON — 不使用模板渲染。

## 关键规则

### Python 约定

- 所有函数签名使用 type hints — 使用 `from __future__ import annotations`
- 不使用 `print()` 语句 — 使用 `logging.getLogger(__name__)`
- 使用 f-strings 进行格式化，绝不使用 `%` 或 `.format()`
- 使用 `pathlib.Path` 而非 `os.path` 进行文件操作
- 使用 isort 排序导入：标准库、第三方库、本地库（由 ruff 强制执行）

### 数据库

- 所有查询使用 Django ORM — 仅在使用 `.raw()` 和参数化查询时使用原生 SQL
- 迁移文件在 git 中版本控制 — 生产环境绝不使用 `--fake`
- 使用 `select_related()` 和 `prefetch_related()` 防止 N+1 查询
- 所有模型必须有自动字段 `created_at` 和 `updated_at`
- 在用于 `filter()`、`order_by()` 或 `WHERE` 子句的任何字段上建立索引

```python
# BAD: N+1 query
orders = Order.objects.all()
for order in orders:
    print(order.customer.name)  # hits DB for each order

# GOOD: Single query with join
orders = Order.objects.select_related("customer").all()
```

### 认证

- 通过 `djangorestframework-simplejwt` 实现 JWT — access token (15 分钟) + refresh token (7 天)
- 在所有视图上使用 Permission classes — 绝不依赖默认设置
- 使用 `IsAuthenticated` 作为基础，并为对象级访问添加自定义权限
- 启用 Token 黑名单用于登出

### Serializers

- 简单 CRUD 使用 `ModelSerializer`，复杂验证使用 `Serializer`
- 当输入/输出不同时，分离读取和写入的 serializers
- 在 serializer 层验证，而非在 view 层 — views 应保持精简

```python
class CreateOrderSerializer(serializers.Serializer):
    product_id = serializers.UUIDField()
    quantity = serializers.IntegerField(min_value=1, max_value=100)

    def validate_product_id(self, value):
        if not Product.objects.filter(id=value, active=True).exists():
            raise serializers.ValidationError("Product not found or inactive")
        return value

class OrderDetailSerializer(serializers.ModelSerializer):
    customer = CustomerSerializer(read_only=True)
    product = ProductSerializer(read_only=True)

    class Meta:
        model = Order
        fields = ["id", "customer", "product", "quantity", "total", "status", "created_at"]
```

### 错误处理

- 使用 DRF exception handler 实现一致的错误响应
- 业务规则自定义异常放在 `core/exceptions.py`
- 绝不对客户端暴露内部错误详情

```python
# core/exceptions.py
from rest_framework.exceptions import APIException

class InsufficientStockError(APIException):
    status_code = 409
    default_detail = "Insufficient stock for this order"
    default_code = "insufficient_stock"
```

### 代码风格

- 代码或注释中不使用 emoji
- 最大行长度：120 字符（由 ruff 强制执行）
- 类：PascalCase，函数/变量：snake_case，常量：UPPER_SNAKE_CASE
- Views 精简 — 业务逻辑放在服务函数或模型方法中

## 文件结构

```
config/
  settings/
    base.py              # Shared settings
    local.py             # Dev overrides (DEBUG=True)
    production.py        # Production settings
  urls.py                # Root URL config
  celery.py              # Celery app configuration
apps/
  accounts/              # User auth, registration, profile
    models.py
    serializers.py
    views.py
    services.py          # Business logic
    tests/
      test_views.py
      test_services.py
      factories.py       # Factory Boy factories
  orders/                # Order management
    models.py
    serializers.py
    views.py
    services.py
    tasks.py             # Celery tasks
    tests/
  products/              # Product catalog
    models.py
    serializers.py
    views.py
    tests/
core/
  exceptions.py          # Custom API exceptions
  permissions.py         # Shared permission classes
  pagination.py          # Custom pagination
  middleware.py          # Request logging, timing
  tests/
```

## 关键模式

### 服务层

```python
# apps/orders/services.py
from django.db import transaction

def create_order(*, customer, product_id: uuid.UUID, quantity: int) -> Order:
    """Create an order with stock validation and payment hold."""
    product = Product.objects.select_for_update().get(id=product_id)

    if product.stock < quantity:
        raise InsufficientStockError()

    with transaction.atomic():
        order = Order.objects.create(
            customer=customer,
            product=product,
            quantity=quantity,
            total=product.price * quantity,
        )
        product.stock -= quantity
        product.save(update_fields=["stock", "updated_at"])

    # Async: send confirmation email
    send_order_confirmation.delay(order.id)
    return order
```

### View 模式

```python
# apps/orders/views.py
class OrderViewSet(viewsets.ModelViewSet):
    permission_classes = [IsAuthenticated]
    pagination_class = StandardPagination

    def get_serializer_class(self):
        if self.action == "create":
            return CreateOrderSerializer
        return OrderDetailSerializer

    def get_queryset(self):
        return (
            Order.objects
            .filter(customer=self.request.user)
            .select_related("product", "customer")
            .order_by("-created_at")
        )

    def perform_create(self, serializer):
        order = create_order(
            customer=self.request.user,
            product_id=serializer.validated_data["product_id"],
            quantity=serializer.validated_data["quantity"],
        )
        serializer.instance = order
```

### 测试模式 (pytest + Factory Boy)

```python
# apps/orders/tests/factories.py
import factory
from apps.accounts.tests.factories import UserFactory
from apps.products.tests.factories import ProductFactory

class OrderFactory(factory.django.DjangoModelFactory):
    class Meta:
        model = "orders.Order"

    customer = factory.SubFactory(UserFactory)
    product = factory.SubFactory(ProductFactory, stock=100)
    quantity = 1
    total = factory.LazyAttribute(lambda o: o.product.price * o.quantity)

# apps/orders/tests/test_views.py
import pytest
from rest_framework.test import APIClient

@pytest.mark.django_db
class TestCreateOrder:
    def setup_method(self):
        self.client = APIClient()
        self.user = UserFactory()
        self.client.force_authenticate(self.user)

    def test_create_order_success(self):
        product = ProductFactory(price=29_99, stock=10)
        response = self.client.post("/api/orders/", {
            "product_id": str(product.id),
            "quantity": 2,
        })
        assert response.status_code == 201
        assert response.data["total"] == 59_98

    def test_create_order_insufficient_stock(self):
        product = ProductFactory(stock=0)
        response = self.client.post("/api/orders/", {
            "product_id": str(product.id),
            "quantity": 1,
        })
        assert response.status_code == 409

    def test_create_order_unauthenticated(self):
        self.client.force_authenticate(None)
        response = self.client.post("/api/orders/", {})
        assert response.status_code == 401
```

## 环境变量

```bash
# Django
SECRET_KEY=
DEBUG=False
ALLOWED_HOSTS=api.example.com

# Database
DATABASE_URL=postgres://user:pass@localhost:5432/myapp

# Redis (Celery broker + cache)
REDIS_URL=redis://localhost:6379/0

# JWT
JWT_ACCESS_TOKEN_LIFETIME=15       # minutes
JWT_REFRESH_TOKEN_LIFETIME=10080   # minutes (7 days)

# Email
EMAIL_BACKEND=django.core.mail.backends.smtp.EmailBackend
EMAIL_HOST=smtp.example.com
```

## 测试策略

```bash
# Run all tests
pytest --cov=apps --cov-report=term-missing

# Run specific app tests
pytest apps/orders/tests/ -v

# Run with parallel execution
pytest -n auto

# Only failing tests from last run
pytest --lf
```

## ECC 工作流

```bash
# Planning
/plan "Add order refund system with Stripe integration"

# Development with TDD
/tdd                    # pytest-based TDD workflow

# Review
/python-review          # Python-specific code review
/security-scan          # Django security audit
/code-review            # General quality check

# Verification
/verify                 # Build, lint, test, security scan
```

## Git 流程

- `feat:` 新功能，`fix:` 修复 bug，`refactor:` 代码变更
- 功能分支从 `main` 创建，必须提交 PR
- CI：ruff (lint + format)，mypy (types)，pytest (tests)，safety (依赖检查)
- 部署：Docker 镜像，通过 Kubernetes 或 Railway 管理
