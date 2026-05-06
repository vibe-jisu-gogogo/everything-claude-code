---
name: rust-testing
description: Rust testing patterns including unit tests, integration tests, async testing, property-based testing, mocking, and coverage. Follows TDD methodology.
origin: ECC
---

# Rust 测试模式

遵循 TDD 方法编写可靠、可维护测试的全面 Rust 测试模式。

## 何时使用

- 编写新的 Rust 函数、方法或 trait
- 为现有代码添加测试覆盖
- 为性能关键代码创建 benchmark
- 为输入验证应用 property-based 测试
- 在 Rust 项目中遵循 TDD 工作流

## 如何工作

1. **定义目标代码** — 找到要测试的函数、trait 或模块
2. **编写测试** — 在 `#[cfg(test)]` 模块中使用 `#[test]`，使用 rstest 进行参数化测试或使用 proptest 进行 property-based 测试
3. **Mock 依赖** — 使用 mockall 隔离被测单元
4. **运行测试（RED）** — 验证测试以预期错误失败
5. **实现（GREEN）** — 编写最少代码使测试通过
6. **重构** — 在保持测试通过的同时改进代码
7. **检查覆盖** — 使用 cargo-llvm-cov，目标 80%+

## Rust 的 TDD 工作流

### RED-GREEN-REFACTOR 循环

```
RED     → 先写一个失败的测试
GREEN   → 编写最少代码使测试通过
REFACTOR → 在保持测试通过的同时改进代码
REPEAT  → 继续下一个需求
```

### Rust 中的分步 TDD

```rust
// RED: 先写测试，使用 todo!() 作为占位符
pub fn add(a: i32, b: i32) -> i32 { todo!() }

#[cfg(test)]
mod tests {
    use super::*;
    #[test]
    fn test_add() { assert_eq!(add(2, 3), 5); }
}
// cargo test → 在 'not yet implemented' 处 panic
```

```rust
// GREEN: 用最少实现替换 todo!()
pub fn add(a: i32, b: i32) -> i32 { a + b }
// cargo test → 通过，然后在保持测试通过的同时进行 REFACTOR
```

## Unit 测试

### 模块级测试组织

```rust
// src/user.rs
pub struct User {
    pub name: String,
    pub email: String,
}

impl User {
    pub fn new(name: impl Into<String>, email: impl Into<String>) -> Result<Self, String> {
        let email = email.into();
        if !email.contains('@') {
            return Err(format!("invalid email: {email}"));
        }
        Ok(Self { name: name.into(), email })
    }

    pub fn display_name(&self) -> &str {
        &self.name
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn creates_user_with_valid_email() {
        let user = User::new("Alice", "alice@example.com").unwrap();
        assert_eq!(user.display_name(), "Alice");
        assert_eq!(user.email, "alice@example.com");
    }

    #[test]
    fn rejects_invalid_email() {
        let result = User::new("Bob", "not-an-email");
        assert!(result.is_err());
        assert!(result.unwrap_err().contains("invalid email"));
    }
}
```

### Assertion 宏

```rust
assert_eq!(2 + 2, 4);                                    // 相等
assert_ne!(2 + 2, 5);                                    // 不相等
assert!(vec![1, 2, 3].contains(&2));                     // 布尔值
assert_eq!(value, 42, "expected 42 but got {value}");    // 自定义消息
assert!((0.1_f64 + 0.2 - 0.3).abs() < f64::EPSILON);   // Float 比较
```

## 错误和 Panic 测试

### 测试 `Result` 返回

```rust
#[test]
fn parse_returns_error_for_invalid_input() {
    let result = parse_config("}{invalid");
    assert!(result.is_err());

    // 验证特定错误变体
    let err = result.unwrap_err();
    assert!(matches!(err, ConfigError::ParseError(_)));
}

#[test]
fn parse_succeeds_for_valid_input() -> Result<(), Box<dyn std::error::Error>> {
    let config = parse_config(r#"{"port": 8080}"#)?;
    assert_eq!(config.port, 8080);
    Ok(()) // 任何 ? 返回 Err 都会导致测试失败
}
```

### 测试 Panic

```rust
#[test]
#[should_panic]
fn panics_on_empty_input() {
    process(&[]);
}

#[test]
#[should_panic(expected = "index out of bounds")]
fn panics_with_specific_message() {
    let v: Vec<i32> = vec![];
    let _ = v[0];
}
```

