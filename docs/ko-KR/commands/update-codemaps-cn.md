# 代码映射更新

分析代码库结构并生成令牌高效的架构文档。

## 第1步：项目结构扫描

1. 识别项目类型（monorepo、单一应用、库、微服务）
2. 查找所有源目录（src/、lib/、app/、packages/）
3. 入口点映射（main.ts、index.ts、app.py、main.go 等）

## 第2步：代码映射生成

在 `docs/CODEMAPS/` 中创建或更新代码映射：

| 文件 | 内容 |
|------|------|
| `INDEX.md` | 整个代码库概述和各区域链接 |
| `backend.md` | API 路由、中间件链、服务 → 仓库映射 |
| `frontend.md` | 页面树、组件层级、状态管理流程 |
| `database.md` | 数据库 schema、迁移、存储层 |
| `integrations.md` | 外部服务、第三方集成、适配器 |
| `workers.md` | 后台任务、队列、调度器 |

### 代码映射格式

每个代码映射必须是令牌高效的 — 优化 AI 上下文消耗：

```markdown
# Backend 架构

## 路由
POST /api/users → UserController.create → UserService.create → UserRepo.insert
GET  /api/users/:id → UserController.get → UserService.findById → UserRepo.findById

## 主要文件
src/services/user.ts（业务逻辑，120 行）
src/repos/user.ts（数据库访问，80 行）

## 依赖
- PostgreSQL（主数据存储）
- Redis（会话缓存、速率限制）
- Stripe（支付处理）
```

## 第3步：区域分类

生成器根据文件路径模式自动分类区域：

1. 前端：`app/`、`pages/`、`components/`、`hooks/`、`.tsx`、`.jsx`
2. 后端：`api/`、`routes/`、`controllers/`、`services/`、`.route.ts`
3. 数据库：`db/`、`migrations/`、`prisma/`、`repositories/`
4. 集成：`integrations/`、`adapters/`、`connectors/`、`plugins/`
5. 工作器：`workers/`、`jobs/`、`queues/`、`tasks/`、`cron/`

## 第4步：添加元数据

在每个代码映射中添加最新信息头：

```markdown
**Last Updated:** 2026-03-12
**Total Files:** 42
**Total Lines:** 1875
```

## 第5步：索引与区域文档同步

`INDEX.md` 必须链接并总结生成的区域文档：
- 每个区域的文件数和总行数
- 检测到的入口点
- 存储库树的简单 ASCII 概述
- 各区域详细文档链接

## 提示

- 专注于**高层结构**而非实现细节
- 使用**文件路径和函数签名**而非完整代码块
- 为高效上下文加载，将每个代码映射保持在 **1000 令牌以下**
- 使用 ASCII 图表展示数据流而非冗长说明
- 在主要功能添加或重构会话后运行 `npx tsx scripts/codemaps/generate.ts`
