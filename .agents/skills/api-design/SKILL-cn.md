---
name: api-design
description: REST API 设计模式，包含生产环境 API 的资源命名、状态码、pagination、filtering、错误响应、versioning 和 rate limiting。
---

# API 设计模式

设计一致、对开发者友好的 REST API 的约定和最佳实践。

## 何时启用

- 设计新的 API 端点
- 审核现有 API 契约
- 添加 pagination、filtering 或 sorting 功能
- 为 API 实现错误处理
- 规划 API versioning 策略
- 构建面向公众或合作伙伴的 API

## 资源设计

### URL 结构

```
# 资源使用名词、复数形式、小写、kebab-case
GET    /api/v1/users
GET    /api/v1/users/:id
POST   /api/v1/users
PUT    /api/v1/users/:id
PATCH  /api/v1/users/:id
DELETE /api/v1/users/:id

# 关系型数据使用子资源
GET    /api/v1/users/:id/orders
POST   /api/v1/users/:id/orders

# 不对应 CRUD 的操作（尽量少用动词）
POST   /api/v1/orders/:id/cancel
POST   /api/v1/auth/login
POST   /api/v1/auth/refresh
```

### 命名规则

```
# 正面示例
/api/v1/team-members          # 多词资源使用 kebab-case
/api/v1/orders?status=active  # 过滤使用查询参数
/api/v1/users/123/orders      # 从属关系使用嵌套资源

# 反面示例
/api/v1/getUsers              # URL 中使用动词
/api/v1/user                  # 单数形式（使用复数）
/api/v1/team_members          # URL 中使用 snake_case
/api/v1/users/123/getOrders   # 嵌套资源中使用动词
```

## HTTP 方法与状态码

### 方法语义

| 方法 | 幂等 | 安全 | 用途 |
|--------|-----------|------|---------|
| GET | 是 | 是 | 检索资源 |
| POST | 否 | 否 | 创建资源、触发操作 |
| PUT | 是 | 否 | 全量替换资源 |
| PATCH | 否* | 否 | 部分更新资源 |
| DELETE | 是 | 否 | 删除资源 |

*通过合理实现，PATCH 也可以做到幂等

### 状态码参考

```
# 成功类
200 OK                    — 用于 GET、PUT、PATCH（返回响应体时）
201 Created               — 用于 POST（需包含 Location 响应头）
204 No Content            — 用于 DELETE、PUT（无响应体时）

# 客户端错误
400 Bad Request           — 验证失败、JSON 格式错误
401 Unauthorized          — 缺少认证信息或认证无效
403 Forbidden             — 已认证但无操作权限
404 Not Found             — 资源不存在
409 Conflict              — 条目重复、状态冲突
422 Unprocessable Entity  — 语义无效（JSON 格式正确但数据错误）
429 Too Many Requests     — 超出 rate limit

# 服务端错误
500 Internal Server Error — 意外故障（切勿暴露错误详情）
502 Bad Gateway           — 上游服务故障
503 Service Unavailable   — 临时过载，需包含 Retry-After 响应头
```

### 常见错误

```
# 反面示例：所有请求都返回 200
{ "status": 200, "success": false, "error": "Not found" }

# 正面示例：语义化使用 HTTP 状态码
HTTP/1.1 404 Not Found
{ "error": { "code": "not_found", "message": "User not found" } }

# 反面示例：验证错误返回 500
# 正面示例：返回 400 或 422 并携带字段级错误详情

# 反面示例：资源创建成功返回 200
# 正面示例：返回 201 并携带 Location 响应头
HTTP/1.1 201 Created
Location: /api/v1/users/abc-123
```

## 响应格式

### 成功响应

```json
{
  "data": {
    "id": "abc-123",
    "email": "alice@example.com",
    "name": "Alice",
    "created_at": "2025-01-15T10:30:00Z"
  }
}
```

### 集合响应（含 Pagination）

```json
{
  "data": [
    { "id": "abc-123", "name": "Alice" },
    { "id": "def-456", "name": "Bob" }
  ],
  "meta": {
    "total": 142,
    "page": 1,
    "per_page": 20,
    "total_pages": 8
  },
  "links": {
    "self": "/api/v1/users?page=1&per_page=20",
    "next": "/api/v1/users?page=2&per_page=20",
    "last": "/api/v1/users?page=8&per_page=20"
  }
}
```

### 错误响应

```json
{
  "error": {
    "code": "validation_error",
    "message": "Request validation failed",
    "details": [
      {
        "field": "email",
        "message": "Must be a valid email address",
        "code": "invalid_format"
      },
      {
        "field": "age",
        "message": "Must be between 0 and 150",
        "code": "out_of_range"
      }
    ]
  }
}
```

