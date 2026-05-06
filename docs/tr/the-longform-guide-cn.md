# Claude Code 全攻略：长篇指南

![Header: The Longform Guide to Everything Claude Code](../assets/images/longform/01-header.png)

---

> **前置条件**: 本指南建立在 [Claude Code 全攻略：短篇指南](./the-shortform-guide.md) 的基础之上。如果您尚未安装 skills、hooks、subagents、MCPs 和 plugins，请先阅读该指南。

![Reference to Shorthand Guide](../assets/images/longform/02-shortform-reference.png)
*短篇指南 - 请先阅读它*

在短篇指南中，我介绍了基本安装：构成高效 Claude Code 工作流核心的 skills 和 commands、hooks、subagents、MCPs、plugins 以及配置模式。那是安装指南和基础架构。

本长篇指南将深入探讨区分高效会话与浪费会话的技术。如果您还没有阅读短篇指南，请返回并先完成配置。本文后续内容假设 skills、agents、hooks 和 MCPs 已经配置完成并正常运行。

本文主题包括：token 经济、memory 持久性、验证模式、并行化策略以及创建可重用工作流的复合效应。这些是我在 10 多个月的日常使用中总结出的模式，决定了是在头一小时内就受 context 衰减困扰，还是能持续数小时的高效会话。

短篇和长篇指南中涵盖的所有内容都可在 GitHub 上获取：`github.com/affaan-m/everything-claude-code`

---

## 技巧与窍门

### 部分 MCP 可被替代并能释放您的 Context Window

对于版本控制（GitHub）、数据库（Supabase）、部署（Vercel、Railway）等 MCP - 这些平台中的大多数已经拥有 MCP 本质上只是封装的强大 CLI。MCP 是一个很好的封装，但它有代价。

为了让 CLI 在不实际使用 MCP（以及随之而来的减少的 context window）的情况下更像 MCP 运行，请考虑将功能打包到 skills 和 commands 中。提取 MCP 暴露的简化工作的工具，并将它们转换为 commands。

示例：不要始终保持 GitHub MCP 加载状态，而是创建一个使用您首选选项封装 `gh pr create` 的 `/gh-pr` command。不要让 Supabase MCP 消耗 context，而是创建直接使用 Supabase CLI 的 skills。

通过延迟加载，context window 问题大多已解决。但 token 使用和成本并未同样解决。CLI + skills 方法仍然是一种 token 优化方式。

---

## 重要事项

### Context 和 Memory 管理

对于跨会话的 memory 共享，最佳方式是使用 skill 或 command 总结和跟踪进度，然后保存到 `.claude` 文件夹中的 `.tmp` 文件，并在会话结束前持续追加。第二天您可以将其作为 context 使用并从中断处继续，为每个会话创建新文件，以免旧 context 污染新工作。

![Session Storage File Tree](../assets/images/longform/03-session-storage.png)
*会话存储示例 -> <https://github.com/affaan-m/everything-claude-code/tree/main/examples/sessions>*

Claude 创建一个总结当前状态的文件。审阅它，根据需要请求编辑，然后重新开始。对于新对话，只需提供文件路径。在超过 context 限制和需要继续复杂工作时特别有用。这些文件应包含：
- 哪些方法有效（可通过证据验证）
- 哪些方法已尝试但无效
- 哪些方法尚未尝试以及下一步该做什么

**策略性清理 Context：**

当您的计划就绪且 context 已清理（现在 Claude Code 中在计划模式下是默认选项），您可以从计划开始工作。当您积累了大量与执行不再相关的探索 context 时，这很有用。对于策略性压缩，禁用自动压缩。在逻辑间隔手动压缩，或创建 skill 为您完成。

**高级：动态 System Prompt 注入**

我采用的一种模式：不是将所有内容都放在加载每个会话的 CLAUDE.md（用户范围）或 `.claude/rules/`（项目范围）中，而是使用 CLI flags 动态注入 context。

```bash
claude --system-prompt "$(cat memory.md)"
```

这使您可以更精确地控制何时加载哪些 context。System prompt 内容具有比用户消息更高的权限，用户消息又比工具结果具有更高的权限。

**实际设置：**

```bash
# 日常开发
alias claude-dev='claude --system-prompt "$(cat ~/.claude/contexts/dev.md)"'

# PR 审查模式
alias claude-review='claude --system-prompt "$(cat ~/.claude/contexts/review.md)"'

# 研究/探索模式
alias claude-research='claude --system-prompt "$(cat ~/.claude/contexts/research.md)"'
```

**高级：Memory Persistence Hooks**

大多数人不知道有与 memory 相关的 hooks：

- **PreCompact Hook**：在 context 压缩发生之前，将重要状态保存到文件
- **Stop Hook（会话结束）**：在会话结束时，将学习内容持久化到文件
- **SessionStart Hook**：在新会话中，自动加载先前的 context

我已创建这些 hooks，它们位于仓库中：`github.com/affaan-m/everything-claude-code/tree/main/hooks/memory-persistence`

