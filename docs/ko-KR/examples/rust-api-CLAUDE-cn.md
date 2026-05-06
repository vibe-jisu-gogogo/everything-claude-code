# Rust API Service — 项目 CLAUDE.md

> 使用 Axum、PostgreSQL、Docker 的 Rust API 服务实战示例。
> 复制到项目根目录并根据您的服务进行自定义。

## 项目概述

**技术栈:** Rust 1.78+, Axum (Web 框架), SQLx (异步数据库), PostgreSQL, Tokio (异步运行时), Docker

**架构:** 分层架构，分为 handler -> service -> repository。HTTP 使用 Axum，SQL 使用编译时类型验证的 SQLx，横切关注点使用 Tower 中间件。

## 核心规则

### Rust 规则

- 库错误使用 `thiserror`，仅在二进制 crate 或测试中使用 `anyhow`
- 生产代码中禁止使用 `.unwrap()` 或 `.expect()` — 使用 `?` 传播错误
- 函数参数优先使用 `&str` 而非 `String`；需要所有权转移时返回 `String`
- 使用 `#![deny(clippy::all, clippy::pedantic)]` 配合 `clippy` — 修复所有警告
- 所有公开类型 derive `Debug`；仅在需要时 derive `Clone`、`PartialEq`
- 除非用 `// SAFETY:` 注释证明其合理性，否则禁止使用 `unsafe` 块

### 数据库

- 所有查询使用 SQLx `query!` 或 `query_as!` 宏 — 针对 Schema 进行编译时验证
- 迁移使用 `sqlx migrate` 放在 `migrations/` 目录 — 不要直接修改数据库
- 共享状态使用 `sqlx::Pool<Postgres>` — 不要为每个请求创建连接
- 所有查询使用 parameterized placeholder (`$1`, `$2`) — 禁止使用字符串格式化

```rust
// 反面示例：字符串插值 (SQL 注入风险)
let q = format!("SELECT * FROM users WHERE id = '{}'", id);

// 正面示例：parameterized 查询，编译时验证
let user = sqlx::query_as!(User, "SELECT * FROM users WHERE id = $1", id)
    .fetch_optional(&pool)
    .await?;
```

### 错误处理

- 按模块使用 `thiserror` 定义领域错误枚举
- 通过 `IntoResponse` 将错误映射为 HTTP 响应 — 不要暴露内部细节
- 使用 `tracing` 进行结构化日志记录 — 禁止使用 `println!` 或 `eprintln!`

```rust
use thiserror::Error;

#[derive(Debug, Error)]
pub enum AppError {
    #[error("Resource not found")]
    NotFound,
    #[error("Validation failed: {0}")]
    Validation(String),
    #[error("Unauthorized")]
    Unauthorized,
    #[error(transparent)]
    Database(#[from] sqlx::Error),
    #[error(transparent)]
    Io(#[from] std::io::Error),
}

impl IntoResponse for AppError {
    fn into_response(self) -> Response {
        let (status, message) = match &self {
            Self::NotFound => (StatusCode::NOT_FOUND, self.to_string()),
            Self::Validation(msg) => (StatusCode::BAD_REQUEST, msg.clone()),
            Self::Unauthorized => (StatusCode::UNAUTHORIZED, self.to_string()),
            Self::Database(err) => {
                tracing::error!(?err, "database error");
                (StatusCode::INTERNAL_SERVER_ERROR, "Internal error".into())
            }
            Self::Io(err) => {
                tracing::error!(?err, "internal error");
                (StatusCode::INTERNAL_SERVER_ERROR, "Internal error".into())
            }
        };
        (status, Json(json!({ "error": message }))).into_response()
    }
}
```

### 测试

- 单元测试在每个源文件内的 `#[cfg(test)]` 模块中
- 集成测试在 `tests/` 目录中使用真实的 PostgreSQL (Testcontainers 或 Docker)
- 数据库测试使用 `#[sqlx::test]`，包含自动迁移和回滚
- 外部服务 mocking 使用 `mockall` 或 `wiremock`

### 代码风格

- 最大行长度：100 字符 (由 rustfmt 强制)
- import 分组：`std`、外部 crate、`crate`/`super` — 用空行分隔
- 模块：每个模块一个文件，`mod.rs` 仅用于 re-export
- 类型：PascalCase，函数/变量：snake_case，常量：UPPER_SNAKE_CASE

## 文件结构