## Integration 测试

### 文件结构

```text
my_crate/
├── src/
│   └── lib.rs
├── tests/              # Integration 测试
│   ├── api_test.rs     # 每个文件是一个单独的 test binary
│   ├── db_test.rs
│   └── common/         # 共享测试辅助工具
│       └── mod.rs
```

### 编写 Integration 测试

```rust
// tests/api_test.rs
use my_crate::{App, Config};

#[test]
fn full_request_lifecycle() {
    let config = Config::test_default();
    let app = App::new(config);

    let response = app.handle_request("/health");
    assert_eq!(response.status, 200);
    assert_eq!(response.body, "OK");
}
```

## Async 测试

### 使用 Tokio

```rust
#[tokio::test]
async fn fetches_data_successfully() {
    let client = TestClient::new().await;
    let result = client.get("/data").await;
    assert!(result.is_ok());
    assert_eq!(result.unwrap().items.len(), 3);
}

#[tokio::test]
async fn handles_timeout() {
    use std::time::Duration;
    let result = tokio::time::timeout(
        Duration::from_millis(100),
        slow_operation(),
    ).await;

    assert!(result.is_err(), "should have timed out");
}
```

## 测试组织模式

### 使用 `rstest` 进行参数化测试

```rust
use rstest::{rstest, fixture};

#[rstest]
#[case("hello", 5)]
#[case("", 0)]
#[case("rust", 4)]
fn test_string_length(#[case] input: &str, #[case] expected: usize) {
    assert_eq!(input.len(), expected);
}

// Fixture
#[fixture]
fn test_db() -> TestDb {
    TestDb::new_in_memory()
}

#[rstest]
fn test_insert(test_db: TestDb) {
    test_db.insert("key", "value");
    assert_eq!(test_db.get("key"), Some("value".into()));
}
```

### 测试辅助工具

```rust
#[cfg(test)]
mod tests {
    use super::*;

    /// 使用合理的默认值创建测试用户
    fn make_user(name: &str) -> User {
        User::new(name, &format!("{name}@test.com")).unwrap()
    }

    #[test]
    fn user_display() {
        let user = make_user("alice");
        assert_eq!(user.display_name(), "alice");
    }
}
```

## 使用 `proptest` 进行 Property-Based Testing

### 基本 Property 测试

```rust
use proptest::prelude::*;

proptest! {
    #[test]
    fn encode_decode_roundtrip(input in ".*") {
        let encoded = encode(&input);
        let decoded = decode(&encoded).unwrap();
        assert_eq!(input, decoded);
    }

    #[test]
    fn sort_preserves_length(mut vec in prop::collection::vec(any::<i32>(), 0..100)) {
        let original_len = vec.len();
        vec.sort();
        assert_eq!(vec.len(), original_len);
    }

    #[test]
    fn sort_produces_ordered_output(mut vec in prop::collection::vec(any::<i32>(), 0..100)) {
        vec.sort();
        for window in vec.windows(2) {
            assert!(window[0] <= window[1]);
        }
    }
}
```

### 自定义策略

```rust
use proptest::prelude::*;

fn valid_email() -> impl Strategy<Value = String> {
    ("[a-z]{1,10}", "[a-z]{1,5}")
        .prop_map(|(user, domain)| format!("{user}@{domain}.com"))
}

proptest! {
    #[test]
    fn accepts_valid_emails(email in valid_email()) {
        assert!(User::new("Test", &email).is_ok());
    }
}
```

## 使用 `mockall` 进行 Mocking

### 基于 Trait 的 Mocking

```rust
use mockall::{automock, predicate::eq};

#[automock]
trait UserRepository {
    fn find_by_id(&self, id: u64) -> Option<User>;
    fn save(&self, user: &User) -> Result<(), StorageError>;
}

#[test]
fn service_returns_user_when_found() {
    let mut mock = MockUserRepository::new();
    mock.expect_find_by_id()
        .with(eq(42))
        .times(1)
        .returning(|_| Some(User { id: 42, name: "Alice".into() }));

    let service = UserService::new(Box::new(mock));
    let user = service.get_user(42).unwrap();
    assert_eq!(user.name, "Alice");
}

#[test]
fn service_returns_none_when_not_found() {
    let mut mock = MockUserRepository::new();
    mock.expect_find_by_id()
        .returning(|_| None);

    let service = UserService::new(Box::new(mock));
    assert!(service.get_user(99).is_none());
}
```

