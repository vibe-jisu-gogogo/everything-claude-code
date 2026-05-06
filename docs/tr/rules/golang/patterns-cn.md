---
paths:
  - "**/*.go"
  - "**/go.mod"
  - "**/go.sum"
---
# Go 设计模式

> 本文件扩展了 [common/patterns.md](../common/patterns.md)，添加了 Go 特有的内容。

## Functional Options

```go
type Option func(*Server)

func WithPort(port int) Option {
    return func(s *Server) { s.port = port }
}

func NewServer(opts ...Option) *Server {
    s := &Server{port: 8080}
    for _, opt := range opts {
        opt(s)
    }
    return s
}
```

## 小型接口

在使用接口的地方定义接口，而不是在实现接口的地方定义。

## Dependency Injection

使用构造函数来注入依赖项：

```go
func NewUserService(repo UserRepository, logger Logger) *UserService {
    return &UserService{repo: repo, logger: logger}
}
```

## 参考

有关包括并发、错误处理和包组织在内的全面 Go 设计模式，请参阅 skill: `golang-patterns` 文件。
