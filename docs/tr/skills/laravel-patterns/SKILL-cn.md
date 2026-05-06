---
name: laravel-patterns
description: Laravel architecture patterns, routing/controllers, Eloquent ORM, service layers, queues, events, caching, and API resources for production apps.
origin: ECC
---

# Laravel 开发模式

用于构建可扩展、可维护应用的生产级 Laravel 架构模式。

## 何时使用

- 创建 Laravel web 应用或 APIs
- 构建 Controllers、services 和领域逻辑
- 处理 Eloquent models 和关联关系
- 使用 Resources 和分页设计 APIs
- 添加 Queues、events、caching 和后台作业

## 工作原理

- 围绕清晰的边界构建应用（controllers -> services/actions -> models）
- 使用显式 binding 和 scoped binding 保持路由可预测；始终实施授权进行访问控制
- 优先使用 typed models、casts 和 scopes 保持领域逻辑一致性
- 将 IO 密集型操作放入队列，缓存昂贵的读取操作
- 在 `config/*` 中集中配置，保持环境清晰

## 示例

### 项目结构

使用具有清晰层级边界（HTTP、services/actions、models）的传统 Laravel 布局。

### 推荐布局

```
app/
├── Actions/            # 单用途用例
├── Console/
├── Events/
├── Exceptions/
├── Http/
│   ├── Controllers/
│   ├── Middleware/
│   ├── Requests/       # Form request validation
│   └── Resources/      # API resources
├── Jobs/
├── Models/
├── Policies/
├── Providers/
├── Services/           # 协调领域服务
└── Support/
config/
database/
├── factories/
├── migrations/
└── seeders/
resources/
├── views/
└── lang/
routes/
├── api.php
├── web.php
└── console.php
```

### Controllers -> Services -> Actions

保持 Controllers 精简。将编排逻辑放入 services，单用途逻辑放入 actions。

```php
final class CreateOrderAction
{
    public function __construct(private OrderRepository $orders) {}

    public function handle(CreateOrderData $data): Order
    {
        return $this->orders->create($data);
    }
}

final class OrdersController extends Controller
{
    public function __construct(private CreateOrderAction $createOrder) {}

    public function store(StoreOrderRequest $request): JsonResponse
    {
        $order = $this->createOrder->handle($request->toDto());

        return response()->json([
            'success' => true,
            'data' => OrderResource::make($order),
            'error' => null,
            'meta' => null,
        ], 201);
    }
}
```

### Routing 和 Controllers

优先使用 route-model binding 和 resource controllers 以保持清晰。

```php
use Illuminate\Support\Facades\Route;

Route::middleware('auth:sanctum')->group(function () {
    Route::apiResource('projects', ProjectController::class);
});
```

### Route Model Binding (Scoped)

使用 scoped bindings 防止跨租户访问。

```php
Route::scopeBindings()->group(function () {
    Route::get('/accounts/{account}/projects/{project}', [ProjectController::class, 'show']);
});
```

### 嵌套路由和 Binding 名称

- 使用一致的前缀和路径避免双重嵌套（如 `conversation` 与 `conversations`）
- 使用与 bound model 匹配的单一参数名称（如 `Conversation` 使用 `{conversation}`）
- 嵌套时优先使用 scoped bindings 强制执行父子关系

```php
use App\Http\Controllers\Api\ConversationController;
use App\Http\Controllers\Api\MessageController;
use Illuminate\Support\Facades\Route;

Route::middleware('auth:sanctum')->prefix('conversations')->group(function () {
    Route::post('/', [ConversationController::class, 'store'])->name('conversations.store');

    Route::scopeBindings()->group(function () {
        Route::get('/{conversation}', [ConversationController::class, 'show'])
            ->name('conversations.show');

        Route::post('/{conversation}/messages', [MessageController::class, 'store'])
            ->name('conversation-messages.store');

        Route::get('/{conversation}/messages/{message}', [MessageController::class, 'show'])
            ->name('conversation-messages.show');
    });
});
```

如果需要参数解析为不同的 model class，请定义显式 bindings。对于自定义 binding 逻辑，使用 `Route::bind()` 或在 model 中实现 `resolveRouteBinding()`。

```php
use App\Models\AiConversation;
use Illuminate\Support\Facades\Route;

Route::model('conversation', AiConversation::class);
```

### Service Container Bindings

在 service provider 中将 interfaces 绑定到 implementations，实现清晰的依赖注入。

```php
use App\Repositories\EloquentOrderRepository;
use App\Repositories\OrderRepository;
use Illuminate\Support\ServiceProvider;

final class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->app->bind(OrderRepository::class, EloquentOrderRepository::class);
    }
}
```

### Eloquent Model 模式

### Model 配置

