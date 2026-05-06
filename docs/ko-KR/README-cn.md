**语言:** [English](../../README.md) | [Português (Brasil)](../pt-BR/README.md) | [简体中文](../../README.zh-CN.md) | [繁體中文](../zh-TW/README.md) | [日本語](../ja-JP/README.md) | [한국어](README.md) | [Türkçe](../tr/README.md)

# Everything Claude Code

[![Stars](https://img.shields.io/github/stars/affaan-m/everything-claude-code?style=flat)](https://github.com/affaan-m/everything-claude-code/stargazers)
[![Forks](https://img.shields.io/github/forks/affaan-m/everything-claude-code?style=flat)](https://github.com/affaan-m/everything-claude-code/network/members)
[![Contributors](https://img.shields.io/github/contributors/affaan-m/everything-claude-code?style=flat)](https://github.com/affaan-m/everything-claude-code/graphs/contributors)
[![npm ecc-universal](https://img.shields.io/npm/dw/ecc-universal?label=ecc-universal%20weekly%20downloads&logo=npm)](https://www.npmjs.com/package/ecc-universal)
[![npm ecc-agentshield](https://img.shields.io/npm/dw/ecc-agentshield?label=ecc-agentshield%20weekly%20downloads&logo=npm)](https://www.npmjs.com/package/ecc-agentshield)
[![GitHub App Install](https://img.shields.io/badge/GitHub%20App-150%20installs-2ea44f?logo=github)](https://github.com/marketplace/ecc-tools)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](../../LICENSE)
![Shell](https://img.shields.io/badge/-Shell-4EAA25?logo=gnu-bash&logoColor=white)
![TypeScript](https://img.shields.io/badge/-TypeScript-3178C6?logo=typescript&logoColor=white)
![Python](https://img.shields.io/badge/-Python-3776AB?logo=python&logoColor=white)
![Go](https://img.shields.io/badge/-Go-00ADD8?logo=go&logoColor=white)
![Java](https://img.shields.io/badge/-Java-ED8B00?logo=openjdk&logoColor=white)
![Markdown](https://img.shields.io/badge/-Markdown-000000?logo=markdown&logoColor=white)

> **140K+ stars** | **21K+ forks** | **170+ contributors** | **12+ language ecosystems** | **Anthropic 黑客松冠军**

---

<div align="center">

**Language / 语言 / 語言 / 언어 / Dil**

[**English**](../../README.md) | [Português (Brasil)](../pt-BR/README.md) | [简体中文](../../README.zh-CN.md) | [繁體中文](../zh-TW/README.md) | [日本語](../ja-JP/README.md) | [한국어](README.md) | [Türkçe](../tr/README.md)

</div>

---

**AI Agent 驱动的性能优化系统。由 Anthropic 黑客松冠军创建。**

这不仅仅是配置文件的集合。这是一个完整的系统，涵盖技能、直觉(Instinct)、内存优化、持续学习、安全扫描和研究优先开发。包含经过10个月以上在实际产品开发中日常密集使用而不断完善的生产级 Agent、Hook、命令、规则和 MCP 配置。

可在 **Claude Code**、**Codex**、**Cursor**、**OpenCode**、**Gemini** 等各种 AI Agent 驱动中使用。

---

## 指南

本仓库仅包含代码。完整内容在指南中说明。

<table>
<tr>
<td width="50%">
<a href="https://x.com/affaanmustafa/status/2012378465664745795">
<img src="https://github.com/user-attachments/assets/1a471488-59cc-425b-8345-5245c7efbcef" alt="The Shorthand Guide to Everything Claude Code" />
</a>
</td>
<td width="50%">
<a href="https://x.com/affaanmustafa/status/2014040193557471352">
<img src="https://github.com/user-attachments/assets/c9ca43bc-b149-427f-b551-af6840c368f0" alt="The Longform Guide to Everything Claude Code" />
</a>
</td>
</tr>
<tr>
<td align="center"><b>简明指南</b><br/>设置、基础、理念。<b>请从这里开始。</b></td>
<td align="center"><b>详细指南</b><br/>Token 优化、内存持久化、评估、并行处理。</td>
</tr>
</table>

| 主题 | 可学习内容 |
|------|----------|
| Token 优化 | 模型选择、系统提示优化、后台进程 |
| 内存持久化 | 自动在会话间保存/加载上下文的 Hook |
| 持续学习 | 从会话中自动提取模式并转换为可复用的技能 |
| 验证循环 | 检查点 vs 连续评估、评分类型、pass@k 指标 |
| 并行处理 | Git worktree、级联方式、实例扩展时机 |
| 子 Agent 编排 | 上下文问题、重复搜索模式 |

---

## 最新消息

### v1.8.0 — 驱动性能系统 (2026年3月)

- **以驱动为中心的版本** — ECC 现在明确成为 Agent 驱动性能系统，而不仅仅是配置集合。
- **Hook 稳定性改进** — SessionStart 根回退、Stop 阶段会话摘要、将脆弱的内联单行代码替换为基于脚本的 Hook。
- **Hook 运行时控制** — 通过 `ECC_HOOK_PROFILE=minimal|standard|strict` 和 `ECC_DISABLED_HOOKS=...` 在运行时控制，无需修改 Hook 文件。
- **新驱动命令** — `/harness-audit`、`/loop-start`、`/loop-status`、`/quality-gate`、`/model-route`。
- **NanoClaw v2** — 模型路由、技能热加载、会话分支/搜索/导出/压缩/指标。
- **跨驱动兼容性** — 强化 Claude Code、Cursor、OpenCode、Codex 间的行为一致性。
- **通过 997 个内部测试** — Hook/运行时重构和兼容性更新后全部测试通过。

### v1.7.0 — 跨平台扩展 & 演示文稿生成器 (2026年2月)

- **Codex 应用 + CLI 支持** — 基于 AGENTS.md 的直接 Codex 支持
- **`frontend-slides` 技能** — 无依赖的 HTML 演示文稿生成器
- **5 个新的业务/内容技能** — `article-writing`、`content-engine`、`market-research`、`investor-materials`、`investor-outreach`
- **992 个内部测试** — 扩展的验证和回归测试覆盖

### v1.6.0 — Codex CLI、AgentShield & 市场 (2026年2月)

- **Codex CLI 支持** — 用于 OpenAI Codex CLI 兼容性的 `/codex-setup` 命令
- **7 个新技能** — `search-first`、`swift-actor-persistence`、`swift-protocol-di-testing` 等
- **AgentShield 集成** — 通过 `/security-scan` 直接在 Claude Code 中运行 AgentShield；1282 个测试、102 个规则
- **GitHub 市场** — 在 [github.com/marketplace/ecc-tools](https://github.com/marketplace/ecc-tools) 提供免费/专业/企业版
- **30+ 社区贡献** — 6 种语言的 30 名贡献者
- **978 个内部测试** — 跨 Agent、技能、命令、Hook、规则的全面验证

完整变更记录请查看 [Releases](https://github.com/affaan-m/everything-claude-code/releases)。

---

## 快速开始

2分钟内完成设置：

### 第1步：安装插件

```bash
# 添加市场
/plugin marketplace add https://github.com/affaan-m/everything-claude-code

# 安装插件
/plugin install everything-claude-code
```

### 第2步：安装规则 (必需)

> WARNING: **重要：** Claude Code 插件无法自动分发 `rules`。必须手动安装：

```bash
# 首先克隆仓库
git clone https://github.com/affaan-m/everything-claude-code.git
cd everything-claude-code

# 推荐：使用安装脚本 (安全处理 common + 语言特定规则)
./install.sh typescript    # 或 python、golang
# 可以一次性安装多种语言：
# ./install.sh typescript python golang
# 为 Cursor 安装：
# ./install.sh --target cursor typescript
```

手动安装方法请参考 `rules/` 文件夹的 README。

### 第3步：开始使用

```bash
# 运行命令 (安装插件时使用命名空间形式)
/everything-claude-code:plan "添加用户认证"

# 手动安装(选项2)时使用简短形式：
# /plan "添加用户认证"

# 查看可用命令
/plugin list everything-claude-code@everything-claude-code
```

**完成！** 现在可以使用 16 个 Agent、65 个技能、40 个命令。

---

## 跨平台支持

本插件完全支持 **Windows、macOS、Linux**，并与主要 IDE (Cursor、OpenCode、Antigravity) 及 CLI 驱动紧密集成。所有 Hook 和脚本均为最大兼容性而使用 Node.js 编写。

### 包管理器检测

插件会自动检测首选的包管理器 (npm、pnpm、yarn、bun)：

1. **环境变量**：`CLAUDE_PACKAGE_MANAGER`
2. **项目设置**：`.claude/package-manager.json`
3. **package.json**：`packageManager` 字段
4. **锁文件**：从 package-lock.json、yarn.lock、pnpm-lock.yaml、bun.lockb 检测
5. **全局设置**：`~/.claude/package-manager.json`
6. **回退**：`npm`

包管理器设置方法：

```bash
# 通过环境变量设置
export CLAUDE_PACKAGE_MANAGER=pnpm

# 全局设置
node scripts/setup-package-manager.js --global pnpm

# 项目设置
node scripts/setup-package-manager.js --project bun

# 查看当前设置
node scripts/setup-package-manager.js --detect
```

或在 Claude Code 中使用 `/setup-pm` 命令。

### Hook 运行时控制

可以通过运行时标志调整严格程度或临时禁用特定 Hook：

```bash
# Hook 严格程度配置 (默认：standard)
export ECC_HOOK_PROFILE=standard

# 要禁用的 Hook ID (逗号分隔)
export ECC_DISABLED_HOOKS="pre:bash:tmux-reminder,post:edit:typecheck"
```

---

## 组件

本仓库是一个 **Claude Code 插件** — 可以直接安装或手动复制组件。

```
everything-claude-code/
|-- .claude-plugin/   # 插件和市场清单
|   |-- plugin.json         # 插件元数据和组件路径
|   |-- marketplace.json    # /plugin marketplace add 用的市场目录
|
|-- agents/           # 用于委托的专用子 Agent
|   |-- planner.md           # 功能实现计划
|   |-- architect.md         # 系统设计决策
|   |-- tdd-guide.md         # 测试驱动开发
|   |-- code-reviewer.md     # 质量和安全审查
|   |-- security-reviewer.md # 漏洞分析
|   |-- build-error-resolver.md
|   |-- e2e-runner.md        # Playwright E2E 测试
|   |-- refactor-cleaner.md  # 清理未使用的代码
|   |-- doc-updater.md       # 文档同步
|   |-- go-reviewer.md       # Go 代码审查
|   |-- go-build-resolver.md # Go 构建错误解决
|   |-- python-reviewer.md   # Python 代码审查
|   |-- database-reviewer.md # 数据库/Supabase 审查
|
|-- skills/           # 工作流定义和领域知识
|   |-- coding-standards/           # 语言最佳实践
|   |-- backend-patterns/           # API、数据库、缓存模式
|   |-- frontend-patterns/          # React、Next.js 模式
|   |-- continuous-learning/        # 从会话自动提取模式
|   |-- continuous-learning-v2/     # 带置信度分数的直觉学习
|   |-- tdd-workflow/               # TDD 方法论
|   |-- security-review/            # 安全检查清单
|   |-- 更多...
|
|-- commands/         # 用于快速执行的斜杠命令
|   |-- tdd.md              # /tdd - 测试驱动开发
|   |-- plan.md             # /plan - 实现计划
|   |-- e2e.md              # /e2e - E2E 测试生成
|   |-- code-review.md      # /code-review - 质量审查
|   |-- build-fix.md        # /build-fix - 构建错误修复
|   |-- 更多...
|
|-- rules/            # 始终遵循的指南 (复制到 ~/.claude/rules/)
|   |-- common/              # 语言无关原则
|   |-- typescript/          # TypeScript/JavaScript 专用
|   |-- python/              # Python 专用
|   |-- golang/              # Go 专用
|
|-- hooks/            # 基于触发器的自动化
|   |-- hooks.json                # 所有 Hook 设置
|   |-- memory-persistence/       # 会话生命周期 Hook
|
|-- scripts/          # 跨平台 Node.js 脚本
|-- tests/            # 测试套件
|-- contexts/         # 动态系统提示注入上下文
|-- examples/         # 示例设置和会话
|-- mcp-configs/      # MCP 服务器设置
```

---

## 生态系统工具

### Skill Creator

从仓库创建 Claude Code 技能的两种方法：

#### 选项 A：本地分析 (内置)

如需在本地分析而不使用外部服务，请使用 `/skill-create` 命令：

```bash
/skill-create                    # 分析当前仓库
/skill-create --instincts        # 同时生成直觉(instincts)
```

在本地分析 git 历史以生成 SKILL.md 文件。

#### 选项 B：GitHub 应用 (高级)

如需高级功能 (10k+ 提交、自动 PR、团队共享)：

[安装 GitHub 应用](https://github.com/apps/skill-creator) | [ecc.tools](https://ecc.tools)

### AgentShield — 安全审计工具

> 在 Claude Code 黑客松 (Cerebral Valley x Anthropic，2026年2月) 中开发。1282 个测试、98% 覆盖率、102 个静态分析规则。

扫描 Claude Code 设置中的漏洞、错误配置和注入风险。

```bash
# 快速扫描 (无需安装)
npx ecc-agentshield scan

# 自动安全修复
npx ecc-agentshield scan --fix

# 用 3 个 Opus 4.6 Agent 进行精密分析
npx ecc-agentshield scan --opus --stream

# 从头生成安全设置
npx ecc-agentshield init
```

**扫描目标：** CLAUDE.md、settings.json、MCP 设置、Hook、Agent 定义、技能 — 涵盖密钥检测 (14 种模式)、权限审计、Hook 注入分析、MCP 服务器风险分析、Agent 设置审查 5 个类别。

**`--opus` 标志** 运行 3 个 Claude Opus 4.6 Agent 组成的红队/蓝队/审计员管道。攻击者寻找利用链，防御者评估保护措施，审计员综合双方结果编写优先级风险评估。

在 Claude Code 中使用 `/security-scan`，或通过 [GitHub Action](https://github.com/affaan-m/agentshield) 添加到 CI。

[GitHub](https://github.com/affaan-m/agentshield) | [npm](https://www.npmjs.com/package/ecc-agentshield)

### 持续学习 v2

基于直觉 (Instinct) 的学习系统会自动学习您的模式：

```bash
/instinct-status        # 查看学习到的直觉和置信度
/instinct-import <file> # 导入他人的直觉
/instinct-export        # 导出我的直觉
/evolve                 # 将相关直觉聚类为技能
```

详情请参考 `skills/continuous-learning-v2/`。

---

## 要求

### Claude Code CLI 版本

**最低版本：v2.1.0 或更高**

由于 Hook 系统变更，本插件需要 Claude Code CLI v2.1.0 或更高版本。

版本检查：
```bash
claude --version
```

### 重要：Hook 自动加载行为

> WARNING: **贡献者须知：** 请勿在 `.claude-plugin/plugin.json` 中添加 `"hooks"` 字段。回归测试会强制执行此要求。

Claude Code v2.1+ **自动加载** 已安装插件的 `hooks/hooks.json`。显式声明会导致重复检测错误。

---

## 安装

### 选项 1：作为插件安装 (推荐)

```bash
# 添加市场
/plugin marketplace add https://github.com/affaan-m/everything-claude-code

# 安装插件
/plugin install everything-claude-code
```

或直接添加到 `~/.claude/settings.json`：

```json
{
  "extraKnownMarketplaces": {
    "ecc": {
      "source": {
        "source": "github",
        "repo": "affaan-m/everything-claude-code"
      }
    }
  },
  "enabledPlugins": {
    "everything-claude-code@everything-claude-code": true
  }
}
```

> **注意：** Claude Code 插件系统不支持将 `rules` 作为插件分发。规则必须手动安装：
>
> ```bash
> git clone https://github.com/affaan-m/everything-claude-code.git
>
> # 选项 A：用户级别规则 (应用于所有项目)
> mkdir -p ~/.claude/rules
> cp -r everything-claude-code/rules/common/* ~/.claude/rules/
> cp -r everything-claude-code/rules/typescript/* ~/.claude/rules/   # 选择您使用的技术栈
>
> # 选项 B：项目级别规则 (仅应用于当前项目)
> mkdir -p .claude/rules
> cp -r everything-claude-code/rules/common/* .claude/rules/
> ```

---

### 选项 2：手动安装

如需手动选择要安装的项目：

```bash
# 克隆仓库
git clone https://github.com/affaan-m/everything-claude-code.git

# 复制 Agent
cp everything-claude-code/agents/*.md ~/.claude/agents/

# 复制规则 (common + 语言特定)
cp -r everything-claude-code/rules/common/* ~/.claude/rules/
cp -r everything-claude-code/rules/typescript/* ~/.claude/rules/   # 选择您使用的技术栈

# 复制命令
cp everything-claude-code/commands/*.md ~/.claude/commands/

# 复制技能
cp -r everything-claude-code/skills/* ~/.claude/skills/
cp -r everything-claude-code/skills/search-first ~/.claude/skills/
```

---

## 核心概念

### Agent

子 Agent 在有限范围内处理委托任务。示例：

```markdown
---
name: code-reviewer
description: 审查代码的质量、安全性、可维护性
tools: ["Read", "Grep", "Glob", "Bash"]
model: opus
---

你是一名高级代码审查员...
```

### 技能

技能是由命令或 Agent 调用的工作流定义：

```markdown
# TDD 工作流

1. 首先定义接口
2. 编写失败的测试 (RED)
3. 实现最少代码 (GREEN)
4. 重构 (IMPROVE)
5. 确认 80% 以上覆盖率
```

### Hook

Hook 响应工具事件执行。示例 - console.log 警告：

```json
{
  "matcher": "tool == \"Edit\" && tool_input.file_path matches \"\\\\.(ts|tsx|js|jsx)$\"",
  "hooks": [{
    "type": "command",
    "command": "#!/bin/bash\ngrep -n 'console\\.log' \"$file_path\" && echo '[Hook] 请移除 console.log' >&2"
  }]
}
```

### 规则

规则是必须始终遵循的指南，由 `common/` (语言无关) + 语言特定目录组成：

```
rules/
  common/          # 通用原则 (始终安装)
  typescript/      # TS/JS 专用模式和工具
  python/          # Python 专用模式和工具
  golang/          # Go 专用模式和工具
```

详情请参考 [`rules/README.md`](../../rules/README.md)。

---

## 我应该使用哪个 Agent？

如果不知道从哪里开始，请参考此表：

| 想做的事 | 使用的命令 | 使用的 Agent |
|---------|-----------|-------------|
| 规划新功能 | `/everything-claude-code:plan "添加认证"` | planner |
| 系统架构设计 | `/everything-claude-code:plan` + architect Agent | architect |
| 先写测试再编码 | `/tdd` | tdd-guide |
| 审查刚写的代码 | `/code-review` | code-reviewer |
| 修复构建失败 | `/build-fix` | build-error-resolver |
| 运行 E2E 测试 | `/e2e` | e2e-runner |
| 查找安全漏洞 | `/security-scan` | security-reviewer |
| 移除未使用的代码 | `/refactor-clean` | refactor-cleaner |
| 更新文档 | `/update-docs` | doc-updater |
| 修复 Go 构建失败 | `/go-build` | go-build-resolver |
| Go 代码审查 | `/go-review` | go-reviewer |
| 数据库模式/查询审查 | `/code-review` + database-reviewer Agent | database-reviewer |
| Python 代码审查 | `/python-review` | python-reviewer |

### 一般工作流

**开始新功能：**
```
/everything-claude-code:plan "添加使用 OAuth 的用户认证"
                                              → planner 编写实现蓝图
/tdd                                          → tdd-guide 强制先写测试
/code-review                                  → code-reviewer 审查代码
```

**修复 Bug：**
```
/tdd                                          → tdd-guide: 编写重现 Bug 的失败测试
                                              → 实现修复，确认测试通过
/code-review                                  → code-reviewer: 回归检查
```

**生产准备：**
```
/security-scan                                → security-reviewer: OWASP Top 10 审计
/e2e                                          → e2e-runner: 核心用户流程测试
/test-coverage                                → 确认 80% 以上覆盖率
```

---

## FAQ

<details>
<summary><b>如何查看已安装的 Agent/命令？</b></summary>

```bash
/plugin list everything-claude-code@everything-claude-code
```

显示插件中可用的所有 Agent、命令、技能。
</details>

<details>
<summary><b>Hook 不工作或显示 "Duplicate hooks file" 错误</b></summary>

这是最常见的问题。**请勿**在 `.claude-plugin/plugin.json` 中添加 `"hooks"` 字段。Claude Code v2.1+ 会自动加载已安装插件的 `hooks/hooks.json`。
</details>

<details>
<summary><b>上下文窗口缩小 / Claude 缺少上下文</b></summary>

太多 MCP 服务器会消耗上下文。每个 MCP 工具描述会在 200k 窗口中消耗 Token，可能缩小到约 70k。

**解决：** 按项目禁用不使用的 MCP：
```json
// 在项目的 .claude/settings.json 中
{
  "disabledMcpServers": ["supabase", "railway", "vercel"]
}
```

保持激活少于 10 个 MCP 和少于 80 个工具。
</details>

<details>
<summary><b>可以只使用部分组件吗？(例如：仅 Agent)</b></summary>

可以。使用选项 2 (手动安装) 只复制需要的内容：

```bash
# 仅 Agent
cp everything-claude-code/agents/*.md ~/.claude/agents/

# 仅规则
cp -r everything-claude-code/rules/common/* ~/.claude/rules/
```

每个组件都是完全独立的。
</details>

<details>
<summary><b>在 Cursor / OpenCode / Codex / Antigravity 中也能工作吗？</b></summary>

可以。ECC 是跨平台的：
- **Cursor**：提供转换后的设置到 `.cursor/`
- **OpenCode**：提供完整插件支持到 `.opencode/`
- **Codex**：macOS 应用和 CLI 均为一等支持
- **Antigravity**：将工作流、技能、扁平化规则集成到 `.agent/`
- **Claude Code**：原生 — 这是主要目标
</details>

<details>
<summary><b>想贡献新技能或 Agent</b></summary>

请参考 [CONTRIBUTING.md](../../CONTRIBUTING.md)。简单来说：
1. Fork 仓库
2. 在 `skills/your-skill-name/SKILL.md` 创建技能 (包含 YAML frontmatter)
3. 或在 `agents/your-agent.md` 创建 Agent
4. 提交带有清晰说明的 PR
</details>

---

## 运行测试

```bash
# 运行所有测试
node tests/run-all.js

# 运行单个测试文件
node tests/lib/utils.test.js
node tests/lib/package-manager.test.js
node tests/hooks/hooks.test.js
```

---

## 贡献

**欢迎贡献。**

本仓库是作为社区资源创建的。如果您有：
- 有用的 Agent 或技能
- 出色的 Hook
- 更好的 MCP 设置
- 改进的规则

请贡献！指南请参考 [CONTRIBUTING.md](../../CONTRIBUTING.md)。

### 贡献创意

- 语言特定技能 (Rust、C#、Swift、Kotlin) — Go、Python、Java 已包含
- 框架特定设置 (Rails、Laravel、FastAPI) — Django、NestJS、Spring Boot 已包含
- DevOps Agent (Kubernetes、Terraform、AWS、Docker)
- 测试策略 (各种框架、视觉回归)
- 领域特定知识 (ML、数据工程、移动)

---

## Token 优化

如果 Claude Code 使用成本成为负担，您需要管理 Token 消耗。通过此设置，您可以在不降低质量的情况下大幅降低成本。

### 推荐设置

添加到 `~/.claude/settings.json`：

```json
{
  "model": "sonnet",
  "env": {
    "MAX_THINKING_TOKENS": "10000",
    "CLAUDE_AUTOCOMPACT_PCT_OVERRIDE": "50"
  }
}
```

| 设置 | 默认值 | 推荐值 | 效果 |
|------|--------|--------|------|
| `model` | opus | **sonnet** | 成本降低 ~60%；可处理 80% 以上的编码工作 |
| `MAX_THINKING_TOKENS` | 31,999 | **10,000** | 每次请求隐藏思维成本降低 ~70% |
| `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` | 95 | **50** | 更早压缩 — 长会话中质量更好 |

只有在需要深度架构推理时才切换到 Opus：
```
/model opus
```

### 日常工作流命令

| 命令 | 使用时机 |
|------|----------|
| `/model sonnet` | 大多数工作的默认值 |
| `/model opus` | 复杂架构、调试、深度推理 |
| `/clear` | 在不相关的工作之间 (免费、立即重置) |
| `/compact` | 在逻辑工作转换点 (研究完成、里程碑达成) |
| `/cost` | 会话期间监控 Token 支出 |

### 上下文窗口管理

**重要：** 请勿一次性激活所有 MCP。每个 MCP 工具描述会在 200k 窗口中消耗 Token，可能缩小到约 70k。

- 每个项目激活少于 10 个 MCP
- 保持激活少于 80 个工具
- 通过项目设置中的 `disabledMcpServers` 禁用不使用的内容

---

## WARNING: 重要注意事项

### 自定义

此设置是为我的工作流创建的。您应该：
1. 从您认同的内容开始
2. 根据您的技术栈进行修改
3. 移除不使用的内容
4. 添加您自己的模式

---

## 赞助

此项目是免费开源软件。通过赞助支持维护和发展。

[**成为赞助商**](https://github.com/sponsors/affaan-m) | [赞助等级](../../SPONSORS.md) | [赞助计划](../../SPONSORING.md)

---

## Star 历史

[![Star History Chart](https://api.star-history.com/svg?repos=affaan-m/everything-claude-code&type=Date)](https://star-history.com/#affaan-m/everything-claude-code&Date)

---

## 链接

- **简明指南 (从这里开始)：** [The Shorthand Guide to Everything Claude Code](https://x.com/affaanmustafa/status/2012378465664745795)
- **详细指南 (高级)：** [The Longform Guide to Everything Claude Code](https://x.com/affaanmustafa/status/2014040193557471352)
- **关注：** [@affaanmustafa](https://x.com/affaanmustafa)
- **zenith.chat：** [zenith.chat](https://zenith.chat)

---

## 许可证

MIT — 自由使用，按需修改，可能的话请贡献。

---

**如果此仓库对您有帮助，请点 Star。请阅读两个指南。创造精彩的作品。**
