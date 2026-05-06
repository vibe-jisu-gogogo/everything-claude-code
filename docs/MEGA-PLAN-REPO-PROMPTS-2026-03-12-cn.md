# Mega Plan Repo Prompt List — 2026年3月12日

## Purpose

使用这些 prompts 按仓库拆分剩余的 3 月 11 日 mega-plan 工作。它们是为并行 agents 编写的，并假设 3 月 12 日的编排和 Windows CI 通道已通过 `#417` 合并。

## Current Snapshot

- `everything-claude-code` 已完成编排、Codex 基线和 Windows CI 恢复通道。
- 接下来的 ECC Phase 1 未完成项目是：
  - 审查 `#399`
  - 将反复出现的讨论压力转化为跟踪的 issues
  - 定义 selective-install 架构
  - 编写 ECC 2.0 发现文档
- `agentshield`、`ECC-website` 和 `skill-creator-app` 的 `main` 工作树都有未提交的更改，不应直接在 `main` 上编辑。
- `applications/` 不是独立的 git 仓库。它位于父工作区仓库的 `<ECC_ROOT>` 内。

## Repo: `everything-claude-code`

### Prompt A — PR `#399` 审查和合并准备

```text
工作目录: <ECC_ROOT>/everything-claude-code

目标:
针对 issue #398 和 3 月 11 日 mega plan 中描述的实际循环问题，审查 PR #399（"fix(observe): 5 层自动化会话防护以防止自循环观察"）。不要假设 PR 上旧的失败 CI 仍然有意义，因为 Windows 基线在 #417 中稍后已修复。

任务:
1. 完整阅读 issue #398 和 PR #399。
2. 本地检查 observe hook 的实现和测试。
3. 确定该 PR 是否真正防止了 observer 自观察、自动化会话观察和失控的递归循环。
4. 识别任何缺失的基于环境的绕过、空闲门控或会话排除行为。
5. 生成合并建议，按严重性排序发现结果。

约束:
- 不要自动合并。
- 不要重写不相关的 hook 行为。
- 如果进行代码更改，保持范围严格限于 observe 行为和测试。

交付物:
- 审查摘要
- 带有文件引用的确切发现结果
- 建议的合并/返工决定
- 运行的测试命令
```

### Prompt B — 路线图 Issues 提取

```text
工作目录: <ECC_ROOT>/everything-claude-code

目标:
将 mega plan 中反复出现的讨论压力转化为具体的 GitHub issues。专注于能够为 ECC 1.x 和 ECC 2.0 解除阻塞的高信号路线图项目。

为以下内容创建 issue 草稿或准备发布的 issue 包：
1. 选择性安装配置文件
2. 卸载/诊断/修复生命周期
3. 生成的 skill 放置和来源政策
4. 工具调用之外的治理
5. ECC 2.0 发现文档/adapter 契约

任务:
1. 阅读 3 月 11 日的 mega plan 和 3 月 12 日的交接。
2. 与已开放的 issues 去重。
3. 起草 issue 标题、问题陈述、范围、非目标、验收标准和受影响的文件/系统区域。

约束:
- 不要创建填充性 issues。
- 优先选择 4-6 个高价值 issues，而不是大量的积压转储。
- 保持每个 issue 的范围，使其可以合理地在一个专注的 PR 系列中完成。

交付物:
- issue 候选清单
- 准备发布的 issue 正文
- 与现有 issues 的重复说明
```

### Prompt C — ECC 2.0 发现和 Adapter 规格

```text
工作目录: <ECC_ROOT>/everything-claude-code

目标:
将现有的 ECC 2.0 愿景转化为第一份具体的发现文档，重点关注 adapter 契约、会话/任务状态、token 计数和安全/策略事件。

任务:
1. 使用当前的编排/会话快照代码作为基线。
2. 为 Claude Code、Codex、OpenCode 以及后来的 Cursor/GitHub App 集成定义标准化的 adapter 契约。
3. 为 sessions、tasks、worktrees、events、findings 和 approvals 定义初始的 SQLite 支持的数据模型。
4. 定义什么保留在 ECC 1.x 中，什么属于 ECC 2.0。
5. 将未解决的产品决策与实现要求分开列出。

约束:
- 将当前的 tmux/worktree/session 快照底层视为起点，而不是一张白纸。
- 保持文档以实现为导向。

交付物:
- 发现文档
- adapter 契约草图
- 事件模型草图
- 未解决问题列表
```

## Repo: `agentshield`

### Prompt — 误报审计和回归计划

