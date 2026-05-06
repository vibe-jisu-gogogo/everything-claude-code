---
name: laravel-tdd
description: Test-driven development for Laravel with PHPUnit and Pest, factories, database testing, fakes, and coverage targets.
origin: ECC
---

# Laravel TDD 工作流

使用 80%+ 覆盖率（unit + feature）进行 Laravel 应用的 test-driven development。

## 何时使用

- Laravel 中的新功能或 endpoints
- Bug 修复或 refactoring
- 测试 Eloquent models、policies、jobs 和 notifications
- 如果项目尚未标准化使用 PHPUnit，优先选择 Pest 进行新测试

## 如何工作

### Red-Green-Refactor 循环

1) 编写一个失败的测试
2) 进行最小化修改使其通过
3) 在保持测试绿色的同时进行 refactor

### 测试层级

- **Unit**: 纯 PHP 类、value objects、services
- **Feature**: HTTP endpoints、auth、validation、policies
- **Integration**: database + queue + 外部边界

根据覆盖范围选择层级：

- 对于纯业务逻辑和 services，使用 **Unit** 测试。
- 对于 HTTP、auth、validation 和响应格式，使用 **Feature** 测试。
- 验证 DB/queues/外部服务组合时，使用 **Integration** 测试。

### Database 策略

- 对于大多数 feature/integration 测试使用 `RefreshDatabase`（每次 test run 运行一次 migrations，然后在支持时将每个测试包装在 transaction 中；in-memory 数据库可能会在每个测试前重新 migrate）
- 如果 schema 已经 migrate 且只需要在每个测试后进行 rollback，使用 `DatabaseTransactions`
- 如果每个测试需要完整的 migrate/fresh 且能承受其成本，使用 `DatabaseMigrations`

对于接触数据库的测试，默认使用 `RefreshDatabase`：对于支持 transaction 的数据库，它在每次 test run 开始时（通过静态标志）运行一次 migrations 并将每个测试包装在 transaction 中；对于 `:memory:` SQLite 或无 transaction 的连接，它在每个测试前 migrate。如果 schema 已经 migrate 且只需要 per-test rollbacks，使用 `DatabaseTransactions`。

### 测试框架选择

- 当可用时，对于新测试默认使用 **Pest**。
- 仅当项目已标准化使用 PHPUnit 或需要 PHPUnit 特定工具时，使用 **PHPUnit**。

## 示例

### PHPUnit 示例

```php
use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

final class ProjectControllerTest extends TestCase
{
    use RefreshDatabase;

    public function test_owner_can_create_project(): void
    {
        $user = User::factory()->create();

        $response = $this->actingAs($user)->postJson('/api/projects', [
            'name' => 'New Project',
        ]);

        $response->assertCreated();
        $this->assertDatabaseHas('projects', ['name' => 'New Project']);
    }
}
```

### Feature Test 示例（HTTP 层级）

```php
use App\Models\Project;
use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

final class ProjectIndexTest extends TestCase
{
    use RefreshDatabase;

    public function test_projects_index_returns_paginated_results(): void
    {
        $user = User::factory()->create();
        Project::factory()->count(3)->for($user)->create();

        $response = $this->actingAs($user)->getJson('/api/projects');

        $response->assertOk();
        $response->assertJsonStructure(['success', 'data', 'error', 'meta']);
    }
}
```

### Pest 示例

```php
use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;

use function Pest\Laravel\actingAs;
use function Pest\Laravel\assertDatabaseHas;

uses(RefreshDatabase::class);

test('owner can create project', function () {
    $user = User::factory()->create();

    $response = actingAs($user)->postJson('/api/projects', [
        'name' => 'New Project',
    ]);

    $response->assertCreated();
    assertDatabaseHas('projects', ['name' => 'New Project']);
});
```

### Feature Test Pest 示例（HTTP 层级）

```php
use App\Models\Project;
use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;

use function Pest\Laravel\actingAs;

uses(RefreshDatabase::class);

test('projects index returns paginated results', function () {
    $user = User::factory()->create();
    Project::factory()->count(3)->for($user)->create();

    $response = actingAs($user)->getJson('/api/projects');

    $response->assertOk();
    $response->assertJsonStructure(['success', 'data', 'error', 'meta']);
});
```

### Factories 和 States

- 使用 factories 生成测试数据
- 为边缘情况定义 states（archived、admin、trial）

```php
$user = User::factory()->state(['role' => 'admin'])->create();
```

### Database 测试

- 使用 `RefreshDatabase` 获得干净状态
- 保持测试隔离和确定性
- 优先使用 `assertDatabaseHas` 而非手动查询

### Persistence Test 示例

```php
use App\Models\Project;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

final class ProjectRepositoryTest extends TestCase
{
    use RefreshDatabase;

    public function test_project_can_be_retrieved_by_slug(): void
    {
        $project = Project::factory()->create(['slug' => 'alpha']);

        $found = Project::query()->where('slug', 'alpha')->firstOrFail();

        $this->assertSame($project->id, $found->id);
    }
}
```

### 副作用的 Fakes

- 对于 Jobs 使用 `Bus::fake()`
- 对于队列中的 jobs 使用 `Queue::fake()`
- 对于 notifications 使用 `Mail::fake()` 和 `Notification::fake()`
- 对于 domain events 使用 `Event::fake()`

```php
use Illuminate\Support\Facades\Queue;

Queue::fake();

dispatch(new SendOrderConfirmation($order->id));

Queue::assertPushed(SendOrderConfirmation::class);
```

```php
use Illuminate\Support\Facades\Notification;

Notification::fake();

$user->notify(new InvoiceReady($invoice));

Notification::assertSentTo($user, InvoiceReady::class);
```

### Auth 测试 (Sanctum)

```php
use Laravel\Sanctum\Sanctum;

Sanctum::actingAs($user);

$response = $this->getJson('/api/projects');
$response->assertOk();
```

### HTTP 和外部服务

- 使用 `Http::fake()` 隔离外部 APIs
- 使用 `Http::assertSent()` 验证传出的 payloads

### 覆盖率目标

- 对于 unit + feature 测试，强制执行 80%+ 覆盖率
- 在 CI 中使用 `pcov` 或 `XDEBUG_MODE=coverage`

### 测试命令

- `php artisan test`
- `vendor/bin/phpunit`
- `vendor/bin/pest`

### 测试配置

- 在 `phpunit.xml` 中设置 `DB_CONNECTION=sqlite` 和 `DB_DATABASE=:memory:` 以进行快速测试
- 为测试维护单独的 env，避免接触 dev/prod 数据

### 授权测试

```php
use Illuminate\Support\Facades\Gate;

$this->assertTrue(Gate::forUser($user)->allows('update', $project));
$this->assertFalse(Gate::forUser($otherUser)->allows('update', $project));
```

### Inertia Feature 测试

使用 Inertia.js 时，使用 Inertia 测试辅助工具验证 component 名称和 props。

```php
use App\Models\User;
use Inertia\Testing\AssertableInertia;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

final class DashboardInertiaTest extends TestCase
{
    use RefreshDatabase;

    public function test_dashboard_inertia_props(): void
    {
        $user = User::factory()->create();

        $response = $this->actingAs($user)->get('/dashboard');

        $response->assertOk();
        $response->assertInertia(fn (AssertableInertia $page) => $page
            ->component('Dashboard')
            ->where('user.id', $user->id)
            ->has('projects')
        );
    }
}
```

优先使用 `assertInertia` 而非原始 JSON assertions，以保持测试与 Inertia 响应一致。
