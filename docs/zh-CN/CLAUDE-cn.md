# CLAUDE.md

本文件为 Claude Code (claude.ai/code) 处理此仓库代码时提供指导。

## 项目概述

这是一个 **Claude Code 插件** - 一个包含生产就绪的 agents、skills、hooks、commands、rules 和 MCP 配置的集合。该项目提供了使用 Claude Code 进行软件开发的经过实战验证的工作流。

## 运行测试

```bash
# Run all tests
node tests/run-all.js

# Run individual test files
node tests/lib/utils.test.js
node tests/lib/package-manager.test.js
node tests/hooks/hooks.test.js
```

## 架构

项目组织为以下几个核心组件：

- **agents/** - 用于委派的专业化子代理（planner、code-reviewer、tdd-guide 等）
- **skills/** - 工作流定义和领域知识（编码标准、模式、测试）
- **commands/** - 由用户调用的斜杠命令（/tdd, /plan, /e2e 等）
- **hooks/** - 基于触发器的自动化（会话持久化、工具前后钩子）
- **rules/** - 始终遵循的指南（安全、编码风格、测试要求）
- **mcp-configs/** - 用于外部集成的 MCP 服务器配置
- **scripts/** - 用于钩子和设置的跨平台 Node.js 工具
- **tests/** - 脚本和工具的测试套件

## 关键命令

- `/tdd` - 测试驱动开发工作流
- `/plan` - 实施规划
- `/e2e` - 生成并运行 E2E 测试
- `/code-review` - 质量审查
- `/build-fix` - 修复构建错误
- `/learn` - 从会话中提取模式
- `/skill-create` - 从 git 历史记录生成技能

## 开发说明

- 包管理器检测：npm、pnpm、yarn、bun（可通过 `CLAUDE_PACKAGE_MANAGER` env var 或项目配置设置）
- 跨平台：通过 Node.js scripts 支持 Windows、macOS、Linux
- Agent 格式：带有 YAML frontmatter 的 Markdown（name、description、tools、model）
- Skill 格式：带有清晰章节的 Markdown（何时使用、如何工作、示例）
- Skill 放置：精选于 skills/；生成/导入于 ~/.claude/skills/。详见 docs/SKILL-PLACEMENT-POLICY.md
- Hook 格式：带有 matcher 条件和 command/notification hooks 的 JSON

## 贡献

遵循 CONTRIBUTING.md 中的格式：

- Agents：带有 frontmatter 的 Markdown（name、description、tools、model）
- Skills：清晰的章节（何时使用、如何工作、示例）
- Commands：带有 description frontmatter 的 Markdown
- Hooks：带有 matcher 和 hooks 数组的 JSON

文件命名：小写字母并用连字符连接（例如 `python-reviewer.md`、`tdd-workflow.md`）

## Skills

处理相关文件时使用以下 skills：

| File(s) | Skill |
|---------|-------|
| `README.md` | `/readme` |
| `.github/workflows/*.yml` | `/ci-workflow` |

在生成 subagents 时，始终将相应 skill 中的约定传递到 agent 的 prompt 中。
