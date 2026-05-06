**语言:** [English](../../README.md) | [简体中文](../../README.zh-CN.md) | [繁體中文](../zh-TW/README.md) | [日本語](../ja-JP/README.md) | [한국어](../ko-KR/README.md) | Português (Brasil) | [Türkçe](../tr/README.md)

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

> **140K+ stars** | **21K+ forks** | **170+ contributors** | **12+ language ecosystems** | **Anthropic Hackathon Winner**

---

<div align="center">

**语言 / Language / 语言 / Dil**

[**English**](../../README.md) | [简体中文](../../README.zh-CN.md) | [繁體中文](../zh-TW/README.md) | [日本語](../ja-JP/README.md) | [한국어](../ko-KR/README.md) | [Português (Brasil)](README.md) | [Türkçe](../tr/README.md)

</div>

---

**AI Agent Harness 性能优化系统。来自 Anthropic 黑客马拉松获胜者。**

不只是配置。一个完整的系统：skills、instincts、内存优化、持续学习、安全扫描和研究优先的开发。经过 10+ 个月日常深度使用构建真实产品所开发的、生产就绪的 agents、hooks、commands、rules 和 MCP 配置。

兼容 **Claude Code**、**Codex**、**Cursor**、**OpenCode**、**Gemini** 和其他 AI Agent harnesses。

---

## 指南

这个仓库只包含代码。指南解释了一切。

<table>
<tr>
<td width="33%">
<a href="https://x.com/affaanmustafa/status/2012378465664745795">
<img src="../../assets/images/guides/shorthand-guide.png" alt="The Shorthand Guide to Everything Claude Code" />
</a>
</td>
<td width="33%">
<a href="https://x.com/affaanmustafa/status/2014040193557471352">
<img src="../../assets/images/guides/longform-guide.png" alt="The Longform Guide to Everything Claude Code" />
</a>
</td>
<td width="33%">
<a href="https://x.com/affaanmustafa/status/2033263813387223421">
<img src="../../assets/images/security/security-guide-header.png" alt="The Shorthand Guide to Everything Agentic Security" />
</a>
</td>
</tr>
<tr>
<td align="center"><b>简明指南</b><br/>配置、基础、哲学。<b>先读这个。</b></td>
<td align="center"><b>完整指南</b><br/>Token 优化、内存持久化、evals、并行化。</td>
<td align="center"><b>安全指南</b><br/>攻击向量、sandboxing、净化、CVEs、AgentShield。</td>
</tr>
</table>

| 主题 | 你将学到什么 |
|--------|----------------------|
| Token 优化 | 模型选择、系统提示压缩、后台进程 |
| 内存持久化 | 自动在会话间保存/加载上下文的 Hooks |
| 持续学习 | 从会话中自动提取模式为可复用的 skills |
| 验证循环 | Checkpoint vs 持续 evals、评估器类型、pass@k 指标 |
| 并行化 | Git worktrees、级联方法、何时扩展实例 |
| 子代理编排 | 上下文问题、迭代恢复模式 |

---

## 新功能

### v2.0.0-rc.1 — 表面同步、运营流程和 ECC 2.0 Alpha（2026 年 4 月）

- **与真实仓库同步的公共表面** — 元数据、目录计数、插件清单和安装文档现在反映了实际交付的 OSS 表面。
- **运营和外部流程扩展** — `brand-voice`、`social-graph-ranker`、`customer-billing-ops`、`google-workspace-ops` 和相关 skills 强化了同一系统内的运营轨迹。
- **媒体和发布工具** — `manim-video`、`remotion-video-creation` 和社交流程将技术解释和发布置于同一仓库中。
- **框架和产品表面增长** — `nestjs-patterns`、为 Codex/OpenCode 提供更丰富的安装表面以及跨 harness 打包改进将使用范围扩展到了 Claude Code 之外。
- **ECC 2.0 alpha 已在仓库中** — `ecc2/` 中的 Rust 控制平面已可本地编译，并暴露了 `dashboard`、`start`、`sessions`、`status`、`stop`、`resume` 和 `daemon`。
- **生态系统强化** — AgentShield、ECC Tools 成本控制、计费门户工作以及网站更新继续围绕核心插件交付。

### v1.9.0 — 选择性安装和语言扩展（2026 年 3 月）

