---
description: 针对惯用模式、并发安全、错误处理、安全性的全面Go代码审查。调用go-reviewer代理。
---

# Go Code Review

此命令调用**go-reviewer**代理进行Go特有的全面代码审查。

## 此命令的功能

1. **识别Go变更**: 使用`git diff`检测已修改的`.go`文件
2. **执行静态分析**: 运行`go vet`、`staticcheck`、`golangci-lint`
3. **安全扫描**: 检查SQL注入、命令注入、竞态条件
4. **并发审查**: 分析goroutine安全性、channel使用、mutex模式
5. **Go惯用性检查**: 确保代码遵循Go约定和最佳实践
6. **生成报告**: 按严重程度对问题进行分类

## 使用时机

在以下情况下使用`/go-review`:
- 创建或修改Go代码后
- 提交Go变更之前
- 审查包含Go代码的拉取请求时
- 入职新的Go代码库时
- 学习惯用的Go模式时

## 审查类别

### CRITICAL (必须修复)
- SQL/命令注入漏洞
- 无同步的竞态条件
- goroutine泄漏
- 硬编码的凭证
- 不安全的指针使用
- 在关键路径忽略错误

### HIGH (推荐修复)
- 缺少包含上下文的错误包装
- 使用panic代替错误返回值
- 上下文未传递
- 可能导致死锁的无缓冲channel
- 接口未实现错误
- 缺少mutex保护

### MEDIUM (考虑修复)
- 非惯用的代码模式
- 导出时缺少godoc注释
- 低效的字符串拼接
- 未预先分配的切片
- 未使用表驱动测试

## 执行的自动检查

```bash
# 静态分析
go vet ./...

# 高级检查（如果已安装）
staticcheck ./...
golangci-lint run

# 竞态检测
go build -race ./...

# 安全漏洞
govulncheck ./...
```

## 使用示例

```text
User: /go-review

Agent:
# Go代码审查报告

## 审查的文件
- internal/handler/user.go (修改)
- internal/service/auth.go (修改)

## 静态分析结果
✓ go vet: 无问题
✓ staticcheck: 无问题

## 发现的问题

[CRITICAL] 竞态条件
文件: internal/service/auth.go:45
问题: 无同步访问共享映射
```go
var cache = map[string]*Session{}  // 并发访问!

func GetSession(id string) *Session {
    return cache[id]  // 竞态条件
}
```
修复: 使用sync.RWMutex或sync.Map
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
修复: 使用上下文包装
```go
return fmt.Errorf("get user %s: %w", userID, err)
```

## 摘要
- CRITICAL: 1
- HIGH: 1
- MEDIUM: 0

建议: FAIL: 在CRITICAL问题修复前阻止合并
```

## 批准标准

| 状态 | 条件 |
|--------|-----------|
| PASS: 批准 | 无CRITICAL或HIGH问题 |
| WARNING: 警告 | 仅有MEDIUM问题（谨慎合并） |
| FAIL: 阻止 | 发现CRITICAL或HIGH问题 |

## 与其他命令的集成

- 首先使用`/go-test`确保测试通过
- 出现构建错误时使用`/go-build`
- 在提交前使用`/go-review`
- 对于非Go特有的问题使用`/code-review`

## 相关

- 代理: `agents/go-reviewer.md`
- 技能: `skills/golang-patterns/`, `skills/golang-testing/`