### 响应封装变体

```typescript
// 选项 A：带 data 包装的封装（推荐用于公开 API）
interface ApiResponse<T> {
  data: T;
  meta?: PaginationMeta;
  links?: PaginationLinks;
}

interface ApiError {
  error: {
    code: string;
    message: string;
    details?: FieldError[];
  };
}

// 选项 B：扁平响应（更简单，常用于内部 API）
// 成功：直接返回资源本身
// 错误：返回错误对象
// 通过 HTTP 状态码区分响应类型
```

## Pagination

### 基于偏移量的实现（简单）

```
GET /api/v1/users?page=2&per_page=20

# 实现示例
SELECT * FROM users
ORDER BY created_at DESC
LIMIT 20 OFFSET 20;
```

**优点：** 易于实现，支持"跳转到第 N 页"
**缺点：** 偏移量很大时性能差（比如 OFFSET 100000），存在并发写入时结果不稳定

### 基于 Cursor 的实现（可扩展）

```
GET /api/v1/users?cursor=eyJpZCI6MTIzfQ&limit=20

# 实现示例
SELECT * FROM users
WHERE id > :cursor_id
ORDER BY id ASC
LIMIT 21;  -- 多取一条记录判断是否有下一页
```

```json
{
  "data": [...],
  "meta": {
    "has_next": true,
    "next_cursor": "eyJpZCI6MTQzfQ"
  }
}
```

**优点：** 性能稳定不受查询位置影响，并发写入时结果一致
**缺点：** 无法跳转到任意页面，cursor 不透明

### 适用场景对比

| 场景 | Pagination 类型 |
|----------|----------------|
| 管理后台、小数据集（<1万条） | 偏移量 |
| 无限滚动、信息流、大数据集 | Cursor |
| 公开 API | Cursor（默认）+ 偏移量（可选） |
| 搜索结果 | 偏移量（用户期望页码展示） |

## Filtering、Sorting 和搜索

### Filtering

```
# 简单等值查询
GET /api/v1/orders?status=active&customer_id=abc-123

# 比较运算符（使用方括号表示法）
GET /api/v1/products?price[gte]=10&price[lte]=100
GET /api/v1/orders?created_at[after]=2025-01-01

# 多值查询（逗号分隔）
GET /api/v1/products?category=electronics,clothing

# 嵌套字段查询（点表示法）
GET /api/v1/orders?customer.country=US
```

### Sorting

```
# 单字段排序（前缀 - 表示降序）
GET /api/v1/products?sort=-created_at

# 多字段排序（逗号分隔）
GET /api/v1/products?sort=-featured,price,-created_at
```

### 全文搜索

```
# 搜索查询参数
GET /api/v1/products?q=wireless+headphones

# 指定字段搜索
GET /api/v1/users?email=alice
```

### 稀疏字段集

```
# 仅返回指定字段（减少响应体积）
GET /api/v1/users?fields=id,name,email
GET /api/v1/orders?fields=id,total,status&include=customer.name
```

## Authentication 和 Authorization

### 基于 Token 的 Auth

```
# Authorization 头中携带 Bearer token
GET /api/v1/users
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...

# API key（用于服务间调用）
GET /api/v1/data
X-API-Key: sk_live_abc123
```

### Authorization 模式

```typescript
// 资源级别：检查所有权
app.get("/api/v1/orders/:id", async (req, res) => {
  const order = await Order.findById(req.params.id);
  if (!order) return res.status(404).json({ error: { code: "not_found" } });
  if (order.userId !== req.user.id) return res.status(403).json({ error: { code: "forbidden" } });
  return res.json({ data: order });
});

// 基于角色：检查权限
app.delete("/api/v1/users/:id", requireRole("admin"), async (req, res) => {
  await User.delete(req.params.id);
  return res.status(204).send();
});
```

## Rate Limiting

### 响应头

```
HTTP/1.1 200 OK
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1640000000

# 超出限制时
HTTP/1.1 429 Too Many Requests
Retry-After: 60
{
  "error": {
    "code": "rate_limit_exceeded",
    "message": "Rate limit exceeded. Try again in 60 seconds."
  }
}
```

### Rate Limit 层级

| 层级 | 限制 | 窗口 | 适用场景 |
|------|-------|--------|----------|
| 匿名用户 | 30次/分钟 | 按 IP | 公开端点 |
| 已认证用户 | 100次/分钟 | 按用户 | 标准 API 访问 |
| 高级用户 | 1000次/分钟 | 按 API key | 付费 API 套餐 |
| 内部服务 | 10000次/分钟 | 按服务 | 服务间调用 |