- **选择性安装架构** — 基于清单的安装管道，使用 `install-plan.js` 和 `install-apply.js` 进行定向组件安装。状态存储跟踪已安装内容并启用增量更新。
- **6 个新 agents** — `typescript-reviewer`、`pytorch-build-resolver`、`java-build-resolver`、`java-reviewer`、`kotlin-reviewer`、`kotlin-build-resolver` 将覆盖范围扩展到 10 种语言。
- **新 skills** — 用于深度学习流程的 `pytorch-patterns`、用于 API 参考查找的 `documentation-lookup`、用于现代 JS 工具链的 `bun-runtime` 和 `nextjs-turbopack`，以及 8 个运营领域 skills 和 `mcp-server-patterns`。
- **会话和状态基础设施** — 带有查询 CLI 的 SQLite 状态存储、用于结构化记录的会话适配器、用于自我改进 skills 的 skill 演进基础。
- **编排审查** — Harness 审计评分变为确定性，编排状态和启动器兼容性得到加强，通过 5 层防护防止 observer 循环。
- **Observer 可靠性** — 通过 throttling 和尾部采样修复内存爆炸，修复沙箱访问，延迟启动逻辑和可重入防护。
- **12 个语言生态系统** — Java、PHP、Perl、Kotlin/Android/KMP、C++ 和 Rust 的新规则加入了现有的 TypeScript、Python、Go 和通用规则。
- **社区贡献** — 韩语和中文翻译，Biome hook 优化，VideoDB skills，Evos 运营 skills，PowerShell 安装程序，Antigravity IDE 支持。
- **CI 强化** — 19 个测试失败修复，目录计数应用，安装清单验证和完整测试套件变绿。

### v1.8.0 — Harness 性能系统（2026 年 3 月）

- **以 Harness 为重点的发布** — ECC 现在被明确框定为 AI Agent Harness 性能系统，而不仅仅是配置包。
- **Hook 可靠性审查** — SessionStart 根回退，Stop 阶段的会话摘要以及基于脚本的 hooks 替换了脆弱的内联 one-liners。
- **Hook 运行时控制** — `ECC_HOOK_PROFILE=minimal|standard|strict` 和 `ECC_DISABLED_HOOKS=...` 用于无需编辑 hook 文件的运行时控制。
- **新 Harness 命令** — `/harness-audit`、`/loop-start`、`/loop-status`、`/quality-gate`、`/model-route`。
- **NanoClaw v2** — 模型路由，Skill 热加载，会话分支/搜索/导出/压缩/指标。
- **Harness 间一致性** — Claude Code、Cursor、OpenCode 和 Codex app/CLI 的统一行为。
- **997 个内部测试通过** — 在 Hook/运行时重构和兼容性更新后，完整套件变绿。

---

## 快速开始

2 分钟内开始：

### 步骤 1：安装插件

```bash
# 添加 marketplace
/plugin marketplace add https://github.com/affaan-m/everything-claude-code

# 安装插件
/plugin install everything-claude-code@everything-claude-code
```

### 步骤 2：安装规则（必需）

> WARNING: **重要：** Claude Code 插件无法自动分发 `rules`。请手动安装：

```bash
# 先克隆仓库
git clone https://github.com/affaan-m/everything-claude-code.git
cd everything-claude-code

# 安装依赖（选择你的包管理器）
npm install        # 或: pnpm install | yarn install | bun install

# macOS/Linux
./install.sh typescript    # 或 python 或 golang 或 swift 或 php
# ./install.sh typescript python golang swift php
# ./install.sh --target cursor typescript
# ./install.sh --target antigravity typescript
```

```powershell
# Windows PowerShell
.\install.ps1 typescript   # 或 python 或 golang 或 swift 或 php
# .\install.ps1 typescript python golang swift php
# .\install.ps1 --target cursor typescript
# .\install.ps1 --target antigravity typescript

# npm 兼容入口点也可跨平台工作
npx ecc-install typescript
```

### 步骤 3：开始使用

```bash
# 尝试一个命令（插件安装使用带命名空间的形式）
/everything-claude-code:plan "添加用户认证"

# 手动安装（选项 2）使用更短的形式：
# /plan "添加用户认证"

# 查看可用命令
/plugin list everything-claude-code@everything-claude-code
```

**完成！** 你现在可以访问 28 个 agents、116 个 skills 和 59 个 commands。

---

## 跨平台支持

此插件现在完全支持 **Windows、macOS 和 Linux**，与主流 IDE（Cursor、OpenCode、Antigravity）和 CLI harnesses 紧密集成。所有 hooks 和脚本都已用 Node.js 重写，以实现最大兼容性。

### 包管理器检测

插件按以下优先级自动检测你喜欢的包管理器（npm、pnpm、yarn 或 bun）：

