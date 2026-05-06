---
description: 针对内存安全、现代 C++ 惯用法、并发和安全性的全面 C++ 代码审查。调用 cpp-reviewer 代理。
---

# C++ 代码审查

此命令调用 **cpp-reviewer** 代理执行全面的 C++ 特定代码审查。

## 该命令功能

1. **识别 C++ 变更**: 通过 `git diff` 查找修改过的 `.cpp`、`.hpp`、`.cc`、`.h` 文件
2. **运行静态分析**: 执行 `clang-tidy` 和 `cppcheck`
3. **内存安全扫描**: 检查原始 new/delete、缓冲区溢出、释放后使用问题
4. **并发审查**: 分析线程安全性、互斥锁使用、数据竞争
5. **现代 C++ 检查**: 验证代码遵循 C++17/20 规范和最佳实践
6. **生成报告**: 按严重程度对问题进行分类

## 何时使用

使用 `/cpp-review` 的场景：
- 编写或修改 C++ 代码后
- 提交 C++ 变更前
- 审查包含 C++ 代码的 PR
- 熟悉新的 C++ 代码库时
- 检查内存安全问题时

## 审查分类

### CRITICAL (必须修复)
- 没有使用 RAII 的原始 `new`/`delete`
- 缓冲区溢出和释放后使用
- 未同步的数据竞争
- 通过 `system()` 的命令注入
- 读取未初始化变量
- 空指针解引用

### HIGH (应该修复)
- 违反五法则
- 缺少 `std::lock_guard` / `std::scoped_lock`
- 没有正确生命周期管理的分离线程
- 使用 C 风格类型转换而非 `static_cast`/`dynamic_cast`
- 缺少 `const` 正确性

### MEDIUM (建议考虑)
- 不必要的拷贝（按值传递而非 `const&`）
- 已知大小的容器缺少 `reserve()` 调用
- 头文件中使用 `using namespace std;`
- 重要返回值缺少 `[[nodiscard]]` 标记
- 过于复杂的模板元编程

## 执行的自动检查

```bash
# 静态分析
clang-tidy --checks='*,-llvmlibc-*' src/*.cpp -- -std=c++17

# 额外分析
cppcheck --enable=all --suppress=missingIncludeSystem src/

# 带警告编译
cmake --build build -- -Wall -Wextra -Wpedantic
```

## 使用示例

```text
用户: /cpp-review

代理:
# C++ 代码审查报告

## 审查文件
- src/handler/user.cpp (已修改)
- src/service/auth.cpp (已修改)

## 静态分析结果
✓ clang-tidy: 2 个警告
✓ cppcheck: 无问题

## 发现问题

[CRITICAL] 内存泄漏
文件: src/service/auth.cpp:45
问题: 原始 `new` 没有对应的 `delete`
```cpp
auto* session = new Session(userId);  // 内存泄漏！
cache[userId] = session;
```
修复方案: 使用 `std::unique_ptr`
```cpp
auto session = std::make_unique<Session>(userId);
cache[userId] = std::move(session);
```

[HIGH] 缺少 const 引用
文件: src/handler/user.cpp:28
问题: 大对象按值传递
```cpp
void processUser(User user) {  // 不必要的拷贝
```
修复方案: 按 const 引用传递
```cpp
void processUser(const User& user) {
```

## 总结
- CRITICAL: 1
- HIGH: 1
- MEDIUM: 0

建议: FAIL: 修复严重问题前禁止合并
```

## 批准标准

| 状态 | 条件 |
|--------|-----------|
| PASS: 批准 | 无 CRITICAL 或 HIGH 级别问题 |
| WARNING: 警告 | 仅存在 MEDIUM 级别问题（谨慎合并） |
| FAIL: 阻止 | 发现 CRITICAL 或 HIGH 级别问题 |

## 与其他命令的集成

- 先使用 `/cpp-test` 确保测试通过
- 出现构建错误时使用 `/cpp-build`
- 提交前使用 `/cpp-review`
- 非 C++ 特定问题使用 `/code-review`

## 相关内容

- 代理: `agents/cpp-reviewer.md`
- 技能: `skills/cpp-coding-standards/`、`skills/cpp-testing/`
