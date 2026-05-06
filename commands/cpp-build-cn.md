---
description: 增量修复 C++ 构建错误、CMake 问题和链接器问题。调用 cpp-build-resolver 代理进行最小化的精准修复。
---

# C++ 构建与修复

此命令调用 **cpp-build-resolver** 代理，以最小化的变更增量修复 C++ 构建错误。

## 命令功能

1. **运行诊断**：执行 `cmake --build`、`clang-tidy`、`cppcheck`
2. **解析错误**：按文件分组并按严重程度排序
3. **增量修复**：每次修复一个错误
4. **验证每次修复**：每次修改后重新运行构建
5. **报告摘要**：显示已修复内容和剩余问题

## 适用场景

当出现以下情况时使用 `/cpp-build`：
- `cmake --build build` 运行失败并报错
- 链接器错误（未定义引用、多重定义）
- 模板实例化失败
- 头文件/依赖问题
- 拉取的代码变更导致构建失败后

## 运行的诊断命令

```bash
# CMake configure
cmake -B build -S .

# Build
cmake --build build 2>&1 | head -100

# Static analysis (if available)
clang-tidy src/*.cpp -- -std=c++17
cppcheck --enable=all src/
```

## 会话示例

```text
用户: /cpp-build

代理:
# C++ 构建问题解决

## 初始诊断

```bash
$ cmake --build build
src/service/user.cpp:25:15: error: use of undeclared identifier 'UserRepository'
src/handler/api.cpp:42:9: error: no matching function for call to 'process'
src/handler/api.cpp:58:1: error: non-void function does not return a value
```

发现错误: 3个

## 修复 1：未声明标识符

文件: src/service/user.cpp:25
错误：使用了未声明的标识符 'UserRepository'
原因：缺少头文件引用

```cpp
// 添加了头文件引用
#include "repository/user_repository.hpp"
```

```bash
$ cmake --build build
# 剩余 2 个错误
```

## 修复 2：无匹配函数

文件: src/handler/api.cpp:42
错误：找不到匹配的 'process' 函数调用

```cpp
// 原代码
process(params.get("count"));
// 修改为
process(std::stoi(params.get("count")));
```

```bash
$ cmake --build build
# 剩余 1 个错误
```

## 修复 3：缺少返回值

文件: src/handler/api.cpp:58
错误：非 void 函数没有返回值

```cpp
std::optional<User> getUser(const std::string& id) {
    if (id.empty()) {
        return std::nullopt;
    }
    auto user = findUser(id);
    // 添加了缺失的返回语句
    return user;
}
```

```bash
$ cmake --build build
# 构建成功！
```

## 最终验证

```bash
$ ctest --test-dir build --output-on-failure
测试项目构建
    1/5 Test #1: unit_tests ........   通过    0.02 秒
    2/5 Test #2: integration_tests    通过    0.15 秒
所有测试通过。
```

## 摘要

| 指标 | 数量 |
|--------|-------|
| 已修复构建错误 | 3 |
| 已修复链接器错误 | 0 |
| 修改文件数 | 2 |
| 剩余问题 | 0 |

构建状态：PASS：成功
```

## 常见修复的错误

| 错误 | 典型修复方案 |
|-------|-------------|
| `undeclared identifier` | 添加 `#include` 或修复拼写错误 |
| `no matching function` | 修复参数类型或添加重载 |
| `undefined reference` | 链接库或添加实现 |
| `multiple definition` | 使用 `inline` 或移动到 .cpp 文件 |
| `incomplete type` | 用 `#include` 替换前向声明 |
| `no member named X` | 修复成员名称或添加头文件引用 |
| `cannot convert X to Y` | 添加适当的类型转换 |
| `CMake Error` | 修复 CMakeLists.txt 配置 |

## 修复策略

1. **优先处理编译错误** - 代码必须先通过编译
2. **其次处理链接器错误** - 解决未定义引用问题
3. **再次处理警告** - 使用 `-Wall -Wextra` 修复警告
4. **每次修复一个问题** - 验证每次修改
5. **最小化变更** - 不要重构，只做必要修复

## 停止条件

当出现以下情况时，代理将停止并报告：
- 尝试 3 次后相同错误仍然存在
- 修复引入了更多错误
- 需要进行架构变更
- 缺少外部依赖

## 相关命令

- `/cpp-test` - 构建成功后运行测试
- `/cpp-review` - 审查代码质量
- `verification-loop` 技能 - 完整验证循环

## 相关资源

- 代理：`agents/cpp-build-resolver.md`
- 技能：`skills/cpp-coding-standards/`