1. **环境变量**: `CLAUDE_PACKAGE_MANAGER`
2. **项目配置**: `.claude/package-manager.json`
3. **package.json**: `packageManager` 字段
4. **Lock 文件**: 通过 package-lock.json、yarn.lock、pnpm-lock.yaml 或 bun.lockb 检测
5. **全局配置**: `~/.claude/package-manager.json`
6. **回退**: 第一个可用的包管理器（pnpm > bun > yarn > npm）

设置你喜欢的包管理器：

```bash
# 通过环境变量
export CLAUDE_PACKAGE_MANAGER=pnpm

# 通过全局配置
node scripts/setup-package-manager.js --global pnpm

# 通过项目配置
node scripts/setup-package-manager.js --project bun

# 检测当前配置
node scripts/setup-package-manager.js --detect
```

或在 Claude Code 中使用 `/setup-pm` 命令。

### Hook 运行时控制

使用运行时标志调整严格程度或临时禁用特定 hooks：

```bash
# Hooks 严格程度配置文件（默认: standard）
export ECC_HOOK_PROFILE=standard

# 逗号分隔的 hook IDs 以禁用
export ECC_DISABLED_HOOKS="pre:bash:tmux-reminder,post:edit:typecheck"
```

---

## 包含内容

```
everything-claude-code/
|-- agents/           # 28 个用于委派的专用子代理
|-- skills/           # 工作流定义和领域知识
|-- commands/         # 快速执行的斜杠命令
|-- rules/            # 始终遵循的指南（复制到 ~/.claude/rules/）
|-- hooks/            # 基于触发器的自动化
|-- scripts/          # 跨平台 Node.js 脚本
|-- tests/            # 测试套件
|-- contexts/         # 系统提示注入上下文
|-- examples/         # 示例配置和会话
|-- mcp-configs/      # MCP 服务器配置
```

---

## 生态系统工具

### Skill 创建器

两种从你的仓库生成 Claude Code skills 的模式：

#### 选项 A：本地分析（集成）

使用 `/skill-create` 命令进行无外部服务的本地分析：

```bash
/skill-create                    # 分析当前仓库
/skill-create --instincts        # 也为 continuous-learning 生成 instincts
```

#### 选项 B：GitHub App（高级）

对于高级功能（10k+ commits、自动 PRs、团队共享）：

