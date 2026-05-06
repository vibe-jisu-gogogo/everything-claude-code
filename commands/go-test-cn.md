---
description: 为 Go 强制执行 TDD 工作流。先编写 table-driven tests，再实现功能。使用 go test -cover 验证 80% 以上的覆盖率。
---

# Go TDD 命令

该命令使用符合 Go 语言习惯的测试模式，为 Go 代码强制执行测试驱动开发方法论。

## 命令功能

1. **定义类型/接口**：首先搭建函数签名框架
2. **编写 table-driven tests**：创建全面的测试用例（RED 阶段）
3. **运行测试**：验证测试按预期失败
4. **实现代码**：编写最少的代码使测试通过（GREEN 阶段）
5. **重构**：优化代码同时保持测试通过
6. **检查覆盖率**：确保覆盖率达到 80% 以上

## 适用场景

在以下场景使用 `/go-test`：
- 实现新的 Go 函数
- 为现有代码添加测试覆盖率
- 修复 bug（先编写失败的测试）
- 构建核心业务逻辑
- 学习 Go 中的 TDD 工作流

## TDD 周期

```
RED     → Write failing table-driven test
GREEN   → Implement minimal code to pass
REFACTOR → Improve code, tests stay green
REPEAT  → Next test case
```

## 示例会话

````
User: /go-test I need a function to validate email addresses

Agent:
# TDD Session: Email Validator

## Step 1: Define Interface

```go
// validator/email.go
package validator

// ValidateEmail checks if the given string is a valid email address.
// Returns nil if valid, error describing the issue otherwise.
func ValidateEmail(email string) error {
    panic("not implemented")
}
```

## Step 2: Write Table-Driven Tests (RED)

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
        // Valid emails
        {"simple email", "user@example.com", false},
        {"with subdomain", "user@mail.example.com", false},
        {"with plus", "user+tag@example.com", false},
        {"with dots", "first.last@example.com", false},

        // Invalid emails
        {"empty string", "", true},
        {"no at sign", "userexample.com", true},
        {"no domain", "user@", true},
        {"no local part", "@example.com", true},
        {"double at", "user@@example.com", true},
        {"spaces", "user @example.com", true},
        {"no tld", "user@example", true},
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

## Step 3: Run Tests - Verify FAIL

```bash
$ go test ./validator/...

--- FAIL: TestValidateEmail (0.00s)
    --- FAIL: TestValidateEmail/simple_email (0.00s)
        panic: not implemented

FAIL
```

✓ Tests fail as expected (panic).

## Step 4: Implement Minimal Code (GREEN)

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

## Step 5: Run Tests - Verify PASS

```bash
$ go test ./validator/...

PASS
ok      project/validator    0.003s
```

✓ All tests passing!

## Step 6: Check Coverage

```bash
$ go test -cover ./validator/...

PASS
coverage: 100.0% of statements
ok      project/validator    0.003s
```

✓ Coverage: 100%

## TDD Complete!
````

## 测试模式

### Table-Driven Tests
```go
tests := []struct {
    name     string
    input    InputType
    want     OutputType
    wantErr  bool
}{
    {"case 1", input1, want1, false},
    {"case 2", input2, want2, true},
}

for _, tt := range tests {
    t.Run(tt.name, func(t *testing.T) {
        got, err := Function(tt.input)
        // assertions
    })
}
```

### Parallel Tests
```go
for _, tt := range tests {
    tt := tt // Capture
    t.Run(tt.name, func(t *testing.T) {
        t.Parallel()
        // test body
    })
}
```

### Test Helpers
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
# Basic coverage
go test -cover ./...

# Coverage profile
go test -coverprofile=coverage.out ./...

# View in browser
go tool cover -html=coverage.out

# Coverage by function
go tool cover -func=coverage.out

# With race detection
go test -race -cover ./...
```

## 覆盖率目标

| 代码类型 | 目标 |
|---------|------|
| 核心业务逻辑 | 100% |
| 公共 APIs | 90%+ |
| 通用代码 | 80%+ |
| 生成的代码 | 排除 |

## TDD 最佳实践

**推荐做法：**
- 先写测试，再写任何实现代码
- 每次修改后运行测试
- 使用 table-driven tests 获得全面的覆盖率
- 测试行为，而非实现细节
- 包含边界情况（空值、nil、最大值）

**禁止做法：**
- 在测试之前编写实现代码
- 跳过 RED 阶段
- 直接测试 private 函数
- 在测试中使用 `time.Sleep`
- 忽略不稳定的测试

## 相关命令

- `/go-build` - 修复构建错误
- `/go-review` - 实现完成后审查代码
- `verification-loop` 技能 - 运行完整的验证循环

## 相关资源

- 技能：`skills/golang-testing/`
- 技能：`skills/tdd-workflow/`
