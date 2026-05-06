---
name: laravel-verification
description: Verification loop for Laravel projects: env checks, linting, static analysis, tests with coverage, security scans, and deployment readiness.
origin: ECC
---

# Laravel 验证循环

在 PR 之前、重大变更之后以及部署前运行。

## 何时使用

- 为 Laravel 项目提交 pull request 之前
- 大型重构或依赖升级之后
- Staging 或 production 部署前验证
- 运行完整的 lint -> test -> security -> deployment 准备流程

## 工作原理

- 按顺序执行各个阶段，从环境检查到部署准备，每个阶段建立在前一阶段之上。
- 环境和 Composer 检查涵盖所有内容；如果失败立即停止。
- 在运行完整测试和覆盖率之前，linting/static analysis 应该通过。
- 安全和 migration 检查在测试之后进行，这样可以在数据或发布步骤之前验证行为。
- Build/deployment 准备和 queue/scheduler 检查是最后关卡；任何失败都会阻止发布。

## 阶段 1：环境检查

```bash
php -v
composer --version
php artisan --version
```

- 验证 `.env` 存在且包含必要的密钥
- 对于 production 环境，确认 `APP_DEBUG=false`
- 确认 `APP_ENV` 与目标 deployment 匹配（`production`、`staging`）

如果在本地使用 Laravel Sail：

```bash
./vendor/bin/sail php -v
./vendor/bin/sail artisan --version
```

## 阶段 1.5：Composer 和 Autoload

```bash
composer validate
composer dump-autoload -o
```

## 阶段 2：Linting 和静态分析

```bash
vendor/bin/pint --test
vendor/bin/phpstan analyse
```

如果项目使用 Psalm 而非 PHPStan：

```bash
vendor/bin/psalm
```

## 阶段 3：测试和覆盖率

```bash
php artisan test
```

覆盖率（CI）：

```bash
XDEBUG_MODE=coverage php artisan test --coverage
```

CI 示例（format -> static analysis -> tests）：

```bash
vendor/bin/pint --test
vendor/bin/phpstan analyse
XDEBUG_MODE=coverage php artisan test --coverage
```

## 阶段 4：安全和依赖检查

```bash
composer audit
```

## 阶段 5：数据库和 Migration

```bash
php artisan migrate --pretend
php artisan migrate:status
```

- 仔细检查破坏性 migration
- 确保 migration 文件名遵循 `Y_m_d_His_*` 格式（例如 `2025_03_14_154210_create_orders_table.php`）并清晰描述变更
- 确保可以回滚
- 验证 `down()` 方法，避免在没有明确备份的情况下发生不可逆的数据丢失

## 阶段 6：构建和部署准备

```bash
php artisan optimize:clear
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

- 确保 cache warmup 在 production 配置中成功
- 验证 queue worker 和 scheduler 已配置
- 确认目标环境中 `storage/` 和 `bootstrap/cache/` 可写

## 阶段 7：队列和调度器检查

```bash
php artisan schedule:list
php artisan queue:failed
```

如果使用 Horizon：

```bash
php artisan horizon:status
```

如果 `queue:monitor` 可用，用它检查 job 在处理前的堆积情况：

```bash
php artisan queue:monitor default --max=100
```

主动验证（仅 staging）：向专用队列分派 no-op job 并运行单个 worker 处理（确保非 `sync` 队列连接已配置）。

```bash
php artisan tinker --execute="dispatch((new App\\Jobs\\QueueHealthcheck())->onQueue('healthcheck'))"
php artisan queue:work --once --queue=healthcheck
```

验证 job 产生了预期的副作用（日志条目、healthcheck 表行或指标）。

仅在处理测试 job 安全的非生产环境中运行此操作。

## 示例

最简流程：

```bash
php -v
composer --version
php artisan --version
composer validate
vendor/bin/pint --test
vendor/bin/phpstan analyse
php artisan test
composer audit
php artisan migrate --pretend
php artisan config:cache
php artisan queue:failed
```

CI 风格 pipeline：

```bash
composer validate
composer dump-autoload -o
vendor/bin/pint --test
vendor/bin/phpstan analyse
XDEBUG_MODE=coverage php artisan test --coverage
composer audit
php artisan migrate --pretend
php artisan optimize:clear
php artisan config:cache
php artisan route:cache
php artisan view:cache
php artisan schedule:list
```