[安装 GitHub App](https://github.com/apps/skill-creator) | [ecc.tools](https://ecc.tools)

### AgentShield — 安全审计器

> 在 Claude Code Hackathon 中构建（Cerebral Valley x Anthropic，2026 年 2 月）。1282 个测试，98% 覆盖率，102 条静态分析规则。

```bash
# 快速扫描（无需安装）
npx ecc-agentshield scan

# 自动修复安全问题
npx ecc-agentshield scan --fix

# 使用三个 Opus 4.6 agents 进行深度分析
npx ecc-agentshield scan --opus --stream

# 从头生成安全配置
npx ecc-agentshield init
```

### 持续学习 v2

基于 instinct 的学习系统自动学习你的模式：

```bash
/instinct-status        # 显示带置信度的已学习 instincts
/instinct-import <file> # 从他人导入 instincts
/instinct-export        # 导出你的 instincts 以共享
/evolve                 # 将相关 instincts 分组为 skills
```

---

## 要求

### Claude Code CLI 版本

**最低版本: v2.1.0 或更高**

检查你的版本：
```bash
claude --version
```

---

## 安装

### 选项 1：作为插件安装（推荐）

```bash
# 添加此仓库作为 marketplace
/plugin marketplace add https://github.com/affaan-m/everything-claude-code

# 安装插件
/plugin install everything-claude-code@everything-claude-code
```

或直接添加到你的 `~/.claude/settings.json`：

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

> **注意：** Claude Code 插件系统不支持通过插件分发 `rules`。你需要手动安装规则：
>
> ```bash
> # 先克隆仓库
> git clone https://github.com/affaan-m/everything-claude-code.git
>
> # 选项 A：用户级规则（应用于所有项目）
> mkdir -p ~/.claude/rules
> cp -r everything-claude-code/rules/common/* ~/.claude/rules/
> cp -r everything-claude-code/rules/typescript/* ~/.claude/rules/   # 选择你的技术栈
>
> # 选项 B：项目级规则（仅应用于当前项目）
> mkdir -p .claude/rules
> cp -r everything-claude-code/rules/common/* .claude/rules/
> ```

---

### 选项 2：手动安装

```bash
# 克隆仓库
git clone https://github.com/affaan-m/everything-claude-code.git

# 复制 agents 到你的 Claude 配置
cp everything-claude-code/agents/*.md ~/.claude/agents/

# 复制规则（通用 + 语言特定）
cp -r everything-claude-code/rules/common/* ~/.claude/rules/
cp -r everything-claude-code/rules/typescript/* ~/.claude/rules/

# 复制命令
cp everything-claude-code/commands/*.md ~/.claude/commands/

# 复制 skills（核心 vs 细分）
cp -r everything-claude-code/.agents/skills/* ~/.claude/skills/
```

---

## 关键概念

### Agents

子代理处理范围有限的委派任务。

### Skills

Skills 是由命令或 agents 调用的工作流定义。

### Hooks

Hooks 在工具事件上触发。示例 — 警告 console.log：

```json
{
  "matcher": "tool == \"Edit\" && tool_input.file_path matches \"\\\\.(ts|tsx|js|jsx)$\"",
  "hooks": [{
    "type": "command",
    "command": "#!/bin/bash\ngrep -n 'console\\.log' \"$file_path\" && echo '[Hook] 移除 console.log' >&2"
  }]
}
```

### Rules

Rules 是始终遵循的指南，组织在 `common/`（语言无关）+ 特定语言目录中。

---

## 我应该使用哪个 Agent？

| 我想要... | 使用此命令 | 使用的 Agent |
|----------|-----------------|--------------|
| 计划新功能 | `/everything-claude-code:plan "添加 auth"` | planner |
| 设计系统架构 | `/everything-claude-code:plan` + architect agent | architect |
| 测试优先写代码 | `/tdd` | tdd-guide |
| 审查刚写完的代码 | `/code-review` | code-reviewer |
| 修复失败的构建 | `/build-fix` | build-error-resolver |
| 执行端到端测试 | `/e2e` | e2e-runner |
| 发现安全漏洞 | `/security-scan` | security-reviewer |
| 移除死代码 | `/refactor-clean` | refactor-cleaner |
| 更新文档 | `/update-docs` | doc-updater |
| 审查 Go 代码 | `/go-review` | go-reviewer |
| 审查 Python 代码 | `/python-review` | python-reviewer |

### 常见工作流

**开始新功能：**
```
/everything-claude-code:plan "使用 OAuth 添加用户认证"
                                              → planner 创建实现蓝图
/tdd                                          → tdd-guide 应用测试优先编写
/code-review                                  → code-reviewer 检查你的工作
```

**修复 Bug：**
```
/tdd                                          → tdd-guide: 编写重现 bug 的失败测试
                                              → 实现修复，验证测试通过
/code-review                                  → code-reviewer: 检测回归
```

**准备生产：**
```
/security-scan                                → security-reviewer: OWASP Top 10 审计
/e2e                                          → e2e-runner: 关键用户流测试
/test-coverage                                → 验证 80%+ 覆盖率
```

---

## FAQ

<details>
<summary><b>如何检查安装了哪些 agents/commands？</b></summary>

```bash
/plugin list everything-claude-code@everything-claude-code
```
</details>

<details>
<summary><b>我的 hooks 不工作 / 看到 "Duplicate hooks file" 错误</b></summary>

这是最常见的问题。**不要**向 `.claude-plugin/plugin.json` 添加 `"hooks"` 字段。Claude Code v2.1+ 会自动从已安装的插件加载 `hooks/hooks.json`。显式声明会导致重复检测错误。
</details>

<details>
<summary><b>我可以将 ECC 与 Cursor / OpenCode / Codex / Antigravity 一起使用吗？</b></summary>

可以。ECC 是跨平台的：
- **Cursor**: `.cursor/` 中的预转换配置
- **OpenCode**: `.opencode/` 中的完整插件支持
- **Codex**: 对 macOS app 和 CLI 的一流支持
- **Antigravity**: `.agent/` 中的集成配置
- **Claude Code**: 原生 — 这是主要目标
</details>

<details>
<summary><b>如何贡献新的 skill 或 agent？</b></summary>

参见 [CONTRIBUTING.md](CONTRIBUTING.md)。简而言之：
1. Fork 仓库
2. 在 `skills/your-skill-name/SKILL.md` 中创建你的 skill（带 YAML frontmatter）
3. 或在 `agents/your-agent.md` 中创建一个 agent
4. 提交带有清晰功能描述和使用时机的 PR
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

**欢迎并鼓励贡献。**

这个仓库是社区的资源。如果你有：
- 有用的 agents 或 skills
- 智能的 hooks
- 更好的 MCP 配置
- 改进的规则

请贡献！参见 [CONTRIBUTING.md](CONTRIBUTING.md) 了解指南。

---

## 许可证

MIT — 有关详细信息，请参阅 [LICENSE 文件](../../LICENSE)。
