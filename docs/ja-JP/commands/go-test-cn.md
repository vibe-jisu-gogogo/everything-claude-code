---
description: 应用Go的测试驱动开发(TDD)工作流。首先编写表驱动测试，然后进行实现。使用go test -cover确保80%以上的覆盖率。
---

# Go TDD 命令

本命令应用使用Go惯用测试模式的测试驱动开发方法。

## 本命令的功能

1. **类型/接口定义**: 首先构建函数签名
2. **创建表驱动测试**: 创建全面的测试用例(RED)
3. **执行测试**: 确认测试因正确原因失败
4. **代码实现**: 编写使测试通过的最小代码(GREEN)
5. **重构**: 在保持测试通过的同时进行改进
6. **确认覆盖率**: 保证80%以上的覆盖率

## 使用时机

在以下情况使用`/go-test`:
- 实现新的Go函数时
- 为现有代码添加测试覆盖率时
- 修复bug时(首先编写失败的测试)
- 构建重要业务逻辑时
- 学习Go的TDD工作流时

## TDD 周期

```
RED     → 创建失败的表驱动测试
GREEN   → 实现使测试通过的最小代码
REFACTOR → 改进代码，保持测试通过
REPEAT  → 进入下一个测试用例
```

## 会话示例

```text
User: /go-test 需要一个验证邮箱地址的函数

Agent:
# TDD 会话: 邮箱验证器

## 步骤 1: 定义接口

```go
// validator/email.go
package validator

// ValidateEmail checks if the given string is a valid email address.
// Returns nil if valid, error describing the issue otherwise.
func ValidateEmail(email string) error {
    panic("not implemented")
}
```

## 步骤 2: 创建表驱动测试 (RED)

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
        // 有效邮箱地址
        {"simple email", "user@example.com", false},
        {"with subdomain", "user@mail.example.com", false},
        {"with plus", "user+tag@example.com", false},
        {"with dots", "first.last@example.com", false},

        // 无效邮箱地址
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

## 步骤 3: 执行测试 - 确认失败

```bash
$ go test ./validator/...

--- FAIL: TestValidateEmail (0.00s)
    --- FAIL: TestValidateEmail/simple_email (0.00s)
        panic: not implemented

FAIL
```

✓ 测试按预期失败(panic)。

## 步骤 4: 实现最小代码 (GREEN)

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

## 步骤 5: 执行测试 - 确认通过

```bash
$ go test ./validator/...

PASS
ok      project/validator    0.003s
```

✓ 所有测试通过!

## 步骤 6: 确认覆盖率

```bash
$ go test -cover ./validator/...

PASS
coverage: 100.0% of statements
ok      project/validator    0.003s
```

✓ 覆盖率: 100%

## TDD 完成!
```

## 测试模式

### 表驱动测试
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

### 并行测试
```go
for _, tt := range tests {
    tt := tt // Capture
    t.Run(tt.name, func(t *testing.T) {
        t.Parallel()
        // test body
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

# 覆盖率分析
go test -coverprofile=coverage.out ./...

# 在浏览器中显示
go tool cover -html=coverage.out

# 按函数显示覆盖率
go tool cover -func=coverage.out

# 带竞态检测
go test -race -cover ./...
```

## 覆盖率目标

| 代码类型 | 目标 |
|-----------|--------|
| 重要业务逻辑 | 100% |
| 公共API | 90%+ |
| 一般代码 | 80%+ |
| 生成代码 | 除外 |

## TDD 最佳实践

**推荐事项:**
- 在实现前首先编写测试
- 每次更改后执行测试
- 使用表驱动测试以获得全面覆盖率
- 测试行为而非实现细节
- 包含边界情况(空、nil、最大值)

**应避免事项:**
- 在测试前编写实现
- 跳过RED阶段
- 直接测试私有函数
- 在测试中使用`time.Sleep`
- 忽略不稳定的测试

## 相关命令

- `/go-build` - 修复构建错误
- `/go-review` - 实现后的代码审查
- `/verify` - 执行完整的验证循环

## 相关

- 技能: `skills/golang-testing/`
- 技能: `skills/tdd-workflow/`