```text
工作目录: <ECC_ROOT>/agentshield

目标:
推进 mega plan 中的 AgentShield Phase 2 工作流：减少误报，特别是在声明性拒绝规则、block hooks、文档示例或配置片段被错误分类为可执行风险的地方。

重要仓库状态:
- 分支当前是 main
- CLAUDE.md 和 README.md 中存在未提交的文件
- 在进行更广泛的更改之前，对现有编辑进行分类或搁置

任务:
1. 检查当前围绕以下内容的误报行为：
   - .claude hook 配置
   - AGENTS.md / CLAUDE.md
   - .cursor 规则
   - .opencode 插件配置
   - 示例拒绝列表模式
2. 分离声明性模式与可执行命令的解析器行为。
3. 提出回归覆盖补充和所需的确切 fixture 集。
4. 如果分支设置后安全，实现分类器修复的第一遍。

约束:
- 不要直接在有未提交更改的 main 上工作
- 保持修复范围限于解析器/分类器
- 明确记录任何剩余的歧义

交付物:
- 分支建议
- 误报分类
- 提议或已落地的回归测试
- 剩余边缘情况
```

## Repo: `ECC-website`

### Prompt — 着陆页重写和产品框架

```text
工作目录: <ECC_ROOT>/ECC-website

目标:
执行 mega plan 中的网站通道，将着陆/产品框架从"配置仓库"重写为"开放 agent harness 系统"以及未来的控制平面方向。

重要仓库状态:
- 分支当前是 main
- favicon 资产和多个页面/组件文件中存在未提交的更改
- 在有意义的工作之前创建分支，保留现有编辑除非明确分类为过时

任务:
1. 对 main 工作树的未提交状态进行分类。
2. 围绕以下内容重写着陆页叙述：
   - 开放 agent harness 系统
   - 运行时防护
   - 跨 harness parity
   - 操作员可见性和安全性
3. 定义或更新下一个关键页面：
   - /skills
   - /security
   - /platforms
   - /system 或 /dashboard
4. 保持页面视觉上有意且以产品为导向，而不是通用的 SaaS。

约束:
- 不要静默覆盖现有的未提交工作
- 在现有设计系统连贯的地方保留它
- 清楚地区分 ECC 1.x 工具包与 ECC 2.0 控制平面

交付物:
- 分支建议
- 着陆页重写 diff 或内容规范
- 后续页面地图
- 部署准备说明
```

## Repo: `skill-creator-app`

### Prompt — Skill 导入管道和产品适配

```text
工作目录: <ECC_ROOT>/skill-creator-app

目标:
使 skill-creator-app 与 mega plan 的外部 skill 来源和经过审计的导入管道工作流对齐。

重要仓库状态:
- 分支当前是 main
- README.md 和 src/lib/github.ts 中存在未提交的文件
- 在进行更广泛的工作之前对现有更改进行分类或搁置

任务:
1. 评估应用是否应支持：
   - 盘点外部 skills
   - 来源标记
   - 依赖/风险审计字段
   - ECC 约定适配工作流
2. 查看 src/lib/github.ts 中现有的 GitHub 集成表面。
3. 为经过审计的导入管道生成具体的产品/技术范围。
4. 如果分支后安全，落地用于元数据捕获或 GitHub 摄取的最小启用更改。

约束:
- 不要将其变成通用的 prompt 构建器
- 保持专注于经过审计的 skill 摄取和 ECC 兼容输出

交付物:
- 产品适配摘要
- v1 的建议范围
- 导入管道的数据字段/工作流步骤
- 如果更改小且理由充分则进行代码更改
```

## Repo: `ECC` Workspace (`applications/`, `knowledge/`, `tasks/`)

### Prompt — 示例应用和工作流可靠性证明

```text
工作目录: <ECC_ROOT>

目标:
使用父 ECC 工作区支持 mega plan 的托管/工作流通道。这不是独立的应用仓库；它是包含 applications/、knowledge/、tasks/ 和相关规划资产的伞形工作区。

任务:
1. 盘点 applications/ 中什么是真实产品代码与占位符。
2. 确定示例仓库或演示应用应位于何处，用于：
   - GitHub App 工作流证明
   - ECC 2.0 原型尖峰
   - 示例安装/设置可靠性检查
3. 提出一个干净的工作区结构，使产品代码、研究和规划不再相互渗透。
4. 推荐应首先构建哪个概念验证。

约束:
- 不要盲目移动大型目录
- 区分仓库结构建议与即时代码更改
- 保持建议与当前的多仓库 ECC 设置兼容

交付物:
- 工作区清单
- 提议的结构
- 第一个演示/应用建议
- 后续分支/工作树计划
```

## Local Continuation

当前的工作树应保留在不触及此处现有未提交的 skill 文件更改的 ECC 原生 Phase 1 工作上。接下来最好的本地任务是：

1. selective-install 架构
2. ECC 2.0 发现文档
3. PR `#399` 审查
