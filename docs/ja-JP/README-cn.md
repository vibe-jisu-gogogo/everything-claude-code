**语言:** [English](../../README.md) | [Português (Brasil)](../pt-BR/README.md) | [简体中文](../../README.zh-CN.md) | [繁體中文](../zh-TW/README.md) | [日本語](README.md) | [한국어](../ko-KR/README.md) | [Türkçe](../tr/README.md)

# Everything Claude Code

[![Stars](https://img.shields.io/github/stars/affaan-m/everything-claude-code?style=flat)](https://github.com/affaan-m/everything-claude-code/stargazers)
[![Forks](https://img.shields.io/github/forks/affaan-m/everything-claude-code?style=flat)](https://github.com/affaan-m/everything-claude-code/network/members)
[![Contributors](https://img.shields.io/github/contributors/affaan-m/everything-claude-code?style=flat)](https://github.com/affaan-m/everything-claude-code/graphs/contributors)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
![Shell](https://img.shields.io/badge/-Shell-4EAA25?logo=gnu-bash&logoColor=white)
![TypeScript](https://img.shields.io/badge/-TypeScript-3178C6?logo=typescript&logoColor=white)
![Python](https://img.shields.io/badge/-Python-3776AB?logo=python&logoColor=white)
![Go](https://img.shields.io/badge/-Go-00ADD8?logo=go&logoColor=white)
![Java](https://img.shields.io/badge/-Java-ED8B00?logo=openjdk&logoColor=white)
![Markdown](https://img.shields.io/badge/-Markdown-000000?logo=markdown&logoColor=white)

> **140K+ stars** | **21K+ forks** | **170+ contributors** | **12+ language ecosystems**

---

<div align="center">

**语言 / Language / 語言 / Dil**

[**English**](../../README.md) | [Português (Brasil)](../pt-BR/README.md) | [简体中文](../../README.zh-CN.md) | [繁體中文](../zh-TW/README.md) | [日本語](README.md) | [한국어](../ko-KR/README.md) | [Türkçe](../tr/README.md)

</div>

---

**Anthropic黑客松冠军打造的完整Claude Code配置集合。**

经过10个月以上的高强度日常使用，在实际产品构建过程中不断进化的生产级代理、技能、钩子、命令、规则、MCP配置。

---

## 指南

此仓库仅包含源代码。指南将解释所有内容。

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
<td align="center"><b>简明指南</b><br/>设置、基础、哲学。<b>请先阅读此内容。</b></td>
<td align="center"><b>长篇指南</b><br/>Token优化、内存持久化、评估、并行化。</td>
</tr>
</table>

| 主题 | 学习内容 |
|-------|-------------------|
| Token优化 | 模型选择、系统提示词缩减、后台进程 |
| 内存持久化 | 在会话间自动保存/加载上下文的钩子 |
| 持续学习 | 从会话中自动提取模式并转化为可重用技能 |
| 验证循环 | 检查点和持续评估、评分器类型、pass@k指标 |
| 并行化 | Git工作树、级联方法、扩展时机 |
| 子代理编排 | 上下文问题、迭代搜索模式 |

---

## 新功能

### v1.4.1 — Bug修复（2026年2月）

- **修复instinct导入时内容丢失问题** — 修复了执行`/instinct-import`时`parse_instinct_file()`隐式删除frontmatter后所有内容（Action、Evidence、Examples部分）的问题。社区贡献者@ericcai0814解决了此问题（[#148](https://github.com/affaan-m/everything-claude-code/issues/148), [#161](https://github.com/affaan-m/everything-claude-code/pull/161)）

### v1.4.0 — 多语言规则、安装向导 & PM2（2026年2月）

- **交互式安装向导** — 新的`configure-ecc`技能提供带合并/覆盖检测的引导式设置
- **PM2 & 多代理编排** — 6个新命令用于复杂的多服务工作流管理（`/pm2`, `/multi-plan`, `/multi-execute`, `/multi-backend`, `/multi-frontend`, `/multi-workflow`）
- **多语言规则架构** — 规则从扁平文件重新组织为`common/` + `typescript/` + `python/` + `golang/`目录。仅安装所需语言
- **中文（zh-CN）翻译** — 所有代理、命令、技能、规则的完整翻译（80+文件）
- **GitHub Sponsors支持** — 可通过GitHub Sponsors赞助项目
- **增强的CONTRIBUTING.md** — 各贡献类型的详细PR模板

### v1.3.0 — OpenCode插件支持（2026年2月）

- **完整OpenCode集成** — 通过20+事件类型在OpenCode插件系统中支持钩子的12个代理、24个命令、16个技能
- **3个原生自定义工具** — run-tests、check-coverage、security-audit
- **LLM文档** — 用于全面OpenCode文档的`llms.txt`

### v1.2.0 — 集成命令 & 技能（2026年2月）

- **Python/Django支持** — Django模式、安全性、TDD、验证技能
- **Java Spring Boot技能** — Spring Boot用模式、安全性、TDD、验证
- **会话管理** — 会话历史用的`/sessions`命令
- **持续学习 v2** — 带置信度评分、导入/导出、进化的instinct基础学习

完整变更日志请参考[Releases](https://github.com/affaan-m/everything-claude-code/releases)

---

## 快速开始

2分钟内即可启动：

### 步骤 1：安装插件

```bash
# 添加市场
/plugin marketplace add https://github.com/affaan-m/everything-claude-code

# 安装插件
/plugin install everything-claude-code
```

### 步骤2：安装规则（必须）

> WARNING: **重要:** Claude Code插件无法自动分发`rules`。请手动安装：

```bash
# 首先克隆仓库
git clone https://github.com/affaan-m/everything-claude-code.git

# 安装公共规则（必须）
cp -r everything-claude-code/rules/common/* ~/.claude/rules/

# 安装语言特定规则（选择你的技术栈）
cp -r everything-claude-code/rules/typescript/* ~/.claude/rules/
cp -r everything-claude-code/rules/python/* ~/.claude/rules/
cp -r everything-claude-code/rules/golang/* ~/.claude/rules/
```

### 步骤3：开始使用

```bash
# 尝试命令（插件为命名空间形式）
/everything-claude-code:plan "添加用户认证"

# 手动安装（选项2）为简写形式：
# /plan "添加用户认证"

# 检查可用命令
/plugin list everything-claude-code@everything-claude-code
```

**完成！** 现在你可以访问13个代理、43个技能、31个命令。

---

## 跨平台支持

此插件完全支持 **Windows、macOS、Linux**。所有钩子和脚本都已用Node.js重写，实现最大兼容性。

### 包管理器检测

插件按以下优先级自动检测你偏好的包管理器（npm、pnpm、yarn、bun）：

1. **环境变量**: `CLAUDE_PACKAGE_MANAGER`
2. **项目设置**: `.claude/package-manager.json`
3. **package.json**: `packageManager` 字段
4. **锁文件**: 从package-lock.json、yarn.lock、pnpm-lock.yaml、bun.lockb检测
5. **全局设置**: `~/.claude/package-manager.json`
6. **回退**: 第一个可用的包管理器

要设置你偏好的包管理器：

```bash
# 通过环境变量
export CLAUDE_PACKAGE_MANAGER=pnpm

# 通过全局设置
node scripts/setup-package-manager.js --global pnpm

# 通过项目设置
node scripts/setup-package-manager.js --project bun

# 检测当前设置
node scripts/setup-package-manager.js --detect
```

或在Claude Code中使用`/setup-pm`命令。

---

## 包含内容

此仓库是**Claude Code插件** - 可直接安装，或手动复制组件。

```
everything-claude-code/
|-- .claude-plugin/   # 插件和市场清单
|   |-- plugin.json         # 插件元数据和组件路径
|   |-- marketplace.json    # /plugin marketplace add 用的市场目录
|
|-- agents/           # 委托用的专业子代理
|   |-- planner.md           # 功能实现规划
|   |-- architect.md         # 系统设计决策
|   |-- tdd-guide.md         # 测试驱动开发
|   |-- code-reviewer.md     # 质量和安全审查
|   |-- security-reviewer.md # 漏洞分析
|   |-- build-error-resolver.md
|   |-- e2e-runner.md        # Playwright E2E 测试
|   |-- refactor-cleaner.md  # 死代码删除
|   |-- doc-updater.md       # 文档同步
|   |-- go-reviewer.md       # Go 代码审查
|   |-- go-build-resolver.md # Go 构建错误解决
|   |-- python-reviewer.md   # Python 代码审查（新）
|   |-- database-reviewer.md # 数据库/Supabase 审查（新）
|
|-- skills/           # 工作流定义和领域知识
|   |-- coding-standards/           # 语言最佳实践
|   |-- backend-patterns/           # API、数据库、缓存模式
|   |-- frontend-patterns/          # React、Next.js 模式
|   |-- continuous-learning/        # 从会话自动提取模式（长篇指南）
|   |-- continuous-learning-v2/     # 带置信度评分的直觉基础学习
|   |-- iterative-retrieval/        # 子代理用的渐进式上下文精炼
|   |-- strategic-compact/          # 手动压缩建议（长篇指南）
|   |-- tdd-workflow/               # TDD 方法论
|   |-- security-review/            # 安全检查清单
|   |-- eval-harness/               # 验证循环评估（长篇指南）
|   |-- verification-loop/          # 持续验证（长篇指南）
|   |-- golang-patterns/            # Go 惯用法和最佳实践
|   |-- golang-testing/             # Go 测试模式、TDD、基准测试
|   |-- cpp-testing/                # C++ 测试 GoogleTest、CMake/CTest（新）
|   |-- django-patterns/            # Django 模式、模型、视图（新）
|   |-- django-security/            # Django 安全最佳实践（新）
|   |-- django-tdd/                 # Django TDD 工作流（新）
|   |-- django-verification/        # Django 验证循环（新）
|   |-- python-patterns/            # Python 惯用法和最佳实践（新）
|   |-- python-testing/             # pytest 用的 Python 测试（新）
|   |-- springboot-patterns/        # Java Spring Boot 模式（新）
|   |-- springboot-security/        # Spring Boot 安全（新）
|   |-- springboot-tdd/             # Spring Boot TDD（新）
|   |-- springboot-verification/    # Spring Boot 验证（新）
|   |-- configure-ecc/              # 交互式安装向导（新）
|   |-- security-scan/              # AgentShield 安全审计集成（新）
|
|-- commands/         # 斜杠命令用快速执行
|   |-- tdd.md              # /tdd - 测试驱动开发
|   |-- plan.md             # /plan - 实现规划
|   |-- e2e.md              # /e2e - E2E 测试生成
|   |-- code-review.md      # /code-review - 质量审查
|   |-- build-fix.md        # /build-fix - 构建错误修复
|   |-- refactor-clean.md   # /refactor-clean - 死代码删除
|   |-- learn.md            # /learn - 会话中模式提取（长篇指南）
|   |-- checkpoint.md       # /checkpoint - 保存验证状态（长篇指南）
|   |-- verify.md           # /verify - 执行验证循环（长篇指南）
|   |-- setup-pm.md         # /setup-pm - 设置包管理器
|   |-- go-review.md        # /go-review - Go 代码审查（新）
|   |-- go-test.md          # /go-test - Go TDD 工作流（新）
|   |-- go-build.md         # /go-build - Go 构建错误修复（新）
|   |-- skill-create.md     # /skill-create - 从 Git 历史生成技能（新）
|   |-- instinct-status.md  # /instinct-status - 显示学习的直觉（新）
|   |-- instinct-import.md  # /instinct-import - 导入直觉（新）
|   |-- instinct-export.md  # /instinct-export - 导出直觉（新）
|   |-- evolve.md           # /evolve - 直觉聚类为技能
|   |-- pm2.md              # /pm2 - PM2 服务生命周期管理（新）
|   |-- multi-plan.md       # /multi-plan - 多代理任务分解（新）
|   |-- multi-execute.md    # /multi-execute - 编排多代理工作流（新）
|   |-- multi-backend.md    # /multi-backend - 后端多服务编排（新）
|   |-- multi-frontend.md   # /multi-frontend - 前端多服务编排（新）
|   |-- multi-workflow.md   # /multi-workflow - 通用多服务工作流（新）
|
|-- rules/            # 始终应遵循的指南（复制到 ~/.claude/rules/）
|   |-- README.md            # 结构概述和安装指南
|   |-- common/              # 语言无关原则
|   |   |-- coding-style.md    # 不可变性、文件组织
|   |   |-- git-workflow.md    # 提交格式、PR 流程
|   |   |-- testing.md         # TDD、80% 覆盖率要求
|   |   |-- performance.md     # 模型选择、上下文管理
|   |   |-- patterns.md        # 设计模式、骨架项目
|   |   |-- hooks.md           # 钩子架构、TodoWrite
|   |   |-- agents.md          # 何时委托给子代理
|   |   |-- security.md        # 必须安全检查
|   |-- typescript/          # TypeScript/JavaScript 特定
|   |-- python/              # Python 特定
|   |-- golang/              # Go 特定
|
|-- hooks/            # 触发器基础自动化
|   |-- hooks.json                # 所有钩子设置（PreToolUse、PostToolUse、Stop 等）
|   |-- memory-persistence/       # 会话生命周期钩子（长篇指南）
|   |-- strategic-compact/        # 压缩建议（长篇指南）
|
|-- scripts/          # 跨平台 Node.js 脚本（新）
|   |-- lib/                     # 共享实用程序
|   |   |-- utils.js             # 跨平台文件/路径/系统实用程序
|   |   |-- package-manager.js   # 包管理器检测和选择
|   |-- hooks/                   # 钩子实现
|   |   |-- session-start.js     # 会话开始时加载上下文
|   |   |-- session-end.js       # 会话结束时保存状态
|   |   |-- pre-compact.js       # 压缩前状态保存
|   |   |-- suggest-compact.js   # 战略压缩建议
|   |   |-- evaluate-session.js  # 从会话提取模式
|   |-- setup-package-manager.js # 交互式 PM 设置
|
|-- tests/            # 测试套件（新）
|   |-- lib/                     # 库测试
|   |-- hooks/                   # 钩子测试
|   |-- run-all.js               # 运行所有测试
|
|-- contexts/         # 动态系统提示词注入上下文（长篇指南）
|   |-- dev.md              # 开发模式上下文
|   |-- review.md           # 代码审查模式上下文
|   |-- research.md         # 研究/探索模式上下文
|
|-- examples/         # 设置示例和会话
|   |-- CLAUDE.md           # 项目级别设置示例
|   |-- user-CLAUDE.md      # 用户级别设置示例
|
|-- mcp-configs/      # MCP 服务器设置
|   |-- mcp-servers.json    # GitHub、Supabase、Vercel、Railway 等
|
|-- marketplace.json  # 自托管市场设置（/plugin marketplace add 用）
```

---

## 生态系统工具

### 技能创建工具

从仓库生成Claude Code技能的2种方法：

#### 选项 A：本地分析（内置）

无需外部服务，对本地分析使用`/skill-create`命令：

```bash
/skill-create                    # 分析当前仓库
/skill-create --instincts        # 也为持续学习生成直觉
```

这会在本地分析Git历史，生成SKILL.md文件。

#### 选项 B：GitHub应用（高级功能）

用于高级功能（10k+提交、自动PR、团队共享）：

[安装GitHub应用](https://github.com/apps/skill-creator) | [ecc.tools](https://ecc.tools)

```bash
# 对任意Issue评论：
/skill-creator analyze

# 或推送到默认分支自动触发
```

两种选项生成的内容：
- **SKILL.md文件** - Claude Code中立即可用的技能
- **instinct集合** - 用于continuous-learning-v2
- **模式提取** - 从提交历史学习

### AgentShield — 安全审计工具

扫描Claude Code设置的漏洞、错误配置、注入风险。

```bash
# 快速扫描（无需安装）
npx ecc-agentshield scan

# 自动修复安全问题
npx ecc-agentshield scan --fix

# Opus 4.6 深度分析
npx ecc-agentshield scan --opus --stream

# 从零生成安全设置
npx ecc-agentshield init
```

检查CLAUDE.md、settings.json、MCP服务器、钩子、代理定义。生成安全等级（A-F）和可执行结果。

可在Claude Code中运行`/security-scan`，或通过[GitHub Action](https://github.com/affaan-m/agentshield)添加到CI。

[GitHub](https://github.com/affaan-m/agentshield) | [npm](https://www.npmjs.com/package/ecc-agentshield)

### 持续学习 v2

基于instinct的学习系统自动学习模式：

```bash
/instinct-status        # 显示带置信度的已学习instinct
/instinct-import <file> # 导入他人的instinct
/instinct-export        # 导出instinct进行共享
/evolve                 # 将相关instinct聚类为技能
```

完整文档请参考`skills/continuous-learning-v2/`。

---

## 要求

### Claude Code CLI 版本

**最低版本: v2.1.0 以上**

此插件需要Claude Code CLI v2.1.0+。因为插件系统处理钩子的方式已更改。

检查版本：
```bash
claude --version
```

### 重要: 钩子自动加载行为

> WARNING: **贡献者:** 不要在`.claude-plugin/plugin.json`中添加`"hooks"`字段。这在回归测试中被强制执行。

Claude Code v2.1+自动加载已安装插件的`hooks/hooks.json`（约定）。在`plugin.json`中显式声明会产生错误：

```
Duplicate hook file detected: ./hooks/hooks.json is already resolved to a loaded file
```

**背景:** 这在此仓库中引起了多次修复/回退循环（[#29](https://github.com/affaan-m/everything-claude-code/issues/29), [#52](https://github.com/affaan-m/everything-claude-code/issues/52), [#103](https://github.com/affaan-m/everything-claude-code/issues/103)）。因为Claude Code版本间行为变化造成了混淆。为防止未来问题，有回归测试。

---

## 安装

### 选项1：作为插件安装（推荐）

使用此仓库的最简单方法 - 作为Claude Code插件安装：

```bash
# 添加此仓库为市场
/plugin marketplace add https://github.com/affaan-m/everything-claude-code

# 安装插件
/plugin install everything-claude-code
```

或，直接添加到`~/.claude/settings.json`：

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

现在，立即可访问所有命令、代理、技能、钩子。

> **注:** Claude Code插件系统无法通过插件分发`rules`（[上游限制](https://code.claude.com/docs/en/plugins-reference)）。规则需要手动安装：
>
> ```bash
> # 首先克隆仓库
> git clone https://github.com/affaan-m/everything-claude-code.git
>
> # 选项 A：用户级别规则（应用于所有项目）
> mkdir -p ~/.claude/rules
> cp -r everything-claude-code/rules/common/* ~/.claude/rules/
> cp -r everything-claude-code/rules/typescript/* ~/.claude/rules/   # 选择你的技术栈
> cp -r everything-claude-code/rules/python/* ~/.claude/rules/
> cp -r everything-claude-code/rules/golang/* ~/.claude/rules/
>
> # 选项 B：项目级别规则（仅当前项目）
> mkdir -p .claude/rules
> cp -r everything-claude-code/rules/common/* .claude/rules/
> cp -r everything-claude-code/rules/typescript/* .claude/rules/     # 选择你的技术栈
> ```

---

### 选项2：手动安装

想要手动控制安装内容时：

```bash
# 克隆仓库
git clone https://github.com/affaan-m/everything-claude-code.git

# 将代理复制到Claude设置
cp everything-claude-code/agents/*.md ~/.claude/agents/

# 复制规则（公共 + 语言特定）
cp -r everything-claude-code/rules/common/* ~/.claude/rules/
cp -r everything-claude-code/rules/typescript/* ~/.claude/rules/   # 选择你的技术栈
cp -r everything-claude-code/rules/python/* ~/.claude/rules/
cp -r everything-claude-code/rules/golang/* ~/.claude/rules/

# 复制命令
cp everything-claude-code/commands/*.md ~/.claude/commands/

# 复制技能
cp -r everything-claude-code/skills/* ~/.claude/skills/
```

#### 将钩子添加到settings.json

仅手动安装时，将`hooks/hooks.json`的钩子复制到`~/.claude/settings.json`。

通过`/plugin install`安装ECC时，不要将这些钩子复制到`settings.json`。因为Claude Code v2.1+自动加载插件的`hooks/hooks.json`，双重注册会导致重复执行或`${CLAUDE_PLUGIN_ROOT}`解析失败。

#### 设置MCP

将`mcp-configs/mcp-servers.json`中需要的MCP服务器复制到`~/.claude.json`。

**重要:** 请将`YOUR_*_HERE`占位符替换为实际API密钥。

---

## 核心概念

### 代理

子代理处理有限范围的任务。例如：

```markdown
---
name: code-reviewer
description: 审查代码的质量、安全性、可维护性
tools: ["Read", "Grep", "Glob", "Bash"]
model: opus
---

你是经验丰富的代码审查者...
```

### 技能

技能是由命令或代理调用的工作流定义：

```markdown
# TDD 工作流

1. 首先定义接口
2. 使测试失败 (RED)
3. 实现最小代码 (GREEN)
4. 重构 (IMPROVE)
5. 确认80%+覆盖率
```

### 钩子

钩子由工具事件触发。例如 - 关于console.log的警告：

```json
{
  "matcher": "tool == \"Edit\" && tool_input.file_path matches \"\\\\.(ts|tsx|js|jsx)$\"",
  "hooks": [{
    "type": "command",
    "command": "#!/bin/bash\ngrep -n 'console\\.log' \"$file_path\" && echo '[Hook] Remove console.log' >&2"
  }]
}
```

### 规则

规则是始终应遵循的指南，组织为`common/`（语言无关）+ 语言特定目录：

```
rules/
  common/          # 普遍原则（始终安装）
  typescript/      # TS/JS 特定模式和工具
  python/          # Python 特定模式和工具
  golang/          # Go 特定模式和工具
```

安装和结构详情请参考[`rules/README.md`](rules/README.md)。

---

## 运行测试

插件包含全面的测试套件：

```bash
# 运行所有测试
node tests/run-all.js

# 运行单独的测试文件
node tests/lib/utils.test.js
node tests/lib/package-manager.test.js
node tests/hooks/hooks.test.js
```

---

## 贡献

**贡献非常欢迎，并被鼓励。**

此仓库旨在成为社区资源。如有以下内容：
- 有用的代理或技能
- 巧妙的钩子
- 更好的MCP设置
- 改进的规则

请务必贡献！指南请参考[CONTRIBUTING.md](CONTRIBUTING.md)。

### 贡献想法

- 语言特定技能（Rust、C#、Swift、Kotlin）— Go、Python、Java已包含
- 框架特定设置（Rails、Laravel、FastAPI）— Django、NestJS、Spring Boot已包含
- DevOps代理（Kubernetes、Terraform、AWS、Docker）
- 测试策略（不同框架、视觉回归）
- 专业领域知识（ML、数据工程、移动开发）

---

## Cursor IDE 支持

ecc-universal包含[Cursor IDE](https://cursor.com)的预翻译设置。`.cursor/`目录包含适用于Cursor格式的规则、代理、技能、命令、MCP设置。

### 快速开始 (Cursor)

```bash
# 安装包
npm install ecc-universal

# 安装语言
./install.sh --target cursor typescript
./install.sh --target cursor python golang
```

### 翻译内容

| 组件 | Claude Code → Cursor | 对等性 |
|-----------|---------------------|--------|
| Rules | 添加YAML frontmatter、路径扁平化 | 完整 |
| Agents | 模型ID展开、工具 → 只读标志 | 完整 |
| Skills | 无需更改（同一标准） | 相同 |
| Commands | 更新路径引用、multi-* 存根化 | 部分 |
| MCP Config | 更新环境插值语法 | 完整 |
| Hooks | 无Cursor等效 | 参考其他方法 |

详情请参考[.cursor/README.md](.cursor/README.md)以及完整迁移指南请参考[.cursor/MIGRATION.md](.cursor/MIGRATION.md)。

---

## OpenCode支持

ECC提供**完整OpenCode支持**，包括插件和钩子。

### 快速开始

```bash
# 安装OpenCode
npm install -g opencode

# 在仓库根目录执行
opencode
```

设置从`.opencode/opencode.json`自动检测。

### 功能对等性

| 功能 | Claude Code | OpenCode | 状态 |
|---------|-------------|----------|--------|
| Agents | PASS: 14 代理 | PASS: 12 代理 | **Claude Code领先** |
| Commands | PASS: 30 命令 | PASS: 24 命令 | **Claude Code领先** |
| Skills | PASS: 28 技能 | PASS: 16 技能 | **Claude Code领先** |
| Hooks | PASS: 3 阶段 | PASS: 20+ 事件 | **OpenCode更多！** |
| Rules | PASS: 8 规则 | PASS: 8 规则 | **完全对等** |
| MCP Servers | PASS: 完整 | PASS: 完整 | **完全对等** |
| Custom Tools | PASS: 通过钩子 | PASS: 原生支持 | **OpenCode更好** |

### 通过插件的钩子支持

OpenCode的插件系统比Claude Code更先进，有20+事件类型：

| Claude Code 钩子 | OpenCode 插件事件 |
|-----------------|----------------------|
| PreToolUse | `tool.execute.before` |
| PostToolUse | `tool.execute.after` |
| Stop | `session.idle` |
| SessionStart | `session.created` |
| SessionEnd | `session.deleted` |

**额外OpenCode事件**: `file.edited`, `file.watcher.updated`, `message.updated`, `lsp.client.diagnostics`, `tui.toast.show`等。

### 可用命令（24）

| 命令 | 说明 |
|---------|-------------|
| `/plan` | 创建实现计划 |
| `/tdd` | 执行TDD工作流 |
| `/code-review` | 审查代码变更 |
| `/security` | 执行安全审查 |
| `/build-fix` | 修复构建错误 |
| `/e2e` | 生成E2E测试 |
| `/refactor-clean` | 删除死代码 |
| `/orchestrate` | 多代理工作流 |
| `/learn` | 从会话提取模式 |
| `/checkpoint` | 保存验证状态 |
| `/verify` | 执行验证循环 |
| `/eval` | 对照基准评估 |
| `/update-docs` | 更新文档 |
| `/update-codemaps` | 更新代码映射 |
| `/test-coverage` | 分析覆盖率 |
| `/go-review` | Go代码审查 |
| `/go-test` | Go TDD工作流 |
| `/go-build` | 修复Go构建错误 |
| `/skill-create` | 从Git生成技能 |
| `/instinct-status` | 显示已学习直觉 |
| `/instinct-import` | 导入直觉 |
| `/instinct-export` | 导出直觉 |
| `/evolve` | 将直觉聚类为技能 |
| `/setup-pm` | 设置包管理器 |

### 插件安装

**选项1：直接使用**
```bash
cd everything-claude-code
opencode
```

**选项2：作为npm包安装**
```bash
npm install ecc-universal
```

然后添加到`opencode.json`：
```json
{
  "plugin": ["ecc-universal"]
}
```

### 文档

- **迁移指南**: `.opencode/MIGRATION.md`
- **OpenCode 插件 README**: `.opencode/README.md`
- **集成规则**: `.opencode/instructions/INSTRUCTIONS.md`
- **LLM 文档**: `llms.txt`（完整OpenCode文档）

---

## 背景

自实验性发布以来，我们一直使用Claude Code。2025年9月，与[@DRodriguezFX](https://x.com/DRodriguezFX)一起使用Claude Code构建[zenith.chat](https://zenith.chat)，并在Anthropic x Forum Ventures黑客松中获胜。

这些设置已在多个生产应用程序中经过实战测试。

---

## WARNING: 重要说明

### 上下文窗口管理

**重要:** 不要一次启用所有MCP。启用太多工具可能会导致200k上下文窗口缩小到70k。

经验法则：
- 设置20-30个MCP
- 每个项目保持启用少于10个
- 活动工具少于80个

在项目设置中使用`disabledMcpServers`禁用未使用的工具。

### 自定义

这些设置是为我的工作流设计的。你应该：
1. 从你认同的部分开始
2. 根据技术栈进行修改
3. 删除不使用的部分
4. 添加你自己的模式

---

## Star 历史

[![Star History Chart](https://api.star-history.com/svg?repos=affaan-m/everything-claude-code&type=Date)](https://star-history.com/#affaan-m/everything-claude-code&Date)

---

## 链接

- **简明指南（先从这里开始）:** [Everything Claude Code 简明指南](https://x.com/affaanmustafa/status/2012378465664745795)
- **详细指南（高级）:** [Everything Claude Code 详细指南](https://x.com/affaanmustafa/status/2014040193557471352)
- **关注:** [@affaanmustafa](https://x.com/affaanmustafa)
- **zenith.chat:** [zenith.chat](https://zenith.chat)
- **技能目录:** awesome-agent-skills（社区管理的代理技能目录）

---

## 许可证

MIT - 自由使用、按需修改、尽可能贡献。

---

**如果此仓库对你有帮助，请给Star。请阅读两份指南。请构建出色的东西。**
