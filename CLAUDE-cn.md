# CLAUDE.md

本文件为 Claude Code (claude.ai/code) 提供在本仓库中处理代码时的指导。

## 项目概述

这是一个 **Claude Code 插件** - 包含生产就绪的 agents、skills、hooks、commands、rules 和 MCP 配置集合。本项目为使用 Claude Code 进行软件开发提供经过实战验证的工作流。

## 运行测试

```bash
# 运行所有测试
node tests/run-all.js

# 运行单个测试文件
node tests/lib/utils.test.js
node tests/lib/package-manager.test.js
node tests/hooks/hooks.test.js
```

## 架构

项目分为几个核心组件：

- **agents/** - 用于任务委派的专用子代理（planner、code-reviewer、tdd-guide 等）
- **skills/** - 工作流定义和领域知识（编码标准、模式、测试）
- **commands/** - 用户调用的斜杠命令（/tdd、/plan、/e2e 等）
- **hooks/** - 基于触发器的自动化功能（会话持久化、工具调用前后钩子）
- **rules/** - 必须遵守的指导方针（安全、编码风格、测试要求）
- **mcp-configs/** - 用于外部集成的 MCP 服务器配置
- **scripts/** - 用于 hooks 和设置的跨平台 Node.js 工具
- **tests/** - 脚本和工具的测试套件

## 核心命令

- `/tdd` - 测试驱动开发工作流
- `/plan` - 实现规划
- `/e2e` - 生成并运行 E2E 测试
- `/code-review` - 质量审查
- `/build-fix` - 修复构建错误
- `/learn` - 从会话中提取模式
- `/skill-create` - 从 git 历史生成 skills

## 开发注意事项

- 包管理器检测：npm、pnpm、yarn、bun（可通过 `CLAUDE_PACKAGE_MANAGER` 环境变量或项目配置进行配置）
- 跨平台：通过 Node.js 脚本支持 Windows、macOS、Linux
- Agent 格式：带有 YAML 头部的 Markdown（name、description、tools、model）
- Skill 格式：带有清晰章节的 Markdown，包含适用场景、工作原理、示例
- Skill 存放位置：精心策划的 skills 放在 skills/ 目录下；生成/导入的 skills 放在 ~/.claude/skills/ 目录下。参见 docs/SKILL-PLACEMENT-POLICY.md
- Hook 格式：带有匹配条件和命令/通知钩子的 JSON

## 贡献指南

遵循 CONTRIBUTING.md 中的格式要求：
- Agents：带有头部的 Markdown（name、description、tools、model）
- Skills：清晰的章节结构（适用场景、工作原理、示例）
- Commands：Markdown 文件必须包含 description 头部字段
- Hooks：带有 matcher 和 hooks 数组的 JSON

文件命名：小写字母加连字符（例如：`python-reviewer.md`、`tdd-workflow.md`）

## Skills

处理相关文件时使用以下 skills：

| 文件 | Skill |
|---------|-------|
| `README.md` | `/readme` |
| `.github/workflows/*.yml` | `/ci-workflow` |

创建子代理时，始终将对应 skill 的约定传递到代理的 prompt 中。
