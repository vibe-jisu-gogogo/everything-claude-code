# Cross-Harness Architecture

ECC 是可重用的工作流层。Harnesses 是执行表面。

目标是将 agentic 工作的持久部分保存在一个仓库中：

- skills
- 规则和指令
- harness 支持的 hooks
- MCP 配置
- 安装清单
- 会话和编排模式

Claude Code、Codex、OpenCode、Cursor、Gemini 以及未来的 harnesses 应该在边缘适配这些资产，而不是为每个工具都需要新的工作流模型。

## Portability Model

| 表面 | 共享源码 | Harness Adapter | 当前状态 |
|---------|---------------|-----------------|----------------|
| Skills | `skills/*/SKILL.md` | Claude plugin、Codex plugin、`.agents/skills`、Cursor skill 副本、OpenCode plugin/config | 支持 harness 特定的打包方式 |
| Rules and instructions | `rules/`、`AGENTS.md`、翻译文档 | Claude rules 安装、Codex `AGENTS.md`、Cursor rules、OpenCode instructions | 已支持，但在各 harness 中不完全相同 |
| Hooks | `hooks/hooks.json`、`scripts/hooks/` | Claude native hooks、OpenCode plugin events、Cursor hook adapter | Claude/OpenCode/Cursor 中基于 hook；Codex 中基于指令 |
| MCPs | `.mcp.json`、`mcp-configs/` | 每个 harness 的原生 MCP 配置导入 | 在 harness 暴露 MCP 的地方支持 |
| Commands | `commands/`、CLI 脚本 | Claude slash commands、兼容性垫片、CLI 入口点 | 已支持，但命令语义各不相同 |
| Sessions | `ecc2/`、会话适配器、编排脚本 | TUI/daemon、tmux/worktree 编排、harness 特定的运行器 | Alpha 阶段 |

## What Travels Unchanged

`SKILL.md` 是最可移植的单元。

一个好的 ECC skill 应该：

- 使用带有 `name`、`description` 和 `origin` 的 YAML frontmatter
- 描述何时使用该 skill
- 说明所需的工具或连接器，不嵌入密钥
- 保持示例与仓库相对或通用
- 避免仅适用于特定 harness 的命令假设，除非该部分有明确标记

同一个源 skill 可以安装到多个 harnesses 中，因为它主要是指令、约束和工作流形态。

## What Gets Adapted

每个 harness 有不同的加载和执行行为：

- Claude Code 加载插件资产并具有原生 hook 执行。
- Codex 读取 `AGENTS.md`、插件元数据、skills 和 MCP 配置，但 hook 对等性是指令驱动的。
- OpenCode 有一个插件/事件系统，可以通过适配器层重用 ECC hook 逻辑。
- Cursor 使用其自己的规则和 hook 布局，因此 ECC 在 `.cursor/` 下维护转换后的表面。
- Gemini 支持以安装/指令为导向，应被视为兼容性表面，而不是完整的 hook 对等性。

适配器应保持精简。共享行为属于 `skills/`、`rules/`、`hooks/`、`scripts/` 和 `mcp-configs/`。

## Hermes Boundary

Hermes 不是公共 ECC 运行时。

Hermes 是一个可以消费 ECC 资产的操作 shell：

- 将选定的 ECC skills 导入 Hermes skills 目录
- 使用 ECC MCP 约定进行工具访问
- 通过可重用的 ECC 模式路由聊天、CLI、cron 和交接工作流
- 将重复的本地操作工作提炼回经过清理的 ECC skills

公共仓库应发布可重用的模式，而不是本地 Hermes 状态。

应该发布：

- 经过清理的设置文档
- 相对于仓库的演示提示
- 通用的操作 skills
- 不依赖私有凭证的示例

不应该发布：

- OAuth tokens 或 API keys
- 原始的 `~/.hermes` 导出
- 个人工作区内存
- 私有数据集
- 未经审查的仅本地自动化包

## Worked Example

使用 `skills/hermes-imports/SKILL.md` 作为跨 harnesses 的相同 skill 源。

工作流是：

1. 在 `skills/hermes-imports/SKILL.md` 中一次性编写持久行为。
2. 将 secrets、本地路径和原始操作内存排除在 skill 之外。
3. 让每个 harness 适配 skill 的加载方式。
4. 分别测试源 skill 和面向 harness 的元数据。

Claude Code 通过 Claude plugin 表面获取 skill，并可以原生强制执行相关 hooks。

Codex 读取仓库指令、`.codex-plugin/plugin.json` 和 MCP 参考配置。相同的 skill 源仍然描述工作流，但 hook 对等性是基于指令的，除非 Codex 添加原生 hook 表面。

OpenCode 通过 OpenCode package/plugin 表面获取 skill。事件处理可以通过适配器层重用 ECC hook 逻辑，而 skill 文本保持不变。

如果一个变更需要编辑同一工作流的三个 harness 副本，那么共享源放在了错误的位置。将工作流放回 `skills/`，然后仅在 harness 边缘适配加载、事件形态或命令路由。

## Today vs Later

今天已支持：

- `skills/` 中的共享 skill 源
- Claude Code 插件打包
- Codex 插件元数据和 MCP 参考配置
- OpenCode package/plugin 表面
- Cursor 适配的 rules、hooks 和 skills
- `ecc2/` 作为 alpha Rust 控制平面

仍在成熟中：

- 所有 harnesses 之间的确切 hook 对等性
- 自动 skill 同步到 Hermes
- `ecc2/` 的发布打包
- 跨 harness 会话恢复语义
- 更深层的内存和操作规划层

## Rule For New Work

添加工作流时，首先将持久行为放在 ECC 中。

仅在以下情况使用 harness 特定文件：

- 加载共享资产
- 适配事件形态
- 映射命令名称
- 处理平台限制

如果一个工作流只在一个 harness 中工作，直接记录该边界。
