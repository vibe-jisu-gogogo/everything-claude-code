# Go Microservice — 项目 CLAUDE.md

> 这是使用 PostgreSQL、gRPC、Docker 的 Go 微服务实战示例。
> 复制到项目根目录并根据服务进行自定义。

## 项目概述

**技术栈:** Go 1.22+, PostgreSQL, gRPC + REST (grpc-gateway), Docker, sqlc (类型安全 SQL), Wire (依赖注入)

**架构:** 由 domain、repository、service、handler 层构成的 Clean Architecture。使用 gRPC 作为基本传输协议，并为外部客户端提供 REST gateway。

## 核心规则

### Go 规则

- 遵循 Effective Go 和 Go Code Review Comments 指南
- 使用 `errors.New` / `fmt.Errorf` 和 `%w` 进行错误包装 — 不要对错误进行字符串匹配
- 禁止使用 `init()` 函数 — 在 `main()` 或构造函数中显式初始化
- 禁止全局可变状态 — 通过构造函数传递依赖
- Context 必须作为第一个参数并在所有层中传播

### 数据库

- 所有查询都以纯 SQL 形式写在 `queries/` 中 — sqlc 生成类型安全的 Go 代码
- 迁移使用 golang-migrate 并放在 `migrations/` 中 — 不要直接修改数据库
- 多阶段操作使用 `pgx.Tx` 事务
- 所有查询使用 parameterized placeholder (`$1`, `$2`) — 禁止使用字符串格式化

### 错误处理

- 返回错误，不要 panic — panic 仅用于真正不可恢复的情况
- 带上下文的错误包装: `fmt.Errorf("creating user: %w", err)`
- 业务逻辑的 sentinel 错误在 `domain/errors.go` 中定义
- 在 handler 层将 domain 错误映射为 gRPC status 代码

```go
// Domain 层 — sentinel 错误
var (
    ErrUserNotFound  = errors.New("user not found")
    ErrEmailTaken    = errors.New("email already registered")
)

// Handler 层 — 映射为 gRPC status
func toGRPCError(err error) error {
    switch {
    case errors.Is(err, domain.ErrUserNotFound):
        return status.Error(codes.NotFound, err.Error())
    case errors.Is(err, domain.ErrEmailTaken):
        return status.Error(codes.AlreadyExists, err.Error())
    default:
        return status.Error(codes.Internal, "internal error")
    }
}
```

### 代码风格

- 禁止在代码或注释中使用 emoji
- 对外公开的类型和函数必须写 doc 注释
- 函数保持在 50 行以内 — 拆分为 helper 函数
- 所有有多 case 的逻辑使用 table-driven 测试
- signal channel 使用 `struct{}` 而非 `bool`

## 文件结构

```
cmd/
  server/
    main.go              # 入口点, Wire 注入, 优雅关闭
internal/
  domain/                # 业务类型和接口
    user.go              # User 实体和 repository 接口
    errors.go            # Sentinel 错误
  service/               # 业务逻辑
    user_service.go
    user_service_test.go
  repository/            # 数据访问 (sqlc 生成 + 自定义)
    postgres/
      user_repo.go
      user_repo_test.go  # 使用 testcontainers 的集成测试
  handler/               # gRPC + REST handler
    grpc/
      user_handler.go
    rest/
      user_handler.go
  config/                # 配置加载
    config.go
proto/                   # Protobuf 定义
  user/v1/
    user.proto
queries/                 # sqlc 用 SQL 查询
  user.sql
migrations/              # 数据库迁移
  001_create_users.up.sql
  001_create_users.down.sql
```

## 核心模式

### Repository 接口

```go
type UserRepository interface {
    Create(ctx context.Context, user *User) error
    FindByID(ctx context.Context, id uuid.UUID) (*User, error)
    FindByEmail(ctx context.Context, email string) (*User, error)
    Update(ctx context.Context, user *User) error
    Delete(ctx context.Context, id uuid.UUID) error
}
```

### 使用依赖注入的 Service

