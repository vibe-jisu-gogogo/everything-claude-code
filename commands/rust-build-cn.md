---
description: 渐进式修复 Rust 构建错误、借用检查器问题和依赖问题。调用 rust-build-resolver agent 进行最小化、精准的修复。
---

# Rust 构建与修复

此命令调用 **rust-build-resolver** agent，以最小化的修改渐进式修复 Rust 构建错误。

## 命令功能

1. **运行诊断**：执行 `cargo check`、`cargo clippy`、`cargo fmt --check`
2. **解析错误**：识别错误代码和受影响的文件
3. **渐进式修复**：一次修复一个错误
4. **验证每个修复**：每次修改后重新运行 `cargo check`
5. **报告摘要**：显示已修复内容和剩余问题

## 何时使用

使用 `/rust-build` 当：
- `cargo build` 或 `cargo check` 报错时
- `cargo clippy` 报告警告时
- 借用检查器或生命周期错误阻止编译时
- Cargo 依赖解析失败时
- 拉取破坏构建的更改后

## 运行的诊断命令

```bash
# 主要构建检查
cargo check 2>&1

# Lints 和建议
cargo clippy -- -D warnings 2>&1

# 格式化检查
cargo fmt --check 2>&1

# 依赖问题
cargo tree --duplicates

# 安全审计（如果可用）
if command -v cargo-audit >/dev/null; then cargo audit; else echo "cargo-audit not installed"; fi
```

## 会话示例

````text
用户: /rust-build

Agent:
# Rust 构建解决方案

## 初始诊断

```bash
$ cargo check
error[E0502]: cannot borrow `map` as mutable because it is also borrowed as immutable
  --> src/service/cache.rs:25:5
error[E0308]: mismatched types
  --> src/handler/api.rs:42:12
error[E0425]: cannot find value `db` in this scope
  --> src/handler/api.rs:58:9
```

发现错误：3 个

## 修复 1：借用检查器错误

文件：src/service/cache.rs:25
错误：E0502 — 无法可变借用 `map` 因为已被不可变借用
原因：可变插入期间不可变借用仍处于活动状态

```rust
// 已更改：重构以在可变访问前结束不可变借用
if !map.contains_key("key") {
    map.insert("key".into(), default);
}
```

```bash
$ cargo check
# 剩余 2 个错误
```

## 修复 2：类型不匹配

文件：src/handler/api.rs:42
错误：E0308 — 类型不匹配：期望 `usize`，找到 `&str`

```rust
// 已更改
let count = params.get("count");
// 改为
let count: usize = params.get("count")
    .and_then(|s| s.parse().ok())
    .unwrap_or(0);
```

```bash
$ cargo check
# 剩余 1 个错误
```

## 修复 3：未解析的名称

文件：src/handler/api.rs:58
错误：E0425 — 在此作用域中找不到值 `db`
原因：缺少导入

```rust
// 已添加
use crate::db;
```

```bash
$ cargo check
# 构建成功！
```

## 最终验证

```bash
$ cargo clippy -- -D warnings
# 无警告

$ cargo test
running 6 tests
test service::cache::tests::test_insert ... ok
test service::cache::tests::test_get ... ok
test handler::api::tests::test_parse_count ... ok
test handler::api::tests::test_missing_count ... ok
test handler::api::tests::test_db_import ... ok
test handler::api::tests::test_response ... ok

test result: ok. 6 passed; 0 failed; 0 ignored
```

## 摘要

| 指标 | 数量 |
|------|------|
| 已修复构建错误 | 3 |
| 已修复 Clippy 警告 | 0 |
| 已修改文件 | 2 |
| 剩余问题 | 0 |

构建状态：成功
````

## 常见修复错误

| 错误 | 典型修复 |
|------|----------|
| `cannot borrow as mutable` | 重构以先结束不可变借用；仅在合理时克隆 |
| `does not live long enough` | 使用自有类型或添加生命周期注解 |
| `cannot move out of` | 重构以获取所有权；仅作为最后手段克隆 |
| `mismatched types` | 添加 `.into()`、`as` 或显式转换 |
| `trait X not implemented` | 添加 `#[derive(Trait)]` 或手动实现 |
| `unresolved import` | 添加到 Cargo.toml 或修复 `use` 路径 |
| `cannot find value` | 添加导入或修复路径 |

## 修复策略

1. **首先修复构建错误** - 代码必须能编译
2. **其次修复 Clippy 警告** - 修复可疑代码结构
3. **再次修复格式化** - 符合 `cargo fmt` 规范
4. **一次一个修复** - 验证每次更改
5. **最小化修改** - 不重构，只修复

## 停止条件

Agent 将停止并报告当：
- 3 次尝试后相同错误仍存在
- 修复引入更多错误
- 需要架构更改
- 借用检查器错误需要重新设计数据所有权

## 相关命令

- `/rust-test` - 构建成功后运行测试
- `/rust-review` - 审查代码质量
- `verification-loop` skill - 完整验证循环

## 相关

- Agent: `agents/rust-build-resolver.md`
- Skill: `skills/rust-patterns/`
