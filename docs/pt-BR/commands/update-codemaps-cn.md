# 更新 Codemaps

分析代码库结构并生成精简的架构文档 tokens。

## 步骤 1：扫描项目结构

1. 识别项目类型 (monorepo, single app, library, microservice)
2. 查找所有源代码目录 (src/, lib/, app/, packages/)
3. 映射 entry points (main.ts, index.ts, app.py, main.go, etc.)

## 步骤 2：生成 Codemaps

创建或更新 `docs/CODEMAPS/` (或 `.reports/codemaps/`) 中的 codemaps：

| File | Contents |
|------|----------|
| `architecture.md` | High-level system diagram, service boundaries, data flow |
| `backend.md` | API routes, middleware chain, service → repository mapping |
| `frontend.md` | Page tree, component hierarchy, state management flow |
| `data.md` | Database tables, relationships, migration history |
| `dependencies.md` | External services, third-party integrations, shared libraries |

### Codemap 格式

每个 codemap 应该是 token 精简的 — 优化用于 AI 上下文消费：

```markdown
# Backend Architecture

## Routes
POST /api/users → UserController.create → UserService.create → UserRepo.insert
GET  /api/users/:id → UserController.get → UserService.findById → UserRepo.findById

## Key Files
src/services/user.ts (business logic, 120 lines)
src/repos/user.ts (database access, 80 lines)

## Dependencies
- PostgreSQL (primary data store)
- Redis (session cache, rate limiting)
- Stripe (payment processing)
```

## 步骤 3：Diff 检测

1. 如果存在以前的 codemaps，计算 diff 百分比
2. 如果变更 > 30%，显示 diff 并在覆盖前请求用户批准
3. 如果变更 <= 30%，直接更新

## 步骤 4：添加元数据

在每个 codemap 中添加 freshness 头部：

```markdown
<!-- Generated: 2026-02-11 | Files scanned: 142 | Token estimate: ~800 -->
```

## 步骤 5：保存分析报告

在 `.reports/codemap-diff.txt` 中写入摘要：
- 自上次扫描以来添加/删除/修改的文件
- 检测到的新 dependencies
- 架构变更 (新 routes, 新 services 等)
- 对 90+ 天未更新的文档的过时警告

## 提示

- 专注于**高层结构**，而不是实现细节
- 优先使用**文件路径和函数签名**而不是完整的代码块
- 保持每个 codemap 在 **1000 tokens** 以下以实现高效的上下文加载
- 使用 ASCII diagrams 表示数据流，而不是冗长的描述
- 在大型 feature 添加或 refactoring 会话后运行
