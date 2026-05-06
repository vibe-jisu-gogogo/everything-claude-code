---
name: update-docs
description: 基于代码库同步文档并更新生成的章节。
---

# 文档更新

将文档与代码库同步，并从源文件生成文档。

## 第1步：识别源

| 源 | 生成目标 |
|------|----------|
| `package.json` scripts | 可用命令参考 |
| `.env.example` | 环境变量文档 |
| `openapi.yaml` / 路由文件 | API 端点参考 |
| 源代码 exports | 公开 API 文档 |
| `Dockerfile` / `docker-compose.yml` | 基础设施配置文档 |

## 第2步：生成脚本参考

1. 读取 `package.json`（或 `Makefile`、`Cargo.toml`、`pyproject.toml`）
2. 提取所有脚本/命令和说明
3. 生成参考表格：

```markdown
| 命令 | 说明 |
|--------|------|
| `npm run dev` | 启动带有 hot reload 的开发服务器 |
| `npm run build` | 包含类型检查的生产构建 |
| `npm test` | 运行包含覆盖率的测试套件 |
```

## 第3步：生成环境变量文档

1. 读取 `.env.example`（或 `.env.template`、`.env.sample`）
2. 提取所有变量和用途
3. 按必需与可选分类
4. 记录预期格式和有效值

```markdown
| 变量 | 必需 | 说明 | 示例 |
|------|------|------|------|
| `DATABASE_URL` | 是 | PostgreSQL 连接字符串 | `postgres://user:pass@host:5432/db` |
| `LOG_LEVEL` | 否 | 日志详细程度（默认：info） | `debug`, `info`, `warn`, `error` |
```

## 第4步：更新贡献指南

创建或更新 `docs/CONTRIBUTING.md`：
- 开发环境设置（先决条件、安装步骤）
- 可用脚本及其用途
- 测试程序（运行方法、编写新测试的方法）
- 代码样式执行（linter、formatter、pre-commit hook）
- PR 提交清单

## 第5步：更新操作手册

创建或更新 `docs/RUNBOOK.md`：
- 部署程序（分步）
- 健康检查端点和监控
- 常见问题及解决方案
- 回滚程序
- 通知和升级路径

## 第6步：检查过时项目

1. 查找超过 90 天未修改的文档文件
2. 与最近的源代码更改交叉参考
3. 将可能过时的文档标记为手动审查目标

## 第7步：显示摘要

```
文档更新
──────────────────────────────
更新: docs/CONTRIBUTING.md (脚本表格)
更新: docs/ENV.md (3 个新变量)
标记:   docs/DEPLOY.md (142 天)
跳过:   docs/API.md (无更改)
──────────────────────────────
```

## 规则

- **单一来源**: 始终从代码生成，不要手动编辑生成的章节
- **保留手动章节**: 仅更新生成的章节；手写内容保持不变
- **标记生成的内容**: 在生成的章节周围使用 `<!-- AUTO-GENERATED -->` 标记
- **不生成未请求的文档**: 仅在命令明确请求时创建新的文档文件
