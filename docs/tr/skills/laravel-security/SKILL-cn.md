---
name: laravel-security
description: Laravel security best practices for authn/authz, validation, CSRF, mass assignment, file uploads, secrets, rate limiting, and secure deployment.
origin: ECC
---

# Laravel 安全最佳实践

保护 Laravel 应用程序免受常见安全漏洞攻击的综合安全指南。

## 何时使用

- 添加身份验证或授权
- 处理用户输入和文件上传
- 创建新的 API endpoints
- 管理机密信息和环境配置
- 强化生产环境部署

## 工作原理

- Middleware 提供基础保护（CSRF 使用 `VerifyCsrfToken`，安全头使用 `SecurityHeaders`）。
- Guard 和 Policy 强制实施访问控制（`auth:sanctum`、`$this->authorize`、policy middleware）。
- Form Request 在请求到达服务之前验证和规范化输入（`UploadInvoiceRequest`）。
- Rate limiting 与认证控制器结合提供滥用保护（`RateLimiter::for('login')`）。
- 数据安全来自 encrypted casts、mass-assignment 保护和 signed routes（`URL::temporarySignedRoute` + `signed` middleware）。

## 基础安全配置

- 生产环境设置 `APP_DEBUG=false`
- 必须设置 `APP_KEY`，泄露后立即轮换
- 设置 `SESSION_SECURE_COOKIE=true` 和 `SESSION_SAME_SITE=lax`（敏感应用使用 `strict`）
- 配置可信代理以正确检测 HTTPS

## Session 和 Cookie 强化

- 设置 `SESSION_HTTP_ONLY=true` 防止 JavaScript 访问
- 高风险流程使用 `SESSION_SAME_SITE=strict`
- 登录和权限变更时重新生成 session

## 身份验证和 Token

- API 认证使用 Laravel Sanctum 或 Passport
- 敏感数据优先使用带刷新流程的短寿命 token
- 登出和账户泄露时撤销 token

路由保护示例：

```php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;

Route::middleware('auth:sanctum')->get('/me', function (Request $request) {
    return $request->user();
});
```

## 密码安全

- 使用 `Hash::make()` 哈希密码，绝不以明文存储
- 密码重置流程使用 Laravel 的 password broker

```php
use Illuminate\Support\Facades\Hash;
use Illuminate\Validation\Rules\Password;

$validated = $request->validate([
    'password' => ['required', 'string', Password::min(12)->letters()->mixedCase()->numbers()->symbols()],
]);

$user->update(['password' => Hash::make($validated['password'])]);
```

## 授权：Policy 和 Gate

- 模型级授权使用 Policy
- 在控制器和服务中强制授权检查

```php
$this->authorize('update', $project);
```

路由级强制使用 policy middleware：

```php
use Illuminate\Support\Facades\Route;

Route::put('/projects/{project}', [ProjectController::class, 'update'])
    ->middleware(['auth:sanctum', 'can:update,project']);
```

## 验证和数据清洗

- 始终使用 Form Request 验证输入
- 使用严格的验证规则和类型检查
- 绝不信任请求 payload 中的派生字段

## Mass Assignment 保护

- 使用 `$fillable` 或 `$guarded`，避免使用 `Model::unguard()`
- 优先使用 DTO 或显式属性映射

## SQL Injection 防护

- 使用 Eloquent 或 query builder 参数绑定
- 除非绝对必要，避免使用 raw SQL

```php
DB::select('select * from users where email = ?', [$email]);
```

## XSS 防护

- Blade 默认转义输出 (`{{ }}`)
- `{!! !!}` 仅用于可信、已清洗的 HTML
- 使用专用库清洗富文本内容

## CSRF 保护

- 保持 `VerifyCsrfToken` middleware 启用
- 表单添加 `@csrf`，SPA 请求发送 XSRF token

使用 Sanctum 进行 SPA 认证时，确保配置了有状态请求：

```php
// config/sanctum.php
'stateful' => explode(',', env('SANCTUM_STATEFUL_DOMAINS', 'localhost')),
```

## 文件上传安全

- 验证文件大小、MIME 类型和扩展名
- 尽可能将上传文件存储在 public path 之外
- 必要时扫描文件中的恶意软件

```php
final class UploadInvoiceRequest extends FormRequest
{
    public function authorize(): bool
    {
        return (bool) $this->user()?->can('upload-invoice');
    }

    public function rules(): array
    {
        return [
            'invoice' => ['required', 'file', 'mimes:pdf', 'max:5120'],
        ];
    }
}
```

```php
$path = $request->file('invoice')->store(
    'invoices',
    config('filesystems.private_disk', 'local') // 设置为非公开磁盘
);
```

## Rate Limiting

- 在认证和写入 endpoints 应用 `throttle` middleware
- 登录、密码重置和 OTP 使用更严格的限制

```php
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\RateLimiter;

RateLimiter::for('login', function (Request $request) {
    return [
        Limit::perMinute(5)->by($request->ip()),
        Limit::perMinute(5)->by(strtolower((string) $request->input('email'))),
    ];
});
```

## 机密信息和凭证

- 绝不将机密信息提交到源代码控制
- 使用环境变量和机密管理器
- 泄露后轮换密钥并使 session 失效

## 加密属性

对于敏感列使用 encrypted casts。

```php
protected $casts = [
    'api_token' => 'encrypted',
];
```

## 安全头

- 在适当位置添加 CSP、HSTS 和 frame 保护
- 使用可信代理配置强制 HTTPS 重定向

设置安全头的示例 middleware：

```php
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

final class SecurityHeaders
{
    public function handle(Request $request, \Closure $next): Response
    {
        $response = $next($request);

        $response->headers->add([
            'Content-Security-Policy' => "default-src 'self'",
            'Strict-Transport-Security' => 'max-age=31536000', // 所有子域名使用 HTTPS 时添加 includeSubDomains/preload
            'X-Frame-Options' => 'DENY',
            'X-Content-Type-Options' => 'nosniff',
            'Referrer-Policy' => 'no-referrer',
        ]);

        return $response;
    }
}
```

## CORS 和 API 访问

- 在 `config/cors.php` 中限制 origins
- 认证路由避免使用 wildcard origin

```php
// config/cors.php
return [
    'paths' => ['api/*', 'sanctum/csrf-cookie'],
    'allowed_methods' => ['GET', 'POST', 'PUT', 'PATCH', 'DELETE'],
    'allowed_origins' => ['https://app.example.com'],
    'allowed_headers' => [
        'Content-Type',
        'Authorization',
        'X-Requested-With',
        'X-XSRF-TOKEN',
        'X-CSRF-TOKEN',
    ],
    'supports_credentials' => true,
];
```

## 日志和 PII

- 绝不记录密码、token 或完整银行卡数据
- 在结构化日志中编辑敏感字段

```php
use Illuminate\Support\Facades\Log;

Log::info('User updated profile', [
    'user_id' => $user->id,
    'email' => '[REDACTED]',
    'token' => '[REDACTED]',
]);
```

## 依赖安全

- 定期运行 `composer audit`
- 谨慎固定依赖版本并在发现 CVE 时快速更新

## Signed URL

使用 signed routes 创建临时、防篡改的链接。

```php
use Illuminate\Support\Facades\URL;

$url = URL::temporarySignedRoute(
    'downloads.invoice',
    now()->addMinutes(15),
    ['invoice' => $invoice->id]
);
```

```php
use Illuminate\Support\Facades\Route;

Route::get('/invoices/{invoice}/download', [InvoiceController::class, 'download'])
    ->name('downloads.invoice')
    ->middleware('signed');
```
