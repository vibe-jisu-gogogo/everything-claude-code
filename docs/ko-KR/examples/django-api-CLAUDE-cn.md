# Django REST API — 项目 CLAUDE.md

> 使用 PostgreSQL 和 Celery 的 Django REST Framework API 实战示例。
> 复制到项目根目录并根据服务进行自定义。

## 项目概述

**技术栈:** Python 3.12+, Django 5.x, Django REST Framework, PostgreSQL, Celery + Redis, pytest, Docker Compose

**架构:** 按业务领域应用组成的领域驱动设计。API 层使用 DRF，异步任务使用 Celery，测试使用 pytest。所有端点返回 JSON，不使用模板渲染。

## 必备规则

### Python 规则

- 所有函数签名使用 type hints — 使用 `from __future__ import annotations`
- 禁止使用 `print()` 语句 — 使用 `logging.getLogger(__name__)`
- 字符串格式化使用 f-strings，禁止使用 `%` 或 `.format()`
- 文件操作使用 `pathlib.Path` 代替 `os.path`
- 使用 isort 排序 import: 标准库、第三方库、本地顺序 (由 ruff 强制)

### 数据库

- 所有查询使用 Django ORM — raw SQL 仅通过 `.raw()` 和参数化查询使用
- 迁移文件提交到 git — 生产环境禁止使用 `--fake`
- 为防止 N+1 查询使用 `select_related()` 和 `prefetch_related()`
- 所有模型必须包含 `created_at` 和 `updated_at` 自动字段
- 所有在 `filter()`, `order_by()`, 或 `WHERE` 子句中使用的字段添加索引

```python
# 坏例子: N+1 查询
orders = Order.objects.all()
for order in orders:
    print(order.customer.name)  # 每个订单都查询数据库

# 好例子: 使用 join 的单次查询
orders = Order.objects.select_related("customer").all()
```

### 认证

- 通过 `djangorestframework-simplejwt` 实现 JWT — access token (15分钟) + refresh token (7天)
- 所有视图指定 permission 类 — 不依赖默认值
- `IsAuthenticated` 作为默认，对象级访问添加自定义 permission
- 启用 token blacklisting 用于登出

### Serializers

- 简单 CRUD 使用 `ModelSerializer`，复杂验证使用 `Serializer`
- 输入/输出形式不同时分离读写 serializer
- 验证在 serializer 层进行 — 保持视图精简

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
- 业务逻辑用的自定义异常在 `core/exceptions.py` 定义
- 不向客户端暴露内部错误详情

```python
# core/exceptions.py
from rest_framework.exceptions import APIException

class InsufficientStockError(APIException):
    status_code = 409
    default_detail = "Insufficient stock for this order"
    default_code = "insufficient_stock"
```

### 代码风格

- 代码或注释中禁止使用 emoji
- 最大行长度: 120 字符 (由 ruff 强制)
- 类: PascalCase, 函数/变量: snake_case, 常量: UPPER_SNAKE_CASE
- 保持视图精简 — 业务逻辑放置在服务函数或模型方法中

## 文件结构

```
config/
  settings/
    base.py              # 共享配置
    local.py             # 开发环境覆盖 (DEBUG=True)
    production.py        # 生产环境配置
  urls.py                # 根 URL 配置
  celery.py              # Celery 应用配置
apps/
  accounts/              # 用户认证、注册、个人资料
    models.py
    serializers.py
    views.py
    services.py          # 业务逻辑
    tests/
      test_views.py
      test_services.py
      factories.py       # Factory Boy 工厂
  orders/                # 订单管理
    models.py
    serializers.py
    views.py
    services.py
    tasks.py             # Celery 任务
    tests/
  products/              # 商品目录
    models.py
    serializers.py
    views.py
    tests/
core/
  exceptions.py          # 自定义 API 异常
  permissions.py         # 共享 permission 类
  pagination.py          # 自定义分页
  middleware.py          # 请求日志、计时
  tests/
```

## 主要模式

### Service 层

```python
# apps/orders/services.py
from django.db import transaction

def create_order(*, customer, product_id: uuid.UUID, quantity: int) -> Order:
    """包含库存验证和支付预留的订单创建。"""
    with transaction.atomic():
        product = Product.objects.select_for_update().get(id=product_id)

        if product.stock < quantity:
            raise InsufficientStockError()

        order = Order.objects.create(
            customer=customer,
            product=product,
            quantity=quantity,
            total=product.price * quantity,
        )
        product.stock -= quantity
        product.save(update_fields=["stock", "updated_at"])

    # 异步: 发送订单确认邮件
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

# 数据库
DATABASE_URL=postgres://user:pass@localhost:5432/myapp

# Redis (Celery broker + 缓存)
REDIS_URL=redis://localhost:6379/0

# JWT
JWT_ACCESS_TOKEN_LIFETIME=15       # 分钟
JWT_REFRESH_TOKEN_LIFETIME=10080   # 分钟 (7天)

# 电子邮件
EMAIL_BACKEND=django.core.mail.backends.smtp.EmailBackend
EMAIL_HOST=smtp.example.com
```

## 测试策略

```bash
# 运行全部测试
pytest --cov=apps --cov-report=term-missing

# 运行特定应用测试
pytest apps/orders/tests/ -v

# 并行执行
pytest -n auto

# 只运行上次失败的测试
pytest --lf
```

## ECC 工作流

```bash
# 制定计划
/plan "Add order refund system with Stripe integration"

# 用 TDD 开发
/tdd                    # pytest 基础 TDD 工作流

# 审查
/python-review          # Python 专用代码审查
/security-scan          # Django 安全审计
/code-review            # 通用质量检查

# 验证
/verify                 # 构建、lint、测试、安全扫描
```

## Git 工作流

- `feat:` 新功能, `fix:` 错误修复, `refactor:` 代码变更
- 从 `main` 创建 feature 分支，必须提交 PR
- CI: ruff (lint + 格式化), mypy (类型检查), pytest (测试), safety (依赖项检查)
- 部署: Docker 镜像，由 Kubernetes 或 Railway 管理
