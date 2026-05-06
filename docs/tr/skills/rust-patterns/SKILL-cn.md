---
name: rust-patterns
description: Idiomatic Rust patterns, ownership, error handling, traits, concurrency, and best practices for building safe, performant applications.
origin: ECC
---

# Rust 开发模式

用于构建安全、高性能和可维护应用的 idiomatic Rust 模式和最佳实践。

## 何时使用

- 编写新 Rust 代码
- 审查 Rust 代码
- 重构现有 Rust 代码
- 设计 crate 结构和模块组织

## 工作原理

此 skill 在六个主要领域强制实施 idiomatic Rust 规则：编译时防止数据竞争的 ownership 和 borrowing，使用 `thiserror`（库）和 `anyhow`（应用程序）的 `Result`/`?` 错误传播，使用 enums 和 exhaustive pattern matching 使非法状态不可表示，用于零成本抽象的 traits 和 generics，使用 `Arc<Mutex<T>>`、channels 和 async/await 的安全并发，以及按领域组织的最小化 `pub` 暴露面。

## 核心原则

### 1. Ownership 和 Borrowing

Rust 的 ownership 系统在编译时防止数据竞争和内存错误。

```rust
// Good: 不需要 ownership 时传递引用
fn process(data: &[u8]) -> usize {
    data.len()
}

// Good: 为了存储或消费获取 ownership
fn store(data: Vec<u8>) -> Record {
    Record { payload: data }
}

// Bad: 为了绕过 borrow checker 进行不必要的 clone
fn process_bad(data: &Vec<u8>) -> usize {
    let cloned = data.clone(); // 浪费 — 只需 borrow
    cloned.len()
}
```

### 为灵活的 Ownership 使用 `Cow`

```rust
use std::borrow::Cow;

fn normalize(input: &str) -> Cow<'_, str> {
    if input.contains(' ') {
        Cow::Owned(input.replace(' ', "_"))
    } else {
        Cow::Borrowed(input) // 无需 mutation 时零成本
    }
}
```

## 错误处理

### 使用 `Result` 和 `?` — 生产环境中永远不要 `unwrap()`

```rust
// Good: 使用 context 传播错误
use anyhow::{Context, Result};

fn load_config(path: &str) -> Result<Config> {
    let content = std::fs::read_to_string(path)
        .with_context(|| format!("failed to read config from {path}"))?;
    let config: Config = toml::from_str(&content)
        .with_context(|| format!("failed to parse config from {path}"))?;
    Ok(config)
}

// Bad: 错误时 panic
fn load_config_bad(path: &str) -> Config {
    let content = std::fs::read_to_string(path).unwrap(); // Panic!
    toml::from_str(&content).unwrap()
}
```

### 库错误使用 `thiserror`，应用程序错误使用 `anyhow`

```rust
// 库代码：结构化、类型化的错误
use thiserror::Error;

#[derive(Debug, Error)]
pub enum StorageError {
    #[error("record not found: {id}")]
    NotFound { id: String },
    #[error("connection failed")]
    Connection(#[from] std::io::Error),
    #[error("invalid data: {0}")]
    InvalidData(String),
}

// 应用程序代码：灵活的错误处理
use anyhow::{bail, Result};

fn run() -> Result<()> {
    let config = load_config("app.toml")?;
    if config.workers == 0 {
        bail!("worker count must be > 0");
    }
    Ok(())
}
```

### 使用 `Option` Combinators 而非嵌套匹配

```rust
// Good: Combinator 链
fn find_user_email(users: &[User], id: u64) -> Option<String> {
    users.iter()
        .find(|u| u.id == id)
        .map(|u| u.email.clone())
}

// Bad: 深度嵌套匹配
fn find_user_email_bad(users: &[User], id: u64) -> Option<String> {
    match users.iter().find(|u| u.id == id) {
        Some(user) => match &user.email {
            email => Some(email.clone()),
        },
        None => None,
    }
}
```

## Enums 和 Pattern Matching

### 将状态建模为 Enums

```rust
// Good: 不可能的状态不可表示
enum ConnectionState {
    Disconnected,
    Connecting { attempt: u32 },
    Connected { session_id: String },
    Failed { reason: String, retries: u32 },
}

fn handle(state: &ConnectionState) {
    match state {
        ConnectionState::Disconnected => connect(),
        ConnectionState::Connecting { attempt } if *attempt > 3 => abort(),
        ConnectionState::Connecting { .. } => wait(),
        ConnectionState::Connected { session_id } => use_session(session_id),
        ConnectionState::Failed { retries, .. } if *retries < 5 => retry(),
        ConnectionState::Failed { reason, .. } => log_failure(reason),
    }
}
```

### Exhaustive Matching — 业务逻辑不要使用 Catch-All