```
src/
  main.rs              # 入口点、服务器配置、优雅关闭
  lib.rs               # 用于集成测试的 re-export
  config.rs            # 使用 envy 或 figment 进行环境配置
  router.rs            # 包含所有路由的 Axum 路由器
  middleware/
    auth.rs            # JWT 提取和验证
    logging.rs         # 请求/响应追踪
  handlers/
    mod.rs             # 路由处理器 (薄 — 委托给服务)
    users.rs
    orders.rs
  services/
    mod.rs             # 业务逻辑
    users.rs
    orders.rs
  repositories/
    mod.rs             # 数据库访问 (SQLx 查询)
    users.rs
    orders.rs
  domain/
    mod.rs             # 领域类型、错误枚举
    user.rs
    order.rs
migrations/
  001_create_users.sql
  002_create_orders.sql
tests/
  common/mod.rs        # 共享测试辅助函数、测试服务器配置
  api_users.rs         # 用户端点集成测试
  api_orders.rs        # 订单端点集成测试
```

## 主要模式

### Handler (薄层)

```rust
async fn create_user(
    State(ctx): State<AppState>,
    Json(payload): Json<CreateUserRequest>,
) -> Result<(StatusCode, Json<UserResponse>), AppError> {
    let user = ctx.user_service.create(payload).await?;
    Ok((StatusCode::CREATED, Json(UserResponse::from(user))))
}
```

### Service (业务逻辑)

```rust
impl UserService {
    pub async fn create(&self, req: CreateUserRequest) -> Result<User, AppError> {
        if self.repo.find_by_email(&req.email).await?.is_some() {
            return Err(AppError::Validation("Email already registered".into()));
        }

        let password_hash = hash_password(&req.password)?;
        let user = self.repo.insert(&req.email, &req.name, &password_hash).await?;

        Ok(user)
    }
}
```

### Repository (数据访问)

```rust
impl UserRepository {
    pub async fn find_by_email(&self, email: &str) -> Result<Option<User>, sqlx::Error> {
        sqlx::query_as!(User, "SELECT * FROM users WHERE email = $1", email)
            .fetch_optional(&self.pool)
            .await
    }

    pub async fn insert(
        &self,
        email: &str,
        name: &str,
        password_hash: &str,
    ) -> Result<User, sqlx::Error> {
        sqlx::query_as!(
            User,
            r#"INSERT INTO users (email, name, password_hash)
               VALUES ($1, $2, $3) RETURNING *"#,
            email, name, password_hash,
        )
        .fetch_one(&self.pool)
        .await
    }
}
```

### 集成测试

```rust
#[tokio::test]
async fn test_create_user() {
    let app = spawn_test_app().await;

    let response = app
        .client
        .post(&format!("{}/api/v1/users", app.address))
        .json(&json!({
            "email": "alice@example.com",
            "name": "Alice",
            "password": "securepassword123"
        }))
        .send()
        .await
        .expect("Failed to send request");

    assert_eq!(response.status(), StatusCode::CREATED);
    let body: serde_json::Value = response.json().await.unwrap();
    assert_eq!(body["email"], "alice@example.com");
}

#[tokio::test]
async fn test_create_user_duplicate_email() {
    let app = spawn_test_app().await;
    // 创建第一个用户
    create_test_user(&app, "alice@example.com").await;
    // 重复尝试
    let response = create_user_request(&app, "alice@example.com").await;
    assert_eq!(response.status(), StatusCode::BAD_REQUEST);
}
```

## 环境变量

```bash
# 服务器
HOST=0.0.0.0
PORT=8080
RUST_LOG=info,tower_http=debug

# 数据库
DATABASE_URL=postgres://user:pass@localhost:5432/myapp

# 认证
JWT_SECRET=your-secret-key-min-32-chars
JWT_EXPIRY_HOURS=24

# 可选
CORS_ALLOWED_ORIGINS=http://localhost:3000
```

## 测试策略

```bash
# 运行所有测试
cargo test

# 带输出运行
cargo test -- --nocapture

# 运行特定测试模块
cargo test api_users

# 检查覆盖率 (需要 cargo-llvm-cov)
cargo llvm-cov --html
open target/llvm-cov/html/index.html

#  lint
cargo clippy -- -D warnings

# 格式检查
cargo fmt -- --check
```

## ECC 工作流

```bash
# 制定计划
/plan "Add order fulfillment with Stripe payment"

# TDD 开发
/tdd                    # 基于 cargo test 的 TDD 工作流

# 审查
/code-review            # Rust 专用代码审查
/security-scan          # 依赖审计 + unsafe 扫描

# 验证
/verify                 # 构建、clippy、测试、安全扫描
```

## Git 工作流

- `feat:` 新功能，`fix:` 修复 bug，`refactor:` 代码变更
- 从 `main` 创建 feature 分支，PR 必需
- CI: `cargo fmt --check`, `cargo clippy`, `cargo test`, `cargo audit`
- 部署：使用 `scratch` 或 `distroless` 基础镜像的 Docker 多阶段构建
