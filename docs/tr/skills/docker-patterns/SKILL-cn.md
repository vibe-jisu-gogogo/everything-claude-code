---
name: docker-patterns
description: 用于本地开发、容器安全、网络、volume 策略和多服务编排的 Docker 和 Docker Compose 模式。
origin: ECC
---

# Docker 模式

用于容器化开发的 Docker 和 Docker Compose 最佳实践。

## 何时激活

- 为本地开发设置 Docker Compose 时
- 设计多容器架构时
- 排查容器网络或 volume 问题时
- 审查 Dockerfile 的安全性和大小时
- 从本地开发迁移到容器化工作流时

## 本地开发的 Docker Compose

### 标准 Web 应用栈

```yaml
# docker-compose.yml
services:
  app:
    build:
      context: .
      target: dev                     # 使用多阶段 Dockerfile 的 dev 阶段
    ports:
      - "3000:3000"
    volumes:
      - .:/app                        # 用于热重载的 bind mount
      - /app/node_modules             # 匿名 volume -- 保留容器依赖项
    environment:
      - DATABASE_URL=postgres://postgres:postgres@db:5432/app_dev
      - REDIS_URL=redis://redis:6379/0
      - NODE_ENV=development
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
    command: npm run dev

  db:
    image: postgres:16-alpine
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: app_dev
    volumes:
      - pgdata:/var/lib/postgresql/data
      - ./scripts/init-db.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 3s
      retries: 5

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redisdata:/data

  mailpit:                            # 本地 email 测试
    image: axllent/mailpit
    ports:
      - "8025:8025"                   # Web UI
      - "1025:1025"                   # SMTP

volumes:
  pgdata:
  redisdata:
```

### 开发 vs 生产 Dockerfile

```dockerfile
# 阶段：依赖
FROM node:22-alpine AS deps
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci

# 阶段：dev（热重载、调试工具）
FROM node:22-alpine AS dev
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
EXPOSE 3000
CMD ["npm", "run", "dev"]

# 阶段：build
FROM node:22-alpine AS build
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build && npm prune --production

# 阶段：production（最小镜像）
FROM node:22-alpine AS production
WORKDIR /app
RUN addgroup -g 1001 -S appgroup && adduser -S appuser -u 1001
USER appuser
COPY --from=build --chown=appuser:appgroup /app/dist ./dist
COPY --from=build --chown=appuser:appgroup /app/node_modules ./node_modules
COPY --from=build --chown=appuser:appgroup /app/package.json ./
ENV NODE_ENV=production
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=3s CMD wget -qO- http://localhost:3000/health || exit 1
CMD ["node", "dist/server.js"]
```

### Override 文件

```yaml
# docker-compose.override.yml（自动加载，仅用于开发设置）
services:
  app:
    environment:
      - DEBUG=app:*
      - LOG_LEVEL=debug
    ports:
      - "9229:9229"                   # Node.js debugger

# docker-compose.prod.yml（生产环境，显式指定）
services:
  app:
    build:
      target: production
    restart: always
    deploy:
      resources:
        limits:
          cpus: "1.0"
          memory: 512M
```

```bash
# 开发（自动加载 override）
docker compose up

# 生产
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

## 网络（Networking）

### 服务发现

同一 Compose 网络中的服务通过服务名解析：
```
# 从 "app" 容器：
postgres://postgres:postgres@db:5432/app_dev    # 解析为 "db" db 容器
redis://redis:6379/0                             # 解析为 "redis" redis 容器
```

### 自定义网络

```yaml
services:
  frontend:
    networks:
      - frontend-net

  api:
    networks:
      - frontend-net
      - backend-net

  db:
    networks:
      - backend-net              # 仅从 api 访问，不能从 frontend 访问

networks:
  frontend-net:
  backend-net:
```

### 仅暴露必要端口

```yaml
services:
  db:
    ports:
      - "127.0.0.1:5432:5432"   # 仅从 host 访问，不从网络访问
    # 生产中完全移除端口 -- 仅从 Docker 网络内部访问
