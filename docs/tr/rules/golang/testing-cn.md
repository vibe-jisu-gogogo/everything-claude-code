---
paths:
  - "**/*.go"
  - "**/go.mod"
  - "**/go.sum"
---
# Go Testing

> 此文件扩展了 [common/testing.md](../common/testing.md)，添加了 Go 特定内容。

## Framework

使用标准的 `go test` 和 **Table-driven tests**。

## Race Detection

始终使用 `-race` 标志运行：

```bash
go test -race ./...
```

## Coverage

```bash
go test -cover ./...
```

## 参考

有关详细的 Go test patterns 和 helpers，请参阅 skill: `golang-testing` 文件。
