# 命令 → Agent / Skill 映射表

本文档列出了每个斜杠命令及其调用的主要 agent 或 skill，以及值得注意的直接调用 agent。使用此文档可以发现哪些命令使用哪些 agent，并保持重构的一致性。

| Command | Primary agent(s) | Notes |
|---------|------------------|--------|
| `/plan` | planner | 编码前的实现规划 |
| `/tdd` | tdd-guide | 测试驱动开发 |
| `/code-review` | code-reviewer | 质量和安全审查 |
| `/build-fix` | build-error-resolver | 修复构建/类型错误 |
| `/e2e` | e2e-runner | Playwright E2E 测试 |
| `/refactor-clean` | refactor-cleaner | 死代码移除 |
| `/update-docs` | doc-updater | 文档同步 |
| `/update-codemaps` | doc-updater | 代码映射 / 架构文档 |
| `/go-review` | go-reviewer | Go 代码审查 |
| `/go-test` | tdd-guide | Go TDD 工作流 |
| `/go-build` | go-build-resolver | 修复 Go 构建错误 |
| `/python-review` | python-reviewer | Python 代码审查 |
| `/harness-audit` | — | Harness 评分卡（无单一 agent） |
| `/loop-start` | loop-operator | 启动自主循环 |
| `/loop-status` | loop-operator | 检查循环状态 |
| `/quality-gate` | — | 质量管道（类似 hook） |
| `/model-route` | — | 模型推荐（无 agent） |
| `/orchestrate` | planner, tdd-guide, code-reviewer, security-reviewer, architect | 多 agent 交接 |
| `/multi-plan` | architect (Codex/Gemini prompts) | 多模型规划 |
| `/multi-execute` | architect / frontend prompts | 多模型执行 |
| `/multi-backend` | architect | 后端多服务 |
| `/multi-frontend` | architect | 前端多服务 |
| `/multi-workflow` | architect | 通用多服务 |
| `/learn` | — | continuous-learning skill, instincts |
| `/learn-eval` | — | continuous-learning-v2, 评估后保存 |
| `/instinct-status` | — | continuous-learning-v2 |
| `/instinct-import` | — | continuous-learning-v2 |
| `/instinct-export` | — | continuous-learning-v2 |
| `/evolve` | — | continuous-learning-v2, 聚类 instincts |
| `/promote` | — | continuous-learning-v2 |
| `/projects` | — | continuous-learning-v2 |
| `/skill-create` | — | skill-create-output 脚本, git 历史 |
| `/checkpoint` | — | verification-loop skill |
| `/verify` | — | verification-loop skill |
| `/eval` | — | eval-harness skill |
| `/test-coverage` | — | 覆盖率分析 |
| `/sessions` | — | 会话历史 |
| `/setup-pm` | — | 包管理器设置脚本 |
| `/claw` | — | NanoClaw CLI (scripts/claw.js) |
| `/pm2` | — | PM2 服务生命周期 |
| `/security-scan` | security-reviewer (skill) | 通过 security-scan skill 运行 AgentShield |

## 直接使用的 Agents

| Direct agent | Purpose | Scope | Notes |
|--------------|---------|-------|-------|
| `typescript-reviewer` | TypeScript/JavaScript 代码审查 | TypeScript/JavaScript 项目 | 当需要针对 TS/JS 的特定审查结果且没有专用斜杠命令时，直接调用此 agent。 |

## 命令引用的 Skills

- **continuous-learning**, **continuous-learning-v2**: `/learn`, `/learn-eval`, `/instinct-*`, `/evolve`, `/promote`, `/projects`
- **verification-loop**: `/checkpoint`, `/verify`
- **eval-harness**: `/eval`
- **security-scan**: `/security-scan` (运行 AgentShield)
- **strategic-compact**: 在压缩点（hooks）建议使用

## 如何使用此映射表

- **可发现性：** 查找哪个命令触发哪个 agent（例如 "使用 `/code-review` 调用 code-reviewer"）。
- **重构：** 重命名或删除 agent 时，搜索此文档和命令文件中的引用。
- **CI/文档：** 目录脚本 (`node scripts/ci/catalog.js`) 输出 agent/command/skill 计数；此映射表补充了命令与 agent 的关系。