```

## Volume 策略

```yaml
volumes:
  # 命名 volume：容器重启后持久化，由 Docker 管理
  pgdata:

  # Bind mount：将 host 目录映射到容器（用于开发）
  # - ./src:/app/src

  # 匿名 volume：从 bind mount override 保留容器创建的内容
  # - /app/node_modules
```

### 常见模式

```yaml
services:
  app:
    volumes:
      - .:/app                   # 源代码（用于热重载的 bind mount）
      - /app/node_modules        # 从 host 保护容器的 node_modules
      - /app/.next               # 保留 build 缓存

  db:
    volumes:
      - pgdata:/var/lib/postgresql/data          # 持久化数据
      - ./scripts/init.sql:/docker-entrypoint-initdb.d/init.sql  # Init 脚本
```

## 容器安全

### Dockerfile 强化

```dockerfile
# 1. 使用特定 tag（绝不要用 :latest）
FROM node:22.12-alpine3.20

# 2. 以非 root 用户运行
RUN addgroup -g 1001 -S app && adduser -S app -u 1001
USER app

# 3. 降低 Capability（在 compose 中）
# 4. 尽可能使用只读根文件系统
# 5. Image layer 中没有 secret
```

### Compose 安全

```yaml
services:
  app:
    security_opt:
      - no-new-privileges:true
    read_only: true
    tmpfs:
      - /tmp
      - /app/.cache
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE          # 仅用于绑定 < 1024 的端口
```

### Secret 管理

```yaml
# 好：使用环境变量（运行时注入）
services:
  app:
    env_file:
      - .env                     # 绝不要将 .env 提交到 git
    environment:
      - API_KEY                  # 从 host 环境继承

# 好：Docker secrets（Swarm 模式）
secrets:
  db_password:
    file: ./secrets/db_password.txt

services:
  db:
    secrets:
      - db_password

# 坏：在 Image 中硬编码
# ENV API_KEY=sk-proj-xxxxx      # 绝不要这样做
```

## .dockerignore

```
node_modules
.git
.env
.env.*
dist
coverage
*.log
.next
.cache
docker-compose*.yml
Dockerfile*
README.md
tests/
```

## 调试

### 常用命令

```bash
# 查看日志
docker compose logs -f app           # 跟踪 app 日志
docker compose logs --tail=50 db     # db 的最后 50 行

# 在运行的容器中执行命令
docker compose exec app sh           # 通过 shell 进入 app
docker compose exec db psql -U postgres  # 连接到 postgres

# 检查
docker compose ps                     # 运行的服务
docker compose top                    # 每个容器中的进程
docker stats                          # 资源使用情况

# 重新构建
docker compose up --build             # 重新构建 image
docker compose build --no-cache app   # 强制完全重建

# 清理
docker compose down                   # 停止并移除容器
docker compose down -v                # 也移除 volume（破坏性）
docker system prune                   # 移除未使用的 image/容器
```

### 调试网络问题

```bash
# 检查容器内的 DNS 解析
docker compose exec app nslookup db

# 检查连接
docker compose exec app wget -qO- http://api:3000/health

# 检查网络
docker network ls
docker network inspect <project>_default
```

## 反模式

```
# 坏：不要在没有编排的生产环境中使用 docker compose
# 对于生产多容器工作负载，使用 Kubernetes、ECS 或 Docker Swarm

# 坏：没有 volume 在容器中存储数据
# 容器是临时的 -- 没有 volume，重启时所有数据都会丢失

# 坏：以 root 身份运行
# 始终创建并使用非 root 用户

# 坏：使用 :latest tag
# 固定到特定版本以获得可重现的构建

# 坏：包含所有服务的单个 dev 容器
# 分离关注点：每个容器一个进程

# 坏：把 secret 放到 docker-compose.yml
# 使用 .env 文件（已 gitignore）或 Docker secrets
```
