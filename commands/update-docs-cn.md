---
description: 从 scripts、schemas、routes 和 exports 等可信源文件同步文档。
---

# 更新文档

与代码库同步文档，从可信源文件生成。

## 步骤 1：识别可信源

| 来源 | 生成 |
|--------|-----------|
| `package.json` scripts | 可用命令参考 |
| `.env.example` | 环境变量文档 |
| `openapi.yaml` / route 文件 | API 端点参考 |
| 源代码 exports | 公共 API 文档 |
| `Dockerfile` / `docker-compose.yml` | 基础设施设置文档 |

## 步骤 2：生成脚本参考

1. 读取 `package.json`（或 `Makefile`、`Cargo.toml`、`pyproject.toml`）
2. 提取所有 scripts/commands 及其描述
3. 生成参考表格：

```markdown
| 命令 | 描述 |
|---------|-------------|
| `npm run dev` | 启动带热重载的开发服务器 |
| `npm run build` | 带类型检查的生产构建 |
| `npm test` | 运行带覆盖率的测试套件 |
```

## 步骤 3：生成环境文档

1. 读取 `.env.example`（或 `.env.template`、`.env.sample`）
2. 提取所有变量及其用途
3. 分类为必填与可选
4. 记录预期格式和有效值

```markdown
| 变量 | 必填 | 描述 | 示例 |
|----------|----------|-------------|---------|
| `DATABASE_URL` | 是 | PostgreSQL 连接字符串 | `postgres://user:pass@host:5432/db` |
| `LOG_LEVEL` | 否 | 日志详细程度（默认：info） | `debug`、`info`、`warn`、`error` |
```

## 步骤 4：更新贡献指南

生成或更新 `docs/CONTRIBUTING.md`，包含：
- 开发环境设置（前置条件、安装步骤）
- 可用脚本及其用途
- 测试流程（如何运行、如何编写新测试）
- 代码风格强制执行（linter、formatter、pre-commit hooks）
- PR 提交检查清单

## 步骤 5：更新运行手册

生成或更新 `docs/RUNBOOK.md`，包含：
- 部署流程（分步说明）
- 健康检查端点和监控
- 常见问题及其修复方法
- 回滚流程
- 告警和升级路径

## 步骤 6：陈旧性检查

1. 查找 90+ 天未修改的文档文件
2. 与最近的源代码更改进行交叉引用
3. 标记可能过时的文档以供人工审核

## 步骤 7：显示摘要

```
文档更新
──────────────────────────────
已更新： docs/CONTRIBUTING.md（脚本表格）
已更新： docs/ENV.md（3 个新变量）
已标记： docs/DEPLOY.md（142 天未更新）
已跳过： docs/API.md（未检测到更改）
──────────────────────────────
```

## 规则

- **单一可信源**：始终从代码生成，切勿手动编辑生成的部分
- **保留手动部分**：仅更新生成的部分；保持手写内容完整
- **标记生成内容**：在生成部分周围使用 `<!-- AUTO-GENERATED -->` 标记
- **不主动创建文档**：仅在命令明确要求时创建新的文档文件