```go
type UserService struct {
    repo   domain.UserRepository
    hasher PasswordHasher
    logger *slog.Logger
}

func NewUserService(repo domain.UserRepository, hasher PasswordHasher, logger *slog.Logger) *UserService {
    return &UserService{repo: repo, hasher: hasher, logger: logger}
}

func (s *UserService) Create(ctx context.Context, req CreateUserRequest) (*domain.User, error) {
    existing, err := s.repo.FindByEmail(ctx, req.Email)
    if err != nil && !errors.Is(err, domain.ErrUserNotFound) {
        return nil, fmt.Errorf("checking email: %w", err)
    }
    if existing != nil {
        return nil, domain.ErrEmailTaken
    }

    hashed, err := s.hasher.Hash(req.Password)
    if err != nil {
        return nil, fmt.Errorf("hashing password: %w", err)
    }

    user := &domain.User{
        ID:       uuid.New(),
        Name:     req.Name,
        Email:    req.Email,
        Password: hashed,
    }
    if err := s.repo.Create(ctx, user); err != nil {
        return nil, fmt.Errorf("creating user: %w", err)
    }
    return user, nil
}
```

### Table-Driven 测试

```go
func TestUserService_Create(t *testing.T) {
    tests := []struct {
        name    string
        req     CreateUserRequest
        setup   func(*MockUserRepo)
        wantErr error
    }{
        {
            name: "valid user",
            req:  CreateUserRequest{Name: "Alice", Email: "alice@example.com", Password: "secure123"},
            setup: func(m *MockUserRepo) {
                m.On("FindByEmail", mock.Anything, "alice@example.com").Return(nil, domain.ErrUserNotFound)
                m.On("Create", mock.Anything, mock.Anything).Return(nil)
            },
            wantErr: nil,
        },
        {
            name: "duplicate email",
            req:  CreateUserRequest{Name: "Alice", Email: "taken@example.com", Password: "secure123"},
            setup: func(m *MockUserRepo) {
                m.On("FindByEmail", mock.Anything, "taken@example.com").Return(&domain.User{}, nil)
            },
            wantErr: domain.ErrEmailTaken,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            repo := new(MockUserRepo)
            tt.setup(repo)
            svc := NewUserService(repo, &bcryptHasher{}, slog.Default())

            _, err := svc.Create(context.Background(), tt.req)

            if tt.wantErr != nil {
                assert.ErrorIs(t, err, tt.wantErr)
            } else {
                assert.NoError(t, err)
            }
        })
    }
}
```

## 环境变量

```bash
# 数据库
DATABASE_URL=postgres://user:pass@localhost:5432/myservice?sslmode=disable

# gRPC
GRPC_PORT=50051
REST_PORT=8080

# 认证
JWT_SECRET=           # 生产环境中从 vault 加载
TOKEN_EXPIRY=24h

# 可观测性
LOG_LEVEL=info        # debug, info, warn, error
OTEL_ENDPOINT=        # OpenTelemetry collector
```

## 测试策略

```bash
/go-test             # Go 用 TDD 工作流
/go-review           # Go 专用代码 review
/go-build            # 修复构建错误
```

### 测试命令

```bash
# 单元测试 (快速, 无外部依赖)
go test ./internal/... -short -count=1

# 集成测试 (testcontainers 需要 Docker)
go test ./internal/repository/... -count=1 -timeout 120s

# 全部测试和覆盖率
go test ./... -coverprofile=coverage.out -count=1
go tool cover -func=coverage.out  # 摘要
go tool cover -html=coverage.out  # 浏览器

# Race detector
go test ./... -race -count=1
```

## ECC 工作流

```bash
# 制定计划
/plan "Add rate limiting to user endpoints"

# 开发
/go-test                  # Go 专用模式的 TDD

# Review
/go-review                # Go idioms, 错误处理, 并发
/security-scan            # 密钥和漏洞检查

# 合并前检查
go vet ./...
staticcheck ./...
```

## Git 工作流

- `feat:` 新功能, `fix:` 修复 bug, `refactor:` 代码变更
- 从 `main` 创建 feature 分支, PR 必须
- CI: `go vet`, `staticcheck`, `go test -race`, `golangci-lint`
- 部署: CI 中构建 Docker 镜像, 部署到 Kubernetes