```php
final class Project extends Model
{
    use HasFactory;

    protected $fillable = ['name', 'owner_id', 'status'];

    protected $casts = [
        'status' => ProjectStatus::class,
        'archived_at' => 'datetime',
    ];

    public function owner(): BelongsTo
    {
        return $this->belongsTo(User::class, 'owner_id');
    }

    public function scopeActive(Builder $query): Builder
    {
        return $query->whereNull('archived_at');
    }
}
```

### 自定义 Casts 和 Value Objects

使用 enums 或 value objects 实现强类型。

```php
use Illuminate\Database\Eloquent\Casts\Attribute;

protected $casts = [
    'status' => ProjectStatus::class,
];
```

```php
protected function budgetCents(): Attribute
{
    return Attribute::make(
        get: fn (int $value) => Money::fromCents($value),
        set: fn (Money $money) => $money->toCents(),
    );
}
```

### 使用 Eager Loading 防止 N+1

```php
$orders = Order::query()
    ->with(['customer', 'items.product'])
    ->latest()
    ->paginate(25);
```

### 使用 Query Objects 处理复杂过滤

```php
final class ProjectQuery
{
    public function __construct(private Builder $query) {}

    public function ownedBy(int $userId): self
    {
        $query = clone $this->query;

        return new self($query->where('owner_id', $userId));
    }

    public function active(): self
    {
        $query = clone $this->query;

        return new self($query->whereNull('archived_at'));
    }

    public function builder(): Builder
    {
        return $this->query;
    }
}
```

### Global Scopes 和 Soft Deletes

使用 global scopes 进行默认过滤，使用 `SoftDeletes` 进行可恢复记录。
除非需要分层行为，否则对于相同的过滤，使用 global scope 或 named scope 之一，不要同时使用。

```php
use Illuminate\Database\Eloquent\SoftDeletes;
use Illuminate\Database\Eloquent\Builder;

final class Project extends Model
{
    use SoftDeletes;

    protected static function booted(): void
    {
        static::addGlobalScope('active', function (Builder $builder): void {
            $builder->whereNull('archived_at');
        });
    }
}
```

### 使用 Query Scopes 实现可重用过滤

```php
use Illuminate\Database\Eloquent\Builder;

final class Project extends Model
{
    public function scopeOwnedBy(Builder $query, int $userId): Builder
    {
        return $query->where('owner_id', $userId);
    }
}

// 在 service、repository 等中使用
$projects = Project::ownedBy($user->id)->get();
```

### 使用 Transactions 处理多步骤更新

```php
use Illuminate\Support\Facades\DB;

DB::transaction(function (): void {
    $order->update(['status' => 'paid']);
    $order->items()->update(['paid_at' => now()]);
});
```

### Migrations

### 命名规则

- 文件名使用时间戳：`YYYY_MM_DD_HHMMSS_create_users_table.php`
- Migrations 使用匿名类（不使用命名类）；文件名传达目的
- 表名默认为 `snake_case` 复数形式

### 示例 Migration

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('orders', function (Blueprint $table): void {
            $table->id();
            $table->foreignId('customer_id')->constrained()->cascadeOnDelete();
            $table->string('status', 32)->index();
            $table->unsignedInteger('total_cents');
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('orders');
    }
};
```

### Form Requests 和 Validation

在 form requests 中保留 validation，并将 inputs 转换为 DTOs。

```php
use App\Models\Order;

final class StoreOrderRequest extends FormRequest
{
    public function authorize(): bool
    {
        return $this->user()?->can('create', Order::class) ?? false;
    }

    public function rules(): array
    {
        return [
            'customer_id' => ['required', 'integer', 'exists:customers,id'],
            'items' => ['required', 'array', 'min:1'],
            'items.*.sku' => ['required', 'string'],
            'items.*.quantity' => ['required', 'integer', 'min:1'],
        ];
    }

    public function toDto(): CreateOrderData
    {
        return new CreateOrderData(
            customerId: (int) $this->validated('customer_id'),
            items: $this->validated('items'),
        );
    }
}
```

### API Resources

使用 Resources 和分页保持 API 响应一致性。

```php
$projects = Project::query()->active()->paginate(25);

return response()->json([
    'success' => true,
    'data' => ProjectResource::collection($projects->items()),
    'error' => null,
    'meta' => [
        'page' => $projects->currentPage(),
        'per_page' => $projects->perPage(),
        'total' => $projects->total(),
    ],
]);
```

### Events、Jobs 和 Queues

- 为副作用发布 domain events（emails、analytics）
- 对慢速操作使用排队 jobs（reports、exports、webhooks）
- 优先使用具有重试和 backoff 的幂等 handlers

### Caching

- 缓存读取密集型 endpoints 和昂贵的查询
- 在 model events（created/updated/deleted）中使缓存失效
- 缓存相关数据时使用 tags，以便轻松失效

### 配置和环境

- 将 secrets 保存在 `.env` 中，配置保存在 `config/*.php` 中
- 使用特定于环境的配置覆盖，在 production 中使用 `config:cache`
