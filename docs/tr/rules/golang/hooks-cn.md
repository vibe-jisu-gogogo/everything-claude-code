---
paths:
  - "**/*.go"
  - "**/go.mod"
  - "**/go.sum"
---
# Go Hooks

> 此文件使用 Go 特定内容扩展了 [common/hooks.md](../common/hooks.md)。

## PostToolUse Hooks

在 `~/.claude/settings.json` 中配置：

- **gofmt/goimports**：在 Edit 后自动格式化 `.go` 文件
- **go vet**：编辑 `.go` 文件后运行静态分析
- **staticcheck**：在已修改的包中运行扩展静态检查
