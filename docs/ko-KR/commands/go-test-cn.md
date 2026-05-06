---
description: 强制 Go TDD 工作流。先编写表格驱动测试，再进行实现。通过 go test -cover 验证 80% 以上覆盖率。
---

# Go TDD 命令

此命令使用惯用的 Go 测试模式，对 Go 代码强制执行测试驱动开发方法论。

## 此命令的功能

1. **类型/接口定义**：首先构建函数签名框架
2. **编写表格驱动测试**：创建全面的测试用例 (RED)
3. **运行测试**：验证测试因正确原因失败
4. **代码实现**：编写最少的代码以通过测试 (GREEN)
5. **重构**：在保持测试通过的同时改进代码
6. **覆盖率检查**：确保 80% 以上覆盖率

## 使用时机

应使用 `/go-test` 的情况：
- 实现新的 Go 函数
- 为现有代码添加测试覆盖率
- 修复 bug（先编写失败的测试）
- 实现核心业务逻辑
- 学习 Go 中的 TDD 工作流

## TDD 循环

```
RED     → 编写失败的表格驱动测试
GREEN   → 实现最少代码以通过测试
REFACTOR → 改进代码，保持测试通过
REPEAT  → 进入下一个测试用例
```

## 示例会话

````
User: /go-test 我需要一个验证电子邮件地址的函数

Agent:
# TDD 会话：邮箱验证器

## 第 1 步：接口定义

```go
// validator/email.go
package validator

// ValidateEmail 检查给定字符串是否为有效的电子邮件地址。
// 如果有效则返回 nil，否则返回描述问题的 error。
func ValidateEmail(email string) error {
    panic("not implemented")
}
```

## 第 2 步：编写表格驱动测试 (RED)

```go
// validator/email_test.go
package validator

import (
    "testing"
)

func TestValidateEmail(t *testing.T) {
    tests := []struct {
        name    string
        email   string
        wantErr bool
    }{
        // 有效邮箱
        {"简单邮箱", "user@example.com", false},
        {"包含子域名", "user@mail.example.com", false},
        {"包含加号", "user+tag@example.com", false},
        {"包含点号", "first.last@example.com", false},

        // 无效邮箱
        {"空字符串", "", true},
        {"无 @ 符号", "userexample.com", true},
        {"无域名", "user@", true},
        {"无本地部分", "@example.com", true},
        {"双重 @", "user@@example.com", true},
        {"包含空格", "user @example.com", true},
        {"无 TLD", "user@example", true},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            err := ValidateEmail(tt.email)
            if tt.wantErr && err == nil {
                t.Errorf("ValidateEmail(%q) = nil; want error", tt.email)
            }
            if !tt.wantErr && err != nil {
                t.Errorf("ValidateEmail(%q) = %v; want nil", tt.email, err)
            }
        })
    }
}
```

## 第 3 步：运行测试 - 验证失败

```bash
$ go test ./validator/...

--- FAIL: TestValidateEmail (0.00s)
    --- FAIL: TestValidateEmail/简单_邮箱 (0.00s)
        panic: not implemented

FAIL
```

✓ 测试按预期失败 (panic)。

## 第 4 步：实现最少代码 (GREEN)

```go
// validator/email.go
package validator

import (
    "errors"
    "regexp"
)

var emailRegex = regexp.MustCompile(`^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`)

var (
    ErrEmailEmpty   = errors.New("email cannot be empty")
    ErrEmailInvalid = errors.New("email format is invalid")
)

func ValidateEmail(email string) error {
    if email == "" {
        return ErrEmailEmpty
    }
    if !emailRegex.MatchString(email) {
        return ErrEmailInvalid
    }
    return nil
}
```

## 第 5 步：运行测试 - 验证通过

```bash
$ go test ./validator/...

PASS
ok      project/validator    0.003s
```

✓ 所有测试通过！

## 第 6 步：检查覆盖率

```bash
$ go test -cover ./validator/...

PASS
coverage: 100.0% of statements
ok      project/validator    0.003s
```

✓ 覆盖率：100%

## TDD 完成！
````

## 测试模式

### 表格驱动测试
```go
tests := []struct {
    name     string
    input    InputType
    want     OutputType
    wantErr  bool
}{
    {"用例 1", input1, want1, false},
    {"用例 2", input2, want2, true},
}

for _, tt := range tests {
    t.Run(tt.name, func(t *testing.T) {
        got, err := Function(tt.input)
        // 断言
    })
}
```

### 并行测试
```go
for _, tt := range tests {
    tt := tt // 捕获
    t.Run(tt.name, func(t *testing.T) {
        t.Parallel()
        // 测试主体
    })
}
```

### 测试辅助函数
```go
func setupTestDB(t *testing.T) *sql.DB {
    t.Helper()
    db := createDB()
    t.Cleanup(func() { db.Close() })
    return db
}
```

## 覆盖率命令

```bash
# 基本覆盖率
go test -cover ./...

# 覆盖率配置文件
go test -coverprofile=coverage.out ./...

# 在浏览器中查看
go tool cover -html=coverage.out

# 按函数查看覆盖率
go tool cover -func=coverage.out

# 配合竞态检测
go test -race -cover ./...
```

## 覆盖率目标

| 代码类型 | 目标 |
|-----------|------|
| 核心业务逻辑 | 100% |
| 公开 API | 90%+ |
| 一般代码 | 80%+ |
| 生成的代码 | 排除 |

## TDD 最佳实践

**应该做：**
- 在实现之前先编写测试
- 每次更改后运行测试
- 使用表格驱动测试获得全面覆盖率
- 测试行为而非实现细节
- 包含边界情况（空值、nil、最大值）

**不应该做：**
- 在测试之前编写实现
- 跳过 RED 阶段
- 直接测试 private 函数
- 在测试中使用 `time.Sleep`
- 忽略不稳定的测试

## 相关命令

- `/go-build` - 修复构建错误
- `/go-review` - 实现后代码审查
- `/verify` - 完整验证循环

## 相关项目

- 技能：`skills/golang-testing/`
- 技能：`skills/tdd-workflow/`