## Doc 测试

### 可执行文档

```rust
/// 两个数字相加。
///
/// # Examples
///
/// ```
/// use my_crate::add;
///
/// assert_eq!(add(2, 3), 5);
/// assert_eq!(add(-1, 1), 0);
/// ```
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}

/// 解析 config 字符串。
///
/// # Errors
///
/// 如果输入不是有效的 TOML，则返回 `Err`。
///
/// ```no_run
/// use my_crate::parse_config;
///
/// let config = parse_config(r#"port = 8080"#).unwrap();
/// assert_eq!(config.port, 8080);
/// ```
///
/// ```no_run
/// use my_crate::parse_config;
///
/// assert!(parse_config("}{invalid").is_err());
/// ```
pub fn parse_config(input: &str) -> Result<Config, ParseError> {
    todo!()
}
```

## 使用 Criterion 进行 Benchmarking

```toml
# Cargo.toml
[dev-dependencies]
criterion = { version = "0.5", features = ["html_reports"] }

[[bench]]
name = "benchmark"
harness = false
```

```rust
// benches/benchmark.rs
use criterion::{black_box, criterion_group, criterion_main, Criterion};

fn fibonacci(n: u64) -> u64 {
    match n {
        0 | 1 => n,
        _ => fibonacci(n - 1) + fibonacci(n - 2),
    }
}

fn bench_fibonacci(c: &mut Criterion) {
    c.bench_function("fib 20", |b| b.iter(|| fibonacci(black_box(20))));
}

criterion_group!(benches, bench_fibonacci);
criterion_main!(benches);
```

## 测试覆盖

### 运行覆盖

```bash
# 安装: cargo install cargo-llvm-cov (或在 CI 中使用 taiki-e/install-action)
cargo llvm-cov                    # 摘要
cargo llvm-cov --html             # HTML 报告
cargo llvm-cov --lcov > lcov.info # 用于 CI 的 LCOV 格式
cargo llvm-cov --fail-under-lines 80  # 如果低于阈值则失败
```

### 覆盖目标

| 代码类型 | 目标 |
|----------|-------|
| 关键业务逻辑 | 100% |
| Public API | 90%+ |
| 通用代码 | 80%+ |
| 生成的 / FFI binding | 排除 |

## 测试命令

```bash
cargo test                        # 运行所有测试
cargo test -- --nocapture         # 显示 println 输出
cargo test test_name              # 运行匹配模式的测试
cargo test --lib                  # 仅 unit 测试
cargo test --test api_test        # 仅 integration 测试
cargo test --doc                  # 仅 doc 测试
cargo test --no-fail-fast         # 不在第一个失败时停止
cargo test -- --ignored           # 运行被忽略的测试
```

## 最佳实践

**要做：**
- 先写测试（TDD）
- 对 unit 测试使用 `#[cfg(test)]` 模块
- 测试行为，而不是实现
- 使用描述性测试名称来解释场景
- 优先使用 `assert_eq!` 而不是 `assert!` 以获得更好的错误消息
- 在返回 `Result` 的测试中使用 `?` 以获得更清晰的错误输出
- 保持测试独立 — 没有共享的可变状态

**不要做：**
- 如果可以测试 `Result::is_err()`，不要使用 `#[should_panic]`
- 不要 mock 一切 — 尽可能优先使用 integration 测试
- 不要忽略不稳定的测试 — 修复它们或隔离它们
- 不要在测试中使用 `sleep()` — 使用 channel、barrier 或 `tokio::time::pause()`
- 不要跳过错误路径测试

## CI 集成

```yaml
# GitHub Actions
test:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    - uses: dtolnay/rust-toolchain@stable
      with:
        components: clippy, rustfmt

    - name: Check formatting
      run: cargo fmt --check

    - name: Clippy
      run: cargo clippy -- -D warnings

    - name: Run tests
      run: cargo test

    - uses: taiki-e/install-action@cargo-llvm-cov

    - name: Coverage
      run: cargo llvm-cov --fail-under-lines 80
```

**记住：** 测试就是文档。它们展示了你的代码应该如何使用。写得清晰并保持更新。
