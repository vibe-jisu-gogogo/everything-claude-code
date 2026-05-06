# 更新日志

## 2.0.0-rc.1 - 2026-04-28

### 亮点

- 新增面向 Hermes operator 场景的 ECC 2.0 候选发布公开接口。
- 说明 ECC 是可跨 Claude Code、Codex、Cursor、OpenCode 和 Gemini 平台复用的跨框架底层基础。
- 新增经过清理的 Hermes 导入技能接口，不对外暴露私有 operator 状态。

### 发布内容

- 将 package、plugin、marketplace、OpenCode、agent 和 README 元数据更新到 `2.0.0-rc.1` 版本。
- 新增 `docs/releases/2.0.0-rc.1/` 目录，包含发布说明、社交文案草稿、发布检查清单、交接说明和演示 prompt。
- 新增 `docs/architecture/cross-harness.md` 文档，以及 ECC/Hermes 边界的回归测试覆盖。
- 目前 `ecc2/` 版本独立管理；除非发布工程团队另有决定，否则它仍为 alpha 版本的控制平面脚手架。

### 注意事项

- 这是一个候选发布版本，并不代表 ECC 2.0 完整控制平面路线图已达到 GA 标准。
- 除非发布工程团队明确选择其他方式，否则预发布版本的 npm 发布应使用 `next` 分发标签。

## 1.10.0 - 2026-04-05

### 亮点

- 经过数周的开源项目增长和积压 PR 合并，公开发布内容已与线上仓库同步。
- Operator 工作流赛道新增语音、图排序、计费、工作空间和出站技能。
- 媒体生成赛道新增以 Manim 和 Remotion 优先的发布工具。
- ECC 2.0 alpha 控制平面二进制文件现在可以从 `ecc2/` 目录本地构建，并提供了首个可用的 CLI/TUI 接口。

### 发布内容

- 将 plugin、marketplace、Codex、OpenCode 和 agent 元数据更新到 `1.10.0` 版本。
- 同步公开计数到开源线上版本：38 个 agent、156 个 skill、72 个 command。
- 更新顶层面向安装的文档和 marketplace 描述，以匹配当前仓库状态。

### 新增工作流赛道

- `brand-voice` — 基于权威来源的写作风格系统。
- `social-graph-ranker` — 加权 warm-intro 图排序原语。
- `connections-optimizer` — 基于图排序的网络修剪/添加工作流。
- `customer-billing-ops`、`google-workspace-ops`、`project-flow-ops`、`workspace-surface-audit`。
- `manim-video`、`remotion-video-creation`、`nestjs-patterns`。

### ECC 2.0 Alpha 版本

- `cargo build --manifest-path ecc2/Cargo.toml` 在仓库基线版本上构建通过。
- `ecc-tui` 目前提供 `dashboard`、`start`、`sessions`、`status`、`stop`、`resume` 和 `daemon` 命令。
- 该 alpha 版本真实可用，适合本地实验，但更广泛的控制平面路线图仍未完成，不应视为 GA 版本。

### 注意事项

- Claude 插件仍受平台级规则分发限制；选择性安装/开源路径仍是最可靠的完整安装方式。
- 本次发布是仓库层面的修正和生态同步，并不代表 ECC 2.0 完整路线图已完成。

## 1.9.0 - 2026-03-20

### 亮点

- 选择性安装架构，包含清单驱动的流水线和 SQLite 状态存储。
- 语言覆盖扩展到 10 多个生态系统，新增 6 个 agent 和语言特定规则。
- Observer 可靠性增强，新增内存节流、沙箱修复和 5 层循环防护。
- 自我改进技能基础，包含技能演化和会话适配器。

### 新增 Agents

