---
description: 针对惯用模式、并发安全性、错误处理、安全的全面 Go 代码审查。调用 go-reviewer 代理。
---

# Go 代码审查

此命令调用 **go-reviewer** 代理执行 Go 专属的全面代码审查。

## 此命令的功能

1. **识别 Go 变更**: 通过 `git diff` 查找已修改的 `.go` 文件
2. **运行静态分析**: 执行 `go vet`、`staticcheck`、`golangci-lint`
3. **安全扫描**: 检查 SQL 注入、命令注入、竞态条件
4. **并发审查**: 分析 goroutine 安全性、channel 使用、mutex 模式
5. **惯用 Go 检查**: 验证是否符合 Go 约定和最佳实践
6. **生成报告**: 按严重程度对问题进行分类

## 使用时机

应使用 `/go-review` 的情况:
- 编写或修改 Go 代码后
- 提交 Go 变更前
- 审查包含 Go 代码的 PR 时
- 加入新 Go 代码库时
- 学习惯用 Go 模式时

## 审查类别

### CRITICAL (必须修复)
- SQL/命令注入漏洞
- 无同步的竞态条件
- goroutine 泄漏
- 硬编码的凭证
- unsafe 指针使用
- 关键路径中忽略错误

### HIGH (建议修复)
- 缺少无上下文的错误包装
- 使用 panic 代替返回错误
- 缺少 context 传播
- 可能导致死锁的无缓冲 channel
- 接口未满足错误
- 缺少 mutex 保护

### MEDIUM (考虑)
- 非惯用代码模式
- 公开项缺少 godoc 注释
- 低效的字符串拼接
- 缺少 slice 预分配
- 未使用表驱动测试

## 执行的自动检查

```bash
# 静态分析
go vet ./...

# 高级检查 (如已安装)
staticcheck ./...
golangci-lint run

# 竞态检测
go build -race ./...

# 安全漏洞
govulncheck ./...
```

## 使用示例

````text
User: /go-review

Agent:
# Go 代码审查报告

## 已审查文件
- internal/handler/user.go (已修改)
- internal/service/auth.go (已修改)

## 静态分析结果
✓ go vet: 无问题
✓ staticcheck: 无问题

## 发现的问题

[CRITICAL] 竞态条件
文件: internal/service/auth.go:45
问题: 无同步访问共享 map
```go
var cache = map[string]*Session{}  // 并发访问！

func GetSession(id string) *Session {
    return cache[id]  // 竞态条件
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
问题: 返回无上下文的错误
```go
return err  // 无上下文
```
修复: 带上下文包装
```go
return fmt.Errorf("get user %s: %w", userID, err)
```

## 摘要
- CRITICAL: 1
- HIGH: 1
- MEDIUM: 0

建议: FAIL: 修复 CRITICAL 问题前阻止 merge
````

## 批准标准

| 状态 | 条件 |
|------|------|
| PASS: 批准 | 无 CRITICAL 或 HIGH 问题 |
| WARNING: 警告 | 仅有 MEDIUM 问题 (谨慎 merge) |
| FAIL: 阻止 | 发现 CRITICAL 或 HIGH 问题 |

## 与其他命令的配合

- 先使用 `/go-test` 确认测试通过
- 使用 `/go-build` 修复构建错误
- 提交前使用 `/go-review`
- 使用 `/code-review` 审查 Go 以外的一般关注点

## 相关项

- 代理: `agents/go-reviewer.md`
- 技能: `skills/golang-patterns/`、`skills/golang-testing/`
