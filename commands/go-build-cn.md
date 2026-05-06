---
description: 增量修复 Go build 错误、go vet 警告和 linter 问题。调用 go-build-resolver 代理进行最小化的精准修复。
---

# Go 构建与修复

该命令调用 **go-build-resolver** 代理以最小化变更增量修复 Go build 错误。

## 命令功能

1. **运行诊断**：执行 `go build`、`go vet`、`staticcheck`
2. **解析错误**：按文件分组并按严重性排序
3. **增量修复**：一次修复一个错误
4. **验证每个修复**：每次变更后重新运行构建
5. **报告总结**：显示已修复和剩余的问题

## 适用场景

当出现以下情况时使用 `/go-build`：
- `go build ./...` 执行失败并报错
- `go vet ./...` 报告问题
- `golangci-lint run` 显示警告
- 模块依赖损坏
- 拉取变更后构建失败

## 运行的诊断命令

```bash
# Primary build check
go build ./...

# Static analysis
go vet ./...

# Extended linting (if available)
staticcheck ./...
golangci-lint run

# Module issues
go mod verify
go mod tidy -v
```

## 会话示例

```text
用户：/go-build

代理：
# Go 构建问题解决

## 初始诊断

```bash
$ go build ./...
internal/service/user.go:25:15: undefined: UserRepository
internal/handler/api.go:42:9: cannot use x (type string) as type int
internal/handler/api.go:58:2: missing return at end of function
```

发现错误：3 个

## 修复 1：未定义标识符

文件：internal/service/user.go:25
错误：undefined: UserRepository
原因：缺少导入

```go
// Added import
import "project/internal/repository"

// Changed
var repo UserRepository
// To
var repo repository.UserRepository
```

```bash
$ go build ./...
# 剩余 2 个错误
```

## 修复 2：类型不匹配

文件：internal/handler/api.go:42
错误：cannot use x (type string) as type int

```go
// Changed
count := params.Get("count")
// To
countStr := params.Get("count")
count, _ := strconv.Atoi(countStr)
```

```bash
$ go build ./...
# 剩余 1 个错误
```

## 修复 3：缺少返回值

文件：internal/handler/api.go:58
错误：missing return at end of function

```go
func GetUser(id string) (*User, error) {
    if id == "" {
        return nil, ErrInvalidID
    }
    user := findUser(id)
    // Added missing return
    return user, nil
}
```

```bash
$ go build ./...
# 构建成功！
```

## 最终验证

```bash
$ go vet ./...
# 无问题

$ go test ./...
ok      project/internal/service   0.015s
ok      project/internal/handler   0.023s
```

## 总结

| 指标 | 数量 |
|--------|-------|
| 已修复构建错误 | 3 |
| 已修复 Vet 警告 | 0 |
| 修改文件数 | 2 |
| 剩余问题数 | 0 |

构建状态：通过：成功
```

## 常见修复错误

| 错误 | 典型修复方案 |
|-------|-------------|
| `undefined: X` | 添加导入或修复拼写错误 |
| `cannot use X as Y` | 类型转换或修复赋值 |
| `missing return` | 添加返回语句 |
| `X does not implement Y` | 添加缺失的方法 |
| `import cycle` | 重构包结构 |
| `declared but not used` | 移除或使用变量 |
| `cannot find package` | 执行 `go get` 或 `go mod tidy` |

## 修复策略

1. **优先修复构建错误** - 代码必须可编译
2. **其次修复 Vet 警告** - 修复可疑代码结构
3. **最后修复 Lint 警告** - 代码风格和最佳实践
4. **一次修复一个问题** - 验证每个变更
5. **最小化变更** - 不进行重构，仅修复问题

## 停止条件

出现以下情况时代理将停止并报告：
- 3 次尝试后同一错误仍然存在
- 修复引入更多错误
- 需要进行架构变更
- 缺少外部依赖

## 相关命令

- `/go-test` - 构建成功后运行测试
- `/go-review` - 审查代码质量
- `verification-loop` 技能 - 完整验证循环

## 相关资源

- 代理：`agents/go-build-resolver.md`
- 技能：`skills/golang-patterns/`
