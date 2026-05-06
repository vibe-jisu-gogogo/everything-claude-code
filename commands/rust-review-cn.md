---
description: 针对所有权、生命周期、错误处理、unsafe使用和惯用模式的全面Rust代码审查。调用rust-reviewer代理。
---

# Rust 代码审查

此命令调用**rust-reviewer**代理进行全面的Rust特定代码审查。

## 命令功能

1. **验证自动化检查**：运行`cargo check`、`cargo clippy -- -D warnings`、`cargo fmt --check`和`cargo test`——如果任何一项失败则停止
2. **识别Rust改动**：通过`git diff HEAD~1`查找修改过的`.rs`文件（PR使用`git diff main...HEAD`）
3. **运行安全审计**：如果可用则执行`cargo audit`
4. **安全扫描**：检查unsafe使用、命令注入、硬编码密钥
5. **所有权审查**：分析不必要的克隆、生命周期问题、借用模式
6. **生成报告**：按严重程度对问题分类

## 何时使用

在以下情况使用`/rust-review`：
- 编写或修改Rust代码后
- 提交Rust改动前
- 审查包含Rust代码的PR时
- 上手新的Rust代码库时
- 学习惯用Rust模式时

## 审查类别

### CRITICAL (必须修复)
- 生产代码路径中未检查的`unwrap()`/`expect()`
- 没有`// SAFETY:`注释说明不变量的`unsafe`代码
- 查询中通过字符串插值导致的SQL注入
- `std::process::Command`中未验证输入导致的命令注入
- 硬编码凭据
- 通过原始指针导致的释放后使用

### HIGH (应该修复)
- 为满足借用检查器而进行的不必要`.clone()`
- 可以使用`&str`或`impl AsRef<str>`的地方使用了`String`参数
- 异步上下文中的阻塞操作（`std::thread::sleep`、`std::fs`）
- 共享类型缺少`Send`/`Sync`约束
- 对业务关键枚举使用通配符`_ =>`匹配
- 大型函数（超过50行）

### MEDIUM (建议考虑)
- 热点路径中的不必要分配
- 已知大小的情况下缺少`with_capacity`
- 无正当理由抑制clippy警告
- 公共API缺少`///`文档
- 对于忽略返回值很可能是bug的非`must_use`返回类型，考虑添加`#[must_use]`

## 运行的自动化检查

```bash
# 构建门禁（审查前必须通过）
cargo check

# lint检查和建议
cargo clippy -- -D warnings

# 格式化检查
cargo fmt --check

# 测试
cargo test

# 安全审计（如果可用）
if command -v cargo-audit >/dev/null; then cargo audit; else echo "cargo-audit not installed"; fi
```

## 使用示例

````text
用户: /rust-review

代理:
# Rust 代码审查报告

## 审查的文件
- src/service/user.rs（已修改）
- src/handler/api.rs（已修改）

## 静态分析结果
- 构建：成功
- Clippy：无警告
- 格式化：通过
- 测试：全部通过

## 发现的问题

[CRITICAL] 生产路径中未检查的unwrap
文件: src/service/user.rs:28
问题: 在数据库查询结果上使用`.unwrap()`
```rust
let user = db.find_by_id(id).unwrap();  // 用户不存在时会panic
```
修复: 通过上下文传播错误
```rust
let user = db.find_by_id(id)
    .context("failed to fetch user")?;
```

[HIGH] 不必要的克隆
文件: src/handler/api.rs:45
问题: 为满足借用检查器克隆String
```rust
let name = user.name.clone();
process(&user, &name);
```
修复: 重构以避免克隆
```rust
let result = process_name(&user.name);
use_user(&user, result);
```

## 总结
- CRITICAL: 1
- HIGH: 1
- MEDIUM: 0

建议: 阻止合并直到严重问题修复
````

## 批准标准

| 状态 | 条件 |
|--------|-----------|
| 批准 | 无CRITICAL或HIGH问题 |
| 警告 | 只有MEDIUM问题（谨慎合并） |
| 阻止 | 发现CRITICAL或HIGH问题 |

## 与其他命令的集成

- 首先使用`/rust-test`确保测试通过
- 如果出现构建错误使用`/rust-build`
- 提交前使用`/rust-review`
- 非Rust特定问题使用`/code-review`

## 相关资源

- 代理: `agents/rust-reviewer.md`
- 技能: `skills/rust-patterns/`, `skills/rust-testing/`
