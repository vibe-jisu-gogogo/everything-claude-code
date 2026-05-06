---
description: 为C++强制执行TDD工作流。先编写GoogleTest测试，再实现代码。使用gcov/lcov验证覆盖率。
---

# C++ TDD 命令

该命令使用GoogleTest/GoogleMock和CMake/CTest，为C++代码强制执行测试驱动开发方法论。

## 该命令功能

1. **定义接口**：首先构建类/函数签名脚手架
2. **编写测试**：创建完整的GoogleTest测试用例（RED阶段）
3. **运行测试**：验证测试因正确原因失败
4. **实现代码**：编写最少代码使测试通过（GREEN阶段）
5. **重构**：在保持测试通过的前提下改进代码
6. **检查覆盖率**：确保覆盖率达到80%以上

## 何时使用

在以下场景使用 `/cpp-test`：
- 实现新的C++函数或类
- 为现有代码添加测试覆盖率
- 修复bug（先编写失败的测试）
- 构建核心业务逻辑
- 学习C++中的TDD工作流

## TDD 循环

```
RED     → Write failing GoogleTest test
GREEN   → Implement minimal code to pass
REFACTOR → Improve code, tests stay green
REPEAT  → Next test case
```

## 示例会话

````
用户：/cpp-test 我需要一个验证电子邮件地址的函数

助理：
# TDD 会话：电子邮件验证器

## 步骤1：定义接口

```cpp
// validator/email.hpp
#pragma once
#include <string>
#include <expected>

enum class EmailError {
    Empty,
    InvalidFormat
};

std::expected<void, EmailError> validate_email(const std::string& email);
```

## 步骤2：编写测试（RED阶段）

```cpp
// validator/email_test.cpp
#include <gtest/gtest.h>
#include "email.hpp"

TEST(ValidateEmail, AcceptsSimpleEmail) {
    auto result = validate_email("user@example.com");
    EXPECT_TRUE(result.has_value());
}

TEST(ValidateEmail, AcceptsSubdomain) {
    EXPECT_TRUE(validate_email("user@mail.example.com").has_value());
}

TEST(ValidateEmail, AcceptsPlus) {
    EXPECT_TRUE(validate_email("user+tag@example.com").has_value());
}

TEST(ValidateEmail, RejectsEmpty) {
    auto result = validate_email("");
    ASSERT_FALSE(result.has_value());
    EXPECT_EQ(result.error(), EmailError::Empty);
}

TEST(ValidateEmail, RejectsNoAtSign) {
    EXPECT_FALSE(validate_email("userexample.com").has_value());
}

TEST(ValidateEmail, RejectsNoDomain) {
    EXPECT_FALSE(validate_email("user@").has_value());
}

TEST(ValidateEmail, RejectsNoLocalPart) {
    EXPECT_FALSE(validate_email("@example.com").has_value());
}
```

## 步骤3：运行测试 - 验证失败

```bash
$ cmake --build build && ctest --test-dir build --output-on-failure

1/1 Test #1: email_validator_test .....***Failed
    --- undefined reference to `validate_email`

FAIL
```

✓ 测试如预期失败（未实现）。

## 步骤4：实现最少代码（GREEN阶段）

```cpp
// validator/email.cpp
#include "email.hpp"
#include <regex>

std::expected<void, EmailError> validate_email(const std::string& email) {
    if (email.empty()) {
        return std::unexpected(EmailError::Empty);
    }
    static const std::regex pattern(R"([a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,})");
    if (!std::regex_match(email, pattern)) {
        return std::unexpected(EmailError::InvalidFormat);
    }
    return {};
}
```

## 步骤5：运行测试 - 验证通过

```bash
$ cmake --build build && ctest --test-dir build --output-on-failure

1/1 Test #1: email_validator_test .....   Passed    0.01 sec

100% tests passed.
```

✓ 所有测试通过！

## 步骤6：检查覆盖率

```bash
$ cmake -DCMAKE_CXX_FLAGS="--coverage" -B build && cmake --build build
$ ctest --test-dir build
$ lcov --capture --directory build --output-file coverage.info
$ lcov --list coverage.info

validator/email.cpp     | 100%
```

✓ 覆盖率：100%

## TDD 完成！
````

## 测试模式

### 基础测试
```cpp
TEST(SuiteName, TestName) {
    EXPECT_EQ(add(2, 3), 5);
    EXPECT_NE(result, nullptr);
    EXPECT_TRUE(is_valid);
    EXPECT_THROW(func(), std::invalid_argument);
}
```

### 测试夹具
```cpp
class DatabaseTest : public ::testing::Test {
protected:
    void SetUp() override { db_ = create_test_db(); }
    void TearDown() override { db_.reset(); }
    std::unique_ptr<Database> db_;
};

TEST_F(DatabaseTest, InsertsRecord) {
    db_->insert("key", "value");
    EXPECT_EQ(db_->get("key"), "value");
}
```

### 参数化测试
```cpp
class PrimeTest : public ::testing::TestWithParam<std::pair<int, bool>> {};

TEST_P(PrimeTest, ChecksPrimality) {
    auto [input, expected] = GetParam();
    EXPECT_EQ(is_prime(input), expected);
}

INSTANTIATE_TEST_SUITE_P(Primes, PrimeTest, ::testing::Values(
    std::make_pair(2, true),
    std::make_pair(4, false),
    std::make_pair(7, true)
));
```

## 覆盖率命令

```bash
# Build with coverage
cmake -DCMAKE_CXX_FLAGS="--coverage" -DCMAKE_EXE_LINKER_FLAGS="--coverage" -B build

# Run tests
cmake --build build && ctest --test-dir build

# Generate coverage report
lcov --capture --directory build --output-file coverage.info
lcov --remove coverage.info '/usr/*' --output-file coverage.info
genhtml coverage.info --output-directory coverage_html
```

## 覆盖率目标

| 代码类型 | 目标 |
|-----------|--------|
| 核心业务逻辑 | 100% |
| 公共API | 90%以上 |
| 通用代码 | 80%以上 |
| 生成的代码 | 排除 |

## TDD 最佳实践

**应该：**
- 先写测试，再写任何实现代码
- 每次变更后运行测试
- 适当情况下优先使用`EXPECT_*`（失败后继续执行）而非`ASSERT_*`（失败后停止）
- 测试行为，而非实现细节
- 包含边界用例（空值、null、最大值、边界条件）

**不应该：**
- 在测试之前编写实现代码
- 跳过RED阶段
- 直接测试私有方法（通过公共API测试）
- 在测试中使用`sleep`
- 忽略不稳定的测试

## 相关命令

- `/cpp-build` - 修复构建错误
- `/cpp-review` - 实现完成后评审代码
- `verification-loop` 技能 - 运行完整验证循环

## 相关内容

- 技能：`skills/cpp-testing/`
- 技能：`skills/tdd-workflow/`