---

### 持续学习 / Memory

当您发现自己不得不多次重复相同的 prompt，而 Claude 陷入相同问题或给出您以前听过的答案时 - 这些模式应该被添加为 skills。

**问题：** 浪费的 tokens、浪费的 context、浪费的时间。

**解决方案：** 当 Claude Code 发现重要内容时 - 调试技术、解决方法、项目特定模式 - 将此信息保存为新 skill。下次出现类似问题时，该 skill 会自动加载。

我创建了一个实现此功能的持续学习 skill：`github.com/affaan-m/everything-claude-code/tree/main/skills/continuous-learning`

**为什么选择 Stop Hook（而非 UserPromptSubmit）：**

关键设计决策是使用 **Stop hook** 而非 UserPromptSubmit。UserPromptSubmit 在每条消息上运行 - 为每个 prompt 添加延迟。Stop 在会话结束时运行一次 - 轻量级，不会在会话期间减慢您的速度。

---

### Token 优化

**主要策略：Subagent 架构**

优化您使用的工具，并设计 subagent 架构以委派适合任务的最便宜模型。

**模型选择快速参考：**

![Model Selection Table](../assets/images/longform/04-model-selection.png)
*各种常见任务的 subagent 假设设置以及选择背后的推理*

| 任务类型                      | 模型   | 原因                                             |
| ----------------------------- | ------ | ------------------------------------------------ |
| 探索/搜索                     | Haiku  | 快速，便宜，足够擅长查找文件                     |
| 简单编辑                      | Haiku  | 单文件更改，明确指令                             |
| 多文件实现                    | Sonnet | 编码的最佳平衡                                   |
| 复杂架构                      | Opus   | 需要深度推理                                     |
| PR 审查                       | Sonnet | 理解 context，捕捉细微差别                        |
| 安全分析                      | Opus   | 不能错过安全漏洞                                 |
| 文档写作                      | Haiku  | 结构简单                                         |
| 复杂 bug 调试                 | Opus   | 需要在脑海中保持整个系统                         |

将 Sonnet 设为 90% 编码任务的默认值。当第一次尝试失败、任务跨越 5+ 个文件、需要架构决策或安全关键代码时，升级到 Opus。

**定价参考：**

![Claude Model Pricing](../assets/images/longform/05-pricing-table.png)
*来源: <https://platform.claude.com/docs/en/about-claude/pricing>*

**特定工具优化：**

用 mgrep 替换 grep - 与传统 grep 或 ripgrep 相比，平均减少约 50% 的 tokens：

![mgrep Benchmark](../assets/images/longform/06-mgrep-benchmark.png)
*在我们的 50 任务基准测试中，mgrep + Claude Code 使用的 tokens 比基于 grep 的工作流少约 2 倍，同时保持相似或更好的评估质量。来源: mgrep by @mixedbread-ai*

**模块化代码库的好处：**

拥有更模块化的代码库，其中主要文件是数百行而非数千行，有助于 token 优化成本和首次就正确完成任务。

---

### 验证循环和 Evals

**基准测试工作流：**

比较使用 skill 和不使用 skill 请求相同事物并检查输出差异：

分叉对话，在其中一个中不使用 skill 启动新的 worktree，最后取 diff，看看记录了什么。

**Eval 模式类型：**

- **基于 Checkpoint 的 Evals**：设置明确的 checkpoints，根据定义的标准验证，在继续之前修复
- **持续 Evals**：每 N 分钟或重大更改后运行，完整测试套件 + lint

**关键指标：**

```
pass@k: k 次尝试中至少有一次成功
        k=1: 70%  k=3: 91%  k=5: 97%

pass^k: 所有 k 次尝试都必须成功
        k=1: 70%  k=3: 34%  k=5: 17%
```

当只需要它工作时使用 **pass@k**。当需要一致性时使用 **pass^k**。

---

## 并行化

在多 Claude 终端设置中分叉对话时，确保分叉和原始对话中操作的范围得到明确定义。涉及代码更改时，目标是最小重叠。

**我首选的模式：**

主对话用于代码更改，分叉用于有关代码库和当前状态的问题或外部服务的研究。

**关于任意终端数量：**

![Boris on Parallel Terminals](../assets/images/longform/07-boris-parallel.png)
*Boris (Anthropic) 关于运行多个 Claude 实例的观点*

Boris 有关于并行化的提示。他建议了诸如在本地运行 5 个 Claude 实例和在上游运行 5 个实例之类的事情。我建议反对设置任意数量的终端。添加终端应该源于真正的必要性。

您的目标应该是：**使用最少可行的并行化量能完成多少工作。**

**并行实例的 Git Worktrees：**

```bash
# 为并行工作创建 worktrees
git worktree add ../project-feature-a feature-a
git worktree add ../project-feature-b feature-b
git worktree add ../project-refactor refactor-branch

# 每个 worktree 获得自己的 Claude 实例
cd ../project-feature-a && claude
```