- `typescript-reviewer` — TypeScript/JavaScript 代码评审专家 (#647)
- `pytorch-build-resolver` — PyTorch 运行时、CUDA 和训练错误解决工具 (#549)
- `java-build-resolver` — Maven/Gradle 构建错误解决工具 (#538)
- `java-reviewer` — Java 和 Spring Boot 代码评审 (#528)
- `kotlin-reviewer` — Kotlin/Android/KMP 代码评审 (#309)
- `kotlin-build-resolver` — Kotlin/Gradle 构建错误解决工具 (#309)
- `rust-reviewer` — Rust 代码评审 (#523)
- `rust-build-resolver` — Rust 构建错误解决工具 (#523)
- `docs-lookup` — 文档和 API 参考查询工具 (#529)

### 新增 Skills

- `pytorch-patterns` — PyTorch 深度学习工作流 (#550)
- `documentation-lookup` — API 参考和库文档查询 (#529)
- `bun-runtime` — Bun 运行时最佳实践 (#529)
- `nextjs-turbopack` — Next.js Turbopack 工作流 (#529)
- `mcp-server-patterns` — MCP 服务器设计模式 (#531)
- `data-scraper-agent` — AI 驱动的公共数据收集工具 (#503)
- `team-builder` — 团队组建技能 (#501)
- `ai-regression-testing` — AI 回归测试工作流 (#433)
- `claude-devfleet` — 多 agent 编排工具 (#505)
- `blueprint` — 多会话构建规划
- `everything-claude-code` — 自引用 ECC 技能 (#335)
- `prompt-optimizer` — Prompt 优化技能 (#418)
- 8 个 Evos 运营领域技能 (#290)
- 3 个 Laravel 技能 (#420)
- VideoDB 技能 (#301)

### 新增 Commands

- `/docs` — 文档查询 (#530)
- `/aside` — 侧边会话 (#407)
- `/prompt-optimize` — Prompt 优化 (#418)
- `/resume-session`、`/save-session` — 会话管理
- `learn-eval` 改进，新增基于检查清单的整体评估功能

### 新增 Rules

- Java 语言规则 (#645)
- PHP 规则包 (#389)
- Perl 语言规则和技能（模式、安全、测试）
- Kotlin/Android/KMP 规则 (#309)
- C++ 语言支持 (#539)
- Rust 语言支持 (#523)

### 基础设施

- 包含清单解析的选择性安装架构（`install-plan.js`、`install-apply.js`）(#509, #512)
- 带有查询 CLI 的 SQLite 状态存储，用于跟踪已安装组件 (#510)
- 用于结构化会话记录的会话适配器 (#511)
- 支持自我改进技能的技能演化基础 (#514)
- 带确定性评分的编排框架 (#524)
- CI 中目录计数强制校验 (#525)
- 对全部 109 个技能的安装清单进行验证 (#537)
- PowerShell 安装器包装 (#532)
- 通过 `--target antigravity` 标志支持 Antigravity IDE (#332)
- Codex CLI 自定义脚本 (#336)

### 错误修复

- 修复了 6 个文件中的 19 个 CI 测试失败问题 (#519)
- 修复了安装流水线、编排器和修复模块中的 8 个测试失败问题 (#564)
- 通过节流、重入防护和尾部采样解决了 Observer 内存暴涨问题 (#536)
- 修复了调用 Haiku 时的 Observer 沙箱访问问题 (#661)
- 修复了 Worktree 项目 ID 不匹配问题 (#665)
- 修复了 Observer 懒启动逻辑 (#508)
- 新增 Observer 5 层循环防护机制 (#399)
- 增强 Hook 可移植性，支持 Windows .cmd 文件
- Biome Hook 优化 — 消除了 npx 开销 (#359)
- InsAIts 安全 Hook 改为可选启用 (#370)
- 修复了 Windows 下 spawnSync 导出问题 (#431)
- 修复了 instinct CLI 的 UTF-8 编码问题 (#353)
- 新增 Hook 中的密钥擦除功能 (#348)

### 翻译

- 韩语（ko-KR）翻译 — README、agents、commands、skills、rules (#392)
- 中文（zh-CN）文档同步 (#428)

### 贡献者

- @ymdvsymd — observer 沙箱和 worktree 修复
- @pythonstrup — biome Hook 优化
- @Nomadu27 — InsAIts 安全 Hook
- @hahmee — 韩语翻译
- @zdocapp — 中文文档同步
- @cookiee339 — Kotlin 生态系统支持
- @pangerlkr — CI 工作流修复
- @0xrohitgarg — VideoDB 技能
- @nocodemf — Evos 运营技能
- @swarnika-cmd — 社区贡献

## 1.8.0 - 2026-03-04

### 亮点

- 以框架为核心的版本，专注于可靠性、评估规范和自主循环操作。
- Hook 运行时现在支持基于配置文件的控制和定向 Hook 禁用功能。
- NanoClaw v2 新增模型路由、技能热加载、分支、搜索、压缩、导出和指标功能。

### 核心功能

- 新增命令：`/harness-audit`、`/loop-start`、`/loop-status`、`/quality-gate`、`/model-route`。
- 新增技能：
  - `agent-harness-construction`
  - `agentic-engineering`
  - `ralphinho-rfc-pipeline`
  - `ai-first-engineering`
  - `enterprise-agent-ops`
  - `nanoclaw-repl`
  - `continuous-agent-loop`
- 新增 agents：
  - `harness-optimizer`
  - `loop-operator`

### Hook 可靠性

- 通过健壮的回退搜索修复了 SessionStart 根目录解析问题。
- 将会话摘要持久化逻辑移到 `Stop` 阶段，此时可以获取完整的会话记录 payload。
- 新增 quality-gate 和 cost-tracker Hook。
- 用专用脚本文件替换了脆弱的内联 Hook 单行命令。
- 新增 `ECC_HOOK_PROFILE` 和 `ECC_DISABLED_HOOKS` 控制参数。

### 跨平台支持

- 改进了文档警告逻辑中的 Windows 安全路径处理。
- 增强了 observer 循环行为，避免非交互模式下的挂起问题。

### 注意事项

- `autonomous-loops` 将作为兼容性别名保留一个版本；`continuous-agent-loop` 是标准名称。

### 致谢

- 灵感来自 [zarazhangrui](https://github.com/zarazhangrui)
- homunculus 设计灵感来自 [humanplane](https://github.com/humanplane)
