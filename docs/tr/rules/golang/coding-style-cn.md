---
paths:
  - "**/*.go"
  - "**/go.mod"
  - "**/go.sum"
---
# Go 编码风格

> 本文件基于 [common/coding-style.md](../common/coding-style.md) 扩展了 Go 特定内容。

## 格式化

- **gofmt** 和 **goimports** 是强制性的——无需讨论风格

## 设计原则

- 接受 interfaces，返回 structs
- 保持 interfaces 精简（1-3 个方法）

## 错误处理

始终使用 context 包装错误：

```go
if err != nil {
    return fmt.Errorf("failed to create user: %w", err)
}
```

## 参考

有关全面的 Go idioms 和 patterns，请参阅 skill: `golang-patterns` 文件。