如果您开始扩展实例数量，并且有多个 Claude 实例在重叠的代码上工作，那么必须使用 git worktrees 并为每个实例有一个明确定义的计划。使用 `/rename <name here>` 命名所有对话。

![Two Terminal Setup](../assets/images/longform/08-two-terminals.png)
*初始设置：左侧终端用于编码，右侧终端用于提问 - 使用 /rename 和 /fork*

**级联方法：**

运行多个 Claude Code 实例时，使用"级联"模式组织：

- 在右侧的新标签页中打开新任务
- 从左到右扫描，从最旧到最新
- 同时最多专注于 3-4 个任务

---

## 核心事务

**双实例启动模式：**

对于我自己的工作流管理，我喜欢以 2 个打开的 Claude 实例启动一个空仓库。

**实例 1：脚手架 Agent**
- 搭建脚手架和基础
- 创建项目结构
- 安装配置（CLAUDE.md、rules、agents）

**实例 2：深度研究 Agent**
- 连接所有服务，网页搜索
- 创建详细的 PRD
- 创建架构 mermaid 图
- 用实际文档片段编译参考

**llms.txt 模式：**

如果可用，在访问文档页面后对它们执行 `/llms.txt` 可以在许多文档参考中找到 `llms.txt`。这为您提供了文档的干净、LLM 优化版本。

**哲学：创建可重用模式**

来自 @omarsar0："早期，我花时间创建可重用的工作流/模式。构建起来很乏味，但随着模型和 agent harness 的改进，这产生了疯狂的复合效应。"

**要投资的事项：**

- Subagent 们
- Skill 们
- Command 们
- 规划模式
- MCP 工具
- Context 工程模式

---

## Agent 和 Sub-Agent 的最佳实践

**Sub-Agent Context 问题：**

Sub-agent 的存在是为了通过返回摘要而非转储所有内容来节省 context。但是 orchestrator 具有 sub-agent 缺乏的语义 context。Sub-agent 只知道实际查询，而不知道请求背后的意图。

**迭代检索模式：**

1. Orchestrator 评估每个 sub-agent 返回
2. 在接受之前提出后续问题
3. Sub-agent 返回来源，获取答案，返回
4. 循环直到足够（最多 3 个循环）

**关键：** 不仅传递查询，还要传递意图 context。

**有序阶段的 Orchestrator：**

```markdown
阶段 1：研究（使用 Explore agent）→ research-summary.md
阶段 2：计划（使用 planner agent）→ plan.md
阶段 3：实现（使用 tdd-guide agent）→ 代码更改
阶段 4：审查（使用 code-reviewer agent）→ review-comments.md
阶段 5：验证（必要时使用 build-error-resolver）→ 完成或回溯
```

**关键规则：**

1. 每个 agent 接收一个明确的输入并产生一个明确的输出
2. 输出成为下一阶段的输入
3. 永远不要跳过阶段
4. 在 agents 之间使用 `/clear`
5. 将中间输出存储在文件中

---

## 有趣的事情 / 非关键但有趣的提示

### 自定义 Status Line

您可以使用 `/statusline` 进行设置 - 然后 Claude 会说没有但可以为您设置并询问您想要什么。

另请参阅：ccstatusline（自定义 Claude Code status line 的社区项目）

### 语音转录

用您的声音与 Claude Code 交谈。对许多人来说比打字更快。

- 在 Mac 上使用 superwhisper、MacWhisper
- 即使有转录错误，Claude 也能理解意图

### 终端 Alias

```bash
alias c='claude'
alias gb='github'
alias co='code'
alias q='cd ~/Desktop/projects'
```

---

## 里程碑

![25k+ GitHub Stars](../assets/images/longform/09-25k-stars.png)
*不到一周达到 25,000+ GitHub stars*

---

## 资源

**Agent 编排：**

- claude-flow — 具有 54+ 个专用 agent 的社区构建的企业编排平台

**自进化 Memory：**

- 查看此仓库中的 `skills/continuous-learning/`
- rlancemartin.github.io/2025/12/01/claude_diary/ - 会话反射模式

**System Prompt 参考：**

- system-prompts-and-models-of-ai-tools — AI system prompt 的社区集合（110k+ stars）

**官方：**

- Anthropic Academy: anthropic.skilljar.com

---

## 参考

- [Anthropic: 为 AI agent 揭开 evals 的神秘面纱](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents)
- [YK: 32 个 Claude Code 技巧](https://agenticcoding.substack.com/p/32-claude-code-tips-from-basics-to)
- [RLanceMartin: 会话反射模式](https://rlancemartin.github.io/2025/12/01/claude_diary/)
- @PerceptualPeak: Sub-Agent Context Negotiation
- @menhguin: Agent Abstractions Tier List
- @omarsar0: Compound Effects Philosophy

---

*这两份指南中涵盖的所有内容都可在 GitHub 上的 [everything-claude-code](https://github.com/affaan-m/everything-claude-code) 获得*