```rust
// Good: 显式处理每个 variant
match command {
    Command::Start => start_service(),
    Command::Stop => stop_service(),
    Command::Restart => restart_service(),
    // 添加新 variant 会强制在此处处理
}

// Bad: Wildcard 隐藏新 variants
match command {
    Command::Start => start_service(),
    _ => {} // 静默忽略 Stop、Restart 和未来 variants
}
```

## Traits 和 Generics

### 接受 Generic 输入，返回具体类型

```rust
// Good: Generic 输入，具体输出
fn read_all(reader: &mut impl Read) -> std::io::Result<Vec<u8>> {
    let mut buf = Vec::new();
    reader.read_to_end(&mut buf)?;
    Ok(buf)
}

// Good: 多个约束的 trait bounds
fn process<T: Display + Send + 'static>(item: T) -> String {
    format!("processed: {item}")
}
```

### 动态分发使用 Trait Objects

```rust
// 需要异构集合或 plugin 系统时使用
trait Handler: Send + Sync {
    fn handle(&self, request: &Request) -> Response;
}

struct Router {
    handlers: Vec<Box<dyn Handler>>,
}

// 需要性能时使用 generics（monomorphization）
fn fast_process<H: Handler>(handler: &H, request: &Request) -> Response {
    handler.handle(request)
}
```

### 类型安全使用 Newtype 模式

```rust
// Good: 不同类型防止参数混淆
struct UserId(u64);
struct OrderId(u64);

fn get_order(user: UserId, order: OrderId) -> Result<Order> {
    // 你不能意外交换 user 和 order ID
    todo!()
}

// Bad: 容易混淆参数
fn get_order_bad(user_id: u64, order_id: u64) -> Result<Order> {
    todo!()
}
```

## Structs 和数据建模

### 复杂配置使用 Builder 模式

```rust
struct ServerConfig {
    host: String,
    port: u16,
    max_connections: usize,
}

impl ServerConfig {
    fn builder(host: impl Into<String>, port: u16) -> ServerConfigBuilder {
        ServerConfigBuilder { host: host.into(), port, max_connections: 100 }
    }
}

struct ServerConfigBuilder { host: String, port: u16, max_connections: usize }

impl ServerConfigBuilder {
    fn max_connections(mut self, n: usize) -> Self { self.max_connections = n; self }
    fn build(self) -> ServerConfig {
        ServerConfig { host: self.host, port: self.port, max_connections: self.max_connections }
    }
}

// Usage: ServerConfig::builder("localhost", 8080).max_connections(200).build()
```

## Iterators 和 Closures

### 优先使用 Iterator 链而非手动循环

```rust
// Good: 声明式、lazy、可组合
let active_emails: Vec<String> = users.iter()
    .filter(|u| u.is_active)
    .map(|u| u.email.clone())
    .collect();

// Bad: 命令式累积
let mut active_emails = Vec::new();
for user in &users {
    if user.is_active {
        active_emails.push(user.email.clone());
    }
}
```

### 使用类型注解的 `collect()`

```rust
// 收集到不同类型
let names: Vec<_> = items.iter().map(|i| &i.name).collect();
let lookup: HashMap<_, _> = items.iter().map(|i| (i.id, i)).collect();
let combined: String = parts.iter().copied().collect();

// 收集 Results — 第一个错误时短路
let parsed: Result<Vec<i32>, _> = strings.iter().map(|s| s.parse()).collect();
```

## 并发

### 共享可变状态使用 `Arc<Mutex<T>>`

```rust
use std::sync::{Arc, Mutex};

let counter = Arc::new(Mutex::new(0));
let handles: Vec<_> = (0..10).map(|_| {
    let counter = Arc::clone(&counter);
    std::thread::spawn(move || {
        let mut num = counter.lock().expect("mutex poisoned");
        *num += 1;
    })
}).collect();

for handle in handles {
    handle.join().expect("worker thread panicked");
}
```

### 消息传递使用 Channels

```rust
use std::sync::mpsc;

let (tx, rx) = mpsc::sync_channel(16); // 带 backpressure 的 bounded channel

for i in 0..5 {
    let tx = tx.clone();
    std::thread::spawn(move || {
        tx.send(format!("message {i}")).expect("receiver disconnected");
    });
}
drop(tx); // 关闭 sender 以便 rx iterator 终止

for msg in rx {
    println!("{msg}");
}
```

### 使用 Tokio 进行 Async

```rust
use tokio::time::Duration;

async fn fetch_with_timeout(url: &str) -> Result<String> {
    let response = tokio::time::timeout(
        Duration::from_secs(5),
        reqwest::get(url),
    )
    .await
    .context("request timed out")?
    .context("request failed")?;

    response.text().await.context("failed to read body")
}

// Spawn 并发任务
async fn fetch_all(urls: Vec<String>) -> Vec<Result<String>> {
    let handles: Vec<_> = urls.into_iter()
        .map(|url| tokio::spawn(async move {
            fetch_with_timeout(&url).await
        }))
        .collect();

    let mut results = Vec::with_capacity(handles.len());
    for handle in handles {
        results.push(handle.await.unwrap_or_else(|e| panic!("spawned task panicked: {e}")));
    }
    results
}
```