## Versioning

### URL 路径 Versioning（推荐）

```
/api/v1/users
/api/v2/users
```

**优点：** 显式清晰、易于路由、支持缓存
**缺点：** 不同版本 URL 不同

### 请求头 Versioning

```
GET /api/users
Accept: application/vnd.myapp.v2+json
```

**优点：** URL 简洁
**缺点：** 测试难度高，容易遗漏

### Versioning 策略

```
1. 从 /api/v1/ 开始——不需要 version 时不要强行添加
2. 最多同时维护2个活跃版本（当前版本 + 上一个版本）
3. 弃用时间线：
   - 发布弃用公告（公开 API 需提前6个月通知）
   - 添加 Sunset 响应头：Sunset: Sat, 01 Jan 2026 00:00:00 GMT
   - 过了弃用日期后返回 410 Gone
4. 非破坏性变更不需要新版本：
   - 为响应添加新字段
   - 添加新的可选查询参数
   - 添加新端点
5. 破坏性变更需要新版本：
   - 删除或重命名字段
   - 更改字段类型
   - 更改 URL 结构
   - 更改认证方式
```

## 实现模式

### TypeScript (Next.js API Route)

```typescript
import { z } from "zod";
import { NextRequest, NextResponse } from "next/server";

const createUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1).max(100),
});

export async function POST(req: NextRequest) {
  const body = await req.json();
  const parsed = createUserSchema.safeParse(body);

  if (!parsed.success) {
    return NextResponse.json({
      error: {
        code: "validation_error",
        message: "Request validation failed",
        details: parsed.error.issues.map(i => ({
          field: i.path.join("."),
          message: i.message,
          code: i.code,
        })),
      },
    }, { status: 422 });
  }

  const user = await createUser(parsed.data);

  return NextResponse.json(
    { data: user },
    {
      status: 201,
      headers: { Location: `/api/v1/users/${user.id}` },
    },
  );
}
```

### Python (Django REST Framework)

```python
from rest_framework import serializers, viewsets, status
from rest_framework.response import Response

class CreateUserSerializer(serializers.Serializer):
    email = serializers.EmailField()
    name = serializers.CharField(max_length=100)

class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ["id", "email", "name", "created_at"]

class UserViewSet(viewsets.ModelViewSet):
    serializer_class = UserSerializer
    permission_classes = [IsAuthenticated]

    def get_serializer_class(self):
        if self.action == "create":
            return CreateUserSerializer
        return UserSerializer

    def create(self, request):
        serializer = CreateUserSerializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        user = UserService.create(**serializer.validated_data)
        return Response(
            {"data": UserSerializer(user).data},
            status=status.HTTP_201_CREATED,
            headers={"Location": f"/api/v1/users/{user.id}"},
        )
```

### Go (net/http)

```go
func (h *UserHandler) CreateUser(w http.ResponseWriter, r *http.Request) {
    var req CreateUserRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        writeError(w, http.StatusBadRequest, "invalid_json", "Invalid request body")
        return
    }

    if err := req.Validate(); err != nil {
        writeError(w, http.StatusUnprocessableEntity, "validation_error", err.Error())
        return
    }

    user, err := h.service.Create(r.Context(), req)
    if err != nil {
        switch {
        case errors.Is(err, domain.ErrEmailTaken):
            writeError(w, http.StatusConflict, "email_taken", "Email already registered")
        default:
            writeError(w, http.StatusInternalServerError, "internal_error", "Internal error")
        }
        return
    }

    w.Header().Set("Location", fmt.Sprintf("/api/v1/users/%s", user.ID))
    writeJSON(w, http.StatusCreated, map[string]any{"data": user})
}
```

## API 设计检查清单

发布新端点前确认：

- [ ] 资源 URL 符合命名约定（复数、kebab-case、无动词）
- [ ] 使用了正确的 HTTP 方法（GET 用于读取、POST 用于创建等）
- [ ] 返回了合适的状态码（不是所有请求都返回 200）
- [ ] 使用 schema 验证输入（Zod、Pydantic、Bean Validation 等）
- [ ] 错误响应遵循标准格式，包含错误码和消息
- [ ] 列表端点实现了 pagination（cursor 或偏移量）
- [ ] 配置了必要的 authentication（或显式标记为公开端点）
- [ ] 实现了 authorization 校验（用户仅能访问自己的资源）
- [ ] 配置了 rate limiting
- [ ] 响应不会泄露内部细节（栈追踪、SQL 错误等）
- [ ] 与现有端点命名风格一致（统一使用 camelCase 或 snake_case）
- [ ] 已完成文档更新（OpenAPI/Swagger 规范）
