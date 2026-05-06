---
description: 针对 idiomatic patterns、并发安全性、错误处理和安全性的全面 Go 代码审查。调用 go-reviewer 代理。
---

# Go 代码审查

此命令调用 **go-reviewer** 代理进行全面的 Go 语言特定代码审查。

## 命令功能

1. **识别 Go 变更**：通过 `git diff` 查找修改过的 `.go` 文件
2. **运行静态分析**：执行 `go vet`、`staticcheck` 和 `golangci-lint`
3. **安全扫描**：检查 SQL 注入、命令注入、竞态条件
4. **并发审查**：分析 goroutine 安全性、channel 使用、mutex 模式
5. **Idiomatic Go 检查**：验证代码遵循 Go 公约和最佳实践
6. **生成报告**：按严重程度对问题分类

## 使用时机

使用 `/go-review` 的场景：
- 编写或修改 Go 代码后
- 提交 Go 变更前
- 审查包含 Go 代码的拉取请求时
- 上手新的 Go 代码库时
- 学习 idiomatic Go 模式时

## 审查类别

### CRITICAL（必须修复）
- SQL/命令注入漏洞
- 无同步机制的竞态条件
- Goroutine 泄漏
- 硬编码凭证
- 不安全的指针使用
- 关键路径中忽略错误

### HIGH（应该修复）
- 缺少带上下文的错误包装
- 使用 panic 而非错误返回
- 上下文未传递
- 无缓冲 channel 导致死锁
- 接口未实现错误
- 缺少 mutex 保护

### MEDIUM（建议考虑）
- 非 idiomatic 代码模式
- 导出对象缺少 godoc 注释
- 低效的字符串拼接
- Slice 未预分配
- 未使用表驱动测试

## 执行的自动化检查

```bash
# Static analysis
go vet ./...

# Advanced checks (if installed)
staticcheck ./...
golangci-lint run

# Race detection
go build -race ./...

# Security vulnerabilities
govulncheck ./...
```

## 使用示例

```text
用户: /go-review

代理:
# Go 代码审查报告

## 已审查文件
- internal/handler/user.go (modified)
- internal/service/auth.go (modified)

## 静态分析结果
✓ go vet: 无问题
✓ staticcheck: 无问题

## 发现的问题

[CRITICAL] 竞态条件
文件: internal/service/auth.go:45
问题: 共享 map 在无同步机制下被访问
```go
var cache = map[string]*Session{}  // Concurrent access!

func GetSession(id string) *Session {
    return cache[id]  // Race condition
}
```
修复: 使用 sync.RWMutex 或 sync.Map
```go
var (
    cache   = map[string]*Session{}
    cacheMu sync.RWMutex
)

func GetSession(id string) *Session {
    cacheMu.RLock()
    defer cacheMu.RUnlock()
    return cache[id]
}
```

[HIGH] 缺少错误上下文
文件: internal/handler/user.go:28
问题: 返回的错误没有上下文
```go
return err  // No context
```
修复: 使用上下文包装
```go
return fmt.Errorf("get user %s: %w", userID, err)
```

## 总结
- CRITICAL: 1
- HIGH: 1
- MEDIUM: 0

建议: 失败：在修复 CRITICAL 问题前阻止合并
```

## 批准标准

| 状态 | 条件 |
|--------|-----------|
| 通过: 批准 | 无 CRITICAL 或 HIGH 级别问题 |
| 警告: 提醒 | 仅存在 MEDIUM 级别问题（谨慎合并） |
| 失败: 阻止 | 发现 CRITICAL 或 HIGH 级别问题 |

## 与其他命令的集成

- 首先使用 `/go-test` 确保测试通过
- 如果出现构建错误，使用 `/go-build`
- 提交前使用 `/go-review`
- 非 Go 特定问题使用 `/code-review`

## 相关资源

- 代理: `agents/go-reviewer.md`
- 技能: `skills/golang-patterns/`、`skills/golang-testing/`
