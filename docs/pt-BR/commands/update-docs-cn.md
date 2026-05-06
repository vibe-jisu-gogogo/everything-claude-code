# 更新文档

将文档与代码库同步，从真实的源文件生成。

## 步骤 1：识别真实数据源

| Source | Generates |
|--------|-----------|
| `package.json` scripts | Available commands reference |
| `.env.example` | Environment variable documentation |
| `openapi.yaml` / route files | API endpoint reference |
| Source code exports | Public API documentation |
| `Dockerfile` / `docker-compose.yml` | Infrastructure setup docs |

## 步骤 2：生成脚本参考

1. 读取 `package.json`（或 `Makefile`、`Cargo.toml`、`pyproject.toml`）
2. 提取所有脚本/命令及其描述
3. 生成参考表格：

```markdown
| Command | Description |
|---------|-------------|
| `npm run dev` | Start development server with hot reload |
| `npm run build` | Production build with type checking |
| `npm test` | Run test suite with coverage |
```

## 步骤 3：生成环境文档

1. 读取 `.env.example`（或 `.env.template`、`.env.sample`）
2. 提取所有变量及其用途
3. 分类为 required 与 optional
4. 记录预期格式和有效值

```markdown
| Variable | Required | Description | Example |
|----------|----------|-------------|---------|
| `DATABASE_URL` | Yes | PostgreSQL connection string | `postgres://user:pass@host:5432/db` |
| `LOG_LEVEL` | No | Logging verbosity (default: info) | `debug`, `info`, `warn`, `error` |
```

## 步骤 4：更新贡献指南

生成或更新 `docs/CONTRIBUTING.md`，包含：
- 开发环境设置（先决条件、安装步骤）
- 可用脚本及其用途
- 测试流程（如何运行、如何编写新测试）
- 代码风格强制执行（linter、formatter、pre-commit hooks）
- PR 提交检查清单

## 步骤 5：更新 Runbook

生成或更新 `docs/RUNBOOK.md`，包含：
- 部署流程（分步说明）
- health check 与监控 endpoints
- 常见问题及其修复方法
- 回滚流程
- 告警与升级路径

## 步骤 6：过时内容检查

1. 查找 90+ 天未修改的文档文件
2. 与源代码中的近期变更进行交叉核对
3. 标记可能过时的文档供人工审核

## 步骤 7：显示摘要

```
Documentation Update
──────────────────────────────
Updated:  docs/CONTRIBUTING.md (scripts table)
Updated:  docs/ENV.md (3 new variables)
Flagged:  docs/DEPLOY.md (142 days stale)
Skipped:  docs/API.md (no changes detected)
──────────────────────────────
```

## 规则

- **唯一真实来源**：始终从代码生成，绝不手动编辑生成的章节
- **保留手动章节**：仅更新生成的章节；保持手动编写的内容完整
- **标记生成内容**：在生成的章节周围使用 `<!-- AUTO-GENERATED -->` 标记
- **不擅自创建文档**：仅在命令明确要求时创建新的文档文件