## Unsafe 代码

### 何时接受 Unsafe

```rust
// 可接受：带有文档化不变量的 FFI boundary（Rust 2024+）
/// # Safety
/// `ptr` 必须是指向初始化的 `Widget` 的有效、对齐指针。
unsafe fn widget_from_raw<'a>(ptr: *const Widget) -> &'a Widget {
    // SAFETY: 调用者保证 ptr 有效且对齐
    unsafe { &*ptr }
}

// 可接受：带有正确性证明的性能关键路径
// SAFETY: 由于循环边界，index 始终 < len
unsafe { slice.get_unchecked(index) }
```

### 何时不接受 Unsafe

```rust
// Bad: 不要为了绕过 borrow checker 使用 unsafe
// Bad: 不要为了方便使用 unsafe
// Bad: 不要没有 safety 注释使用 unsafe
// Bad: 不要在不相关类型之间 transmute
```

## 模块系统和 Crate 结构

### 按领域组织，而非按类型

```text
my_app/
├── src/
│   ├── main.rs
│   ├── lib.rs
│   ├── auth/          # Domain 模块
│   │   ├── mod.rs
│   │   ├── token.rs
│   │   └── middleware.rs
│   ├── orders/        # Domain 模块
│   │   ├── mod.rs
│   │   ├── model.rs
│   │   └── service.rs
│   └── db/            # 基础设施
│       ├── mod.rs
│       └── pool.rs
├── tests/             # 集成测试
├── benches/           # Benchmarks
└── Cargo.toml
```

### 可见性 — 最小化公开暴露

```rust
// Good: 内部共享使用 pub(crate)
pub(crate) fn validate_input(input: &str) -> bool {
    !input.is_empty()
}

// Good: 从 lib.rs 重新导出 public API
pub mod auth;
pub use auth::AuthMiddleware;

// Bad: 将所有内容设为 pub
pub fn internal_helper() {} // 应该是 pub(crate) 或 private
```

## 工具集成

### 基本命令

```bash
# Build 和检查
cargo build
cargo check              # 无 codegen 的快速类型检查
cargo clippy             # Lints 和建议
cargo fmt                # 格式化代码

# 测试
cargo test
cargo test -- --nocapture    # 显示 println 输出
cargo test --lib             # 仅 unit 测试
cargo test --test integration # 仅集成测试

# 依赖
cargo audit              # 安全审计
cargo tree               # 依赖树
cargo update             # 更新依赖

# 性能
cargo bench              # 运行 benchmarks
```

## 快速参考：Rust Idioms

| Idiom | 描述 |
|-------|------|
| Clone 与 borrow | 不需要 ownership 时传递 `&T` 而非 clone |
| 使非法状态不可表示 | 使用 enums 仅建模有效状态 |
| `?` 而非 `unwrap()` | 传播错误，库/生产代码中永远不要 panic |
| 验证与解析 | 在边界将无结构数据转换为类型化 structs |
| 类型安全使用 newtype | 将 primitives 包装到 newtypes 以防止参数交换 |
| 优先使用 iterators 而非循环 | 声明式链更清晰且通常更快 |
| Results 上的 `#[must_use]` | 确保调用者处理返回值 |
| 灵活 ownership 使用 `Cow` | borrow 足够时避免 allocations |
| Exhaustive matching | 业务关键 enums 不使用 wildcard `_` |
| 最小化 `pub` 暴露面 | 内部 API 使用 `pub(crate)` |

## 要避免的 Anti-Patterns

```rust
// Bad: 生产代码中使用 .unwrap()
let value = map.get("key").unwrap();

// Bad: 为了满足 borrow checker 而不知原因地使用 .clone()
let data = expensive_data.clone();
process(&original, &data);

// Bad: `&str` 足够时使用 String
fn greet(name: String) { /* 应该是 &str */ }

// Bad: 库中使用 Box<dyn Error>（改用 thiserror）
fn parse(input: &str) -> Result<Data, Box<dyn std::error::Error>> { todo!() }

// Bad: 忽略 must_use 警告
let _ = validate(input); // 静默丢弃 Result

// Bad: 在 async 上下文中阻塞
async fn bad_async() {
    std::thread::sleep(Duration::from_secs(1)); // 阻塞 executor！
    // 使用：tokio::time::sleep(Duration::from_secs(1)).await;
}
```

**记住**：如果它能编译，可能是正确的 — 但前提是你避免了 `unwrap()`、最小化了 `unsafe` 并让类型系统为你工作。
