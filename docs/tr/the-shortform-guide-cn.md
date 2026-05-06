# Claude Code 简明指南

![Header: Anthropic Hackathon Winner - Tips &amp; Tricks for Claude Code](../assets/images/shortform/00-header.png)

---

**自二月份作为实验性功能推出以来，我一直是 Claude Code 的热衷用户，并且与 [@DRodriguezFX](https://x.com/DRodriguezFX) 一起，完全使用 Claude Code 开发的 [zenith.chat](https://zenith.chat) 项目赢得了 Anthropic x Forum Ventures hackathon。**

以下是我经过 10 个月日常使用后的完整设置：skill、hook、subagent、MCP、plugin，以及真正有用的功能。

---

## Skill 与 Command

Skill 的工作方式类似于针对特定范围和工作流程的约束规则。当您需要执行特定工作流程时，它们充当 prompt 的快捷方式。

想在 Opus 4.5 漫长编码会话后清理死代码和松散的 .md 文件？运行 `/refactor-clean`。需要测试？`/tdd`、`/e2e`、`/test-coverage`。Skill 还可以包含 codemap——一种让 Claude 无需在探索上花费 context 即可快速浏览代码库的方法。

![Terminal showing chained commands](../assets/images/shortform/02-chaining-commands.jpeg)
*链接多个 command*

Command 是通过 slash command 执行的 skill。它们有重叠但存储方式不同：

- **Skill**: `~/.claude/skills/` - 更广泛的工作流程定义
- **Command**: `~/.claude/commands/` - 快速可执行的 prompt

```bash
# 示例 skill 结构
~/.claude/skills/
  pmx-guidelines.md      # 项目特定模式
  coding-standards.md    # 语言相关最佳实践
  tdd-workflow/          # 多文件 skill，带 README.md
  security-review/       # 基于清单的 skill
```

---

## Hook

Hook 是在特定事件上触发的自动化程序。与 skill 不同，它们仅限于工具调用和生命周期事件。

**Hook 类型：**

1. **PreToolUse** - 工具运行前（验证、提醒）
2. **PostToolUse** - 工具完成后（格式化、反馈循环）
3. **UserPromptSubmit** - 发送消息时
4. **Stop** - Claude 停止响应时
5. **PreCompact** - context 压缩前
6. **Notification** - 权限请求

**示例：长时间运行命令前的 tmux 提醒**

```json
{
  "PreToolUse": [
    {
      "matcher": "tool == \"Bash\" &amp;&amp; tool_input.command matches \"(npm|pnpm|yarn|cargo|pytest)\"",
      "hooks": [
        {
          "type": "command",
          "command": "if [ -z \"$TMUX\" ]; then echo '[Hook] Consider tmux for session persistence' &gt;&amp;2; fi"
        }
      ]
    }
  ]
}
```

![PostToolUse hook feedback](../assets/images/shortform/03-posttooluse-hook.png)
*PostToolUse hook 运行时您在 Claude Code 中收到的反馈示例*

**专业提示：** 不用手动编写 JSON，使用 `hookify` plugin 通过对话创建 hook。运行 `/hookify` 并描述您想要什么。

---

## Subagent

Subagent 是您的主 Claude（协调器）可以委派具有有限范围任务的进程。它们可以在后台或前台运行，为主 agent 释放 context。

Subagent 与 skill 配合得很好——您可以将任务委派给能够执行您的 skill 子集的 subagent，并让它们自主使用这些 skill。它们还可以使用特定工具权限进行沙箱化。

```bash
# 示例 subagent 结构
~/.claude/agents/
  planner.md           # 功能实施规划
  architect.md         # 系统设计决策
  tdd-guide.md         # 测试驱动开发
  code-reviewer.md     # 质量/安全审查
  security-reviewer.md # 漏洞分析
  build-error-resolver.md
  e2e-runner.md
  refactor-cleaner.md
```

为了正确的范围界定，请为每个 subagent 配置允许的工具、MCP 和权限。

---

## Rule 与 Memory

您的 `.rules` 文件夹包含 Claude 应**始终**遵循的最佳实践的 `.md` 文件。两种方法：

1. **单一 CLAUDE.md** - 所有内容在一个文件中（用户或项目级别）
2. **Rules 文件夹** - 按关注点分组的模块化 `.md` 文件

```bash
~/.claude/rules/
  security.md      # 无硬编码的 secret，验证输入
  coding-style.md  # 不变性，文件组织
  testing.md       # TDD 工作流程，80% coverage
  git-workflow.md  # Commit 格式，PR 流程
  agents.md        # 何时委派给 subagent
  performance.md   # 模型选择，context 管理
```

**示例 rule：**

- 代码库中无 emoji
- 避免前端中的紫色调
- 部署前始终测试
- 优先模块化代码而非大文件
- 永远不要 commit `console.log`

---

## MCP (Model Context Protocol)

MCP 将 Claude 直接连接到外部服务。它们不替代 API——是围绕 API 的 prompt 优先包装器，在信息导航方面提供更大的灵活性。

**示例：** Supabase MCP 允许 Claude 拉取特定数据，无需复制粘贴即可直接在 upstream 运行 SQL。数据库、部署平台等也是如此。

![Supabase MCP listing tables](../assets/images/shortform/04-supabase-mcp.jpeg)
*Supabase MCP 在 public schema 中列出表示例*

**Claude 中的 Chrome：** 一个内置的 plugin MCP，允许 Claude 自主控制您的浏览器——点击各处了解工作原理。

**关键：Context Window 管理**

对 MCP 要有选择性。我在用户配置中保留所有 MCP，但**禁用所有未使用的**。转到 `/plugins` 并向下滚动，或运行 `/mcp`。

![/plugins interface](../assets/images/shortform/05-plugins-interface.jpeg)
*使用 `/plugins` 导航到 MCP 以查看当前安装了哪些 MCP 及其状态*

如果激活了太多工具，您的 200k context window 在压缩前可能只有 70k。性能会显著下降。

**一般规则：** 在配置中保留 20-30 个 MCP，但保持少于 10 个启用 / 少于 80 个活跃工具。

```bash
# 检查活跃的 MCP
/mcp

# 在 ~/.claude.json 的 projects.disabledMcpServers 下禁用未使用的
```

---

## Plugin

Plugin 打包工具以简化安装，而不是繁琐的手动设置。plugin 可以是组合的 skill + MCP 或打包在一起的 hook/工具。

**安装 plugin：**

```bash
# 添加 marketplace
# @mixedbread-ai 的 mgrep plugin
claude plugin marketplace add https://github.com/mixedbread-ai/mgrep

# 打开 Claude，运行 /plugins，找到新 marketplace，从那里安装
```

![Marketplaces tab showing mgrep](../assets/images/shortform/06-marketplaces-mgrep.jpeg)
*显示新安装的 Mixedbread-Grep marketplace*

**LSP Plugin** 如果您经常在编辑器外运行 Claude Code 特别有用。Language Server Protocol 为 Claude 提供实时类型检查、定义导航和智能补全，无需打开 IDE。

```bash
# 活跃 plugin 示例
typescript-lsp@claude-plugins-official  # TypeScript 智能
pyright-lsp@claude-plugins-official     # Python 类型检查
hookify@claude-plugins-official         # 通过对话创建 hook
mgrep@Mixedbread-Grep                   # 比 ripgrep 更好的搜索
```

与 MCP 相同的警告——关注您的 context window。

---

## 提示与技巧

### 键盘快捷键

- `Ctrl+U` - 删除整行（比反复按 backspace 更快）
- `!` - 快速 bash 命令前缀
- `@` - 文件搜索
- `/` - 启动 slash command
- `Shift+Enter` - 多行输入
- `Tab` - 切换思考视图
- `Esc Esc` - 中断 Claude / 恢复代码

### 并行工作流

- **Fork** (`/fork`) - fork 对话而不是排队消息 spam，以并行执行不冲突的任务
- **Git Worktree** - 无冲突的并行 Claude 的重叠工作。每个 worktree 是独立的 checkout

```bash
git worktree add ../feature-branch feature-branch
# 现在在每个 worktree 中运行单独的 Claude instance
```

### 长时间运行命令的 tmux

流式传输和监控 Claude 运行的 log/bash 进程：

&lt;https://github.com/user-attachments/assets/shortform/07-tmux-video.mp4&gt;

```bash
tmux new -s dev
# Claude 在这里运行命令，您可以分离和重新连接
tmux attach -t dev
```

### mgrep &gt; grep

`mgrep` 是 ripgrep/grep 的重大进步。通过 plugin marketplace 安装，然后使用 `/mgrep` skill。它同时适用于本地搜索和 Web 搜索。

```bash
mgrep "function handleSubmit"  # 本地搜索
mgrep --web "Next.js 15 app router changes"  # Web 搜索
```

### 其他有用的 Command

- `/rewind` - 回滚到之前状态
- `/statusline` - 使用分支、context %、todo 自定义
- `/checkpoints` - 文件级回滚点
- `/compact` - 手动触发 context 压缩

### GitHub Actions CI/CD

在您的 PR 上使用 GitHub Actions 设置代码审查。配置后 Claude 可以自动审查 PR。

![Claude bot approving a PR](../assets/images/shortform/08-github-pr-review.jpeg)
*Claude 批准 bug 修复 PR*

### 沙箱

对风险操作使用沙箱模式——Claude 在受限环境中运行而不影响您的真实系统。

---

## 关于编辑器

您的编辑器选择会显著影响 Claude Code 工作流程。虽然 Claude Code 从任何终端运行，但与功能强大的编辑器配对可提供实时文件跟踪、快速导航和集成命令执行。

### Zed（我的选择）

我使用 [Zed](https://zed.dev)——用 Rust 编写，所以它真的很快。即时打开，处理大型代码库毫不费力，几乎不接触系统资源。

**为什么 Zed + Claude Code 是很棒的组合：**

- **速度** - 基于 Rust 的性能意味着当 Claude 快速编辑文件时无延迟。您的编辑器可以跟上
- **Agent Panel 集成** - Zed 的 Claude 集成允许您在 Claude 编辑时实时跟踪文件更改。无需离开编辑器即可在 Claude 引用的文件之间跳转
- **CMD+Shift+R Command Palette** - 通过可搜索 UI 快速访问您所有的自定义 slash command、debugger、build 脚本
- **最小资源使用** - 繁重操作期间不与 Claude 竞争 RAM/CPU。运行 Opus 时很重要
- **Vim 模式** - 如果这是您的风格，完整的 vim 键绑定

![Zed Editor with custom commands](../assets/images/shortform/09-zed-editor.jpeg)
*Zed Editor 使用 CMD+Shift+R 显示自定义命令下拉菜单。Following 模式显示为右下角的目标图标*

**编辑器无关提示：**

1. **拆分屏幕** - 一侧是带 Claude Code 的终端，另一侧是编辑器
2. **Ctrl + G** - 在 Zed 中快速打开 Claude 正在处理的文件
3. **自动保存** - 启用自动保存，以便 Claude 的文件读取始终是最新的
4. **Git 集成** - 在 commit 前使用编辑器的 git 功能检查 Claude 的更改
5. **文件监视器** - 大多数编辑器自动重新加载更改的文件，验证已启用

### VSCode / Cursor

这也是一个有效的选择，并且与 Claude Code 配合良好。您可以将其与终端格式一起使用，通过 `\ide` 启用 LSP 功能自动与您的编辑器同步（现在使用 plugin 有点多余）。或者您可能更喜欢 extension，它与编辑器更集成并具有匹配的 UI。

![VS Code Claude Code Extension](../assets/images/shortform/10-vscode-extension.jpeg)
*VS Code extension 为直接集成到您的 IDE 中的 Claude Code 提供原生图形界面*

---

## 我的设置

### Plugin

**已安装：**（通常一次只保持 4-5 个活跃）

```markdown
ralph-wiggum@claude-code-plugins       # Loop 自动化
frontend-patterns@claude-code-plugins  # UI/UX 模式
commit-commands@claude-code-plugins    # Git 工作流
security-guidance@claude-code-plugins  # 安全检查
pr-review-toolkit@claude-code-plugins  # PR 自动化
typescript-lsp@claude-plugins-official # TS 智能
hookify@claude-plugins-official        # Hook 创建
code-simplifier@claude-plugins-official
feature-dev@claude-code-plugins
explanatory-output-style@claude-code-plugins
code-review@claude-code-plugins
context7@claude-plugins-official       # 实时文档
pyright-lsp@claude-plugins-official    # Python 类型
mgrep@Mixedbread-Grep                  # 更好的搜索
```

### MCP Server

**已配置（用户级别）：**

```json
{
  "github": { "command": "npx", "args": ["-y", "@modelcontextprotocol/server-github"] },
  "firecrawl": { "command": "npx", "args": ["-y", "firecrawl-mcp"] },
  "supabase": {
    "command": "npx",
    "args": ["-y", "@supabase/mcp-server-supabase@latest", "--project-ref=YOUR_REF"]
  },
  "memory": { "command": "npx", "args": ["-y", "@modelcontextprotocol/server-memory"] },
  "sequential-thinking": {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-sequential-thinking"]
  },
  "vercel": { "type": "http", "url": "https://mcp.vercel.com" },
  "railway": { "command": "npx", "args": ["-y", "@railway/mcp-server"] },
  "cloudflare-docs": { "type": "http", "url": "https://docs.mcp.cloudflare.com/mcp" },
  "cloudflare-workers-bindings": {
    "type": "http",
    "url": "https://bindings.mcp.cloudflare.com/mcp"
  },
  "clickhouse": { "type": "http", "url": "https://mcp.clickhouse.cloud/mcp" },
  "AbletonMCP": { "command": "uvx", "args": ["ableton-mcp"] },
  "magic": { "command": "npx", "args": ["-y", "@magicuidesign/mcp@latest"] }
}
```

这是关键——14 个已配置的 MCP，但每个项目只有约 5-6 个活跃。保持 context window 健康。

### 主要 Hook

```json
{
  "PreToolUse": [
    { "matcher": "npm|pnpm|yarn|cargo|pytest", "hooks": ["tmux reminder"] },
    { "matcher": "Write &amp;&amp; .md file", "hooks": ["block unless README/CLAUDE"] },
    { "matcher": "git push", "hooks": ["open editor for review"] }
  ],
  "PostToolUse": [
    { "matcher": "Edit &amp;&amp; .ts/.tsx/.js/.jsx", "hooks": ["prettier --write"] },
    { "matcher": "Edit &amp;&amp; .ts/.tsx", "hooks": ["tsc --noEmit"] },
    { "matcher": "Edit", "hooks": ["grep console.log warning"] }
  ],
  "Stop": [
    { "matcher": "*", "hooks": ["check modified files for console.log"] }
  ]
}
```

### 自定义 Status Line

显示用户、目录、带有脏指示器的 git 分支、剩余 context %、模型、时间和 todo 计数：

![Custom status line](../assets/images/shortform/11-statusline.jpeg)
*我在 Mac 根目录上的示例 statusline*

```
affoon:~ ctx:65% Opus 4.5 19:52
▌▌ plan mode on (shift+tab to cycle)
```

### Rules 结构

```
~/.claude/rules/
  security.md      # 强制安全检查
  coding-style.md  # 不变性，文件大小限制
  testing.md       # TDD，80% coverage
  git-workflow.md  # Conventional commit
  agents.md        # Subagent 委派规则
  patterns.md      # API 响应格式
  performance.md   # 模型选择（Haiku vs Sonnet vs Opus）
  hooks.md         # Hook 文档
```

### Subagent

```
~/.claude/agents/
  planner.md           # 功能分解
  architect.md         # 系统设计
  tdd-guide.md         # 先写测试
  code-reviewer.md     # 质量审查
  security-reviewer.md # 漏洞扫描
  build-error-resolver.md
  e2e-runner.md        # Playwright 测试
  refactor-cleaner.md  # 死代码移除
  doc-updater.md       # 保持文档同步
```

---

## 要点

1. **不要过度复杂化** - 将配置视为微调而非架构
2. **Context window 宝贵** - 禁用未使用的 MCP 和 plugin
3. **并行执行** - fork 对话，使用 git worktree
4. **自动化重复** - 用于格式化、linting、提醒的 hook
5. **范围您的 Subagent** - 有限工具 = 专注执行

---

## 参考

- [Plugin 参考](https://code.claude.com/docs/en/plugins-reference)
- [Hook 文档](https://code.claude.com/docs/en/hooks)
- [Checkpoint](https://code.claude.com/docs/en/checkpointing)
- [Interactive Mode](https://code.claude.com/docs/en/interactive-mode)
- [Memory 系统](https://code.claude.com/docs/en/memory)
- [Subagent](https://code.claude.com/docs/en/sub-agents)
- [MCP 概述](https://code.claude.com/docs/en/mcp-overview)

---

**注意：** 这是一个详细子集。有关高级模式，请参阅[详细指南](./the-longform-guide-cn.md)。

---

*在 NYC 与 [@DRodriguezFX](https://x.com/DRodriguezFX) 构建 [zenith.chat](https://zenith.chat) 赢得 Anthropic x Forum Ventures hackathon*
