# Mega Plan Repo Prompt List — 2026年3月12日

## 目的

使用这些 prompts 按仓库拆分剩余的 3月11日 mega-plan 工作。
它们是为并行 agents 编写的，并假设 3月12日 的编排和
Windows CI 通道已通过 `#417` 合并。

## 当前快照

- `everything-claude-code` 已完成编排、Codex 基线和
  Windows CI 恢复通道。
- 下一个开放的 ECC Phase 1 项目是：
  - 审查 `#399`
  - 将反复出现的讨论压力转化为跟踪的 issues
  - 定义 selective-install 架构
  - 编写 ECC 2.0 发现文档
- `agentshield`、`ECC-website` 和 `skill-creator-app` 都有未提交的
  `main` worktrees，不应直接在 `main` 上编辑。
- `applications/` 不是独立的 git 仓库。它位于父工作区仓库
  `<ECC_ROOT>` 内。

## 仓库：`everything-claude-code`

### Prompt A — PR `#399` 审查和合并准备

\`\`\`text
工作目录：<ECC_ROOT>/everything-claude-code

目标：
对照 issue #398 中描述的实际 loop 问题和
3月11日 mega plan，审查 PR #399（"fix(observe): 5层自动化 session guard 以防止
self-loop observations"）。不要假设 PR 上旧的失败 CI 仍然有意义，
因为 Windows 基线后来在 #417 中已修复。

任务：
1. 完整阅读 issue #398 和 PR #399。
2. 在本地检查 observe hook 实现和测试。
3. 确定该 PR 是否真的能防止 observer self-observation、
   automated-session observation 和 runaway recursive loops。
4. 识别任何缺失的 env-based bypass、idle gating 或 session exclusion
   行为。
5. 生成 merge recommendation，并按 severity 排序发现结果。

约束：
- 不要自动 merge。
- 不要重写不相关的 hook 行为。
- 如果进行代码更改，请将其严格限制在 observe 行为和
   测试范围内。

交付物：
- review summary
- 带有 file references 的确切发现结果
- 推荐的 merge / rework 决定
- 运行的测试命令
\`\`\`

### Prompt B — Roadmap Issues 提取

\`\`\`text
工作目录：<ECC_ROOT>/everything-claude-code

目标：
将 mega plan 中反复出现的讨论压力转化为具体的 GitHub
issues。专注于为 ECC 1.x 和 ECC 2.0 扫清障碍的
高信号 roadmap 项目。

为以下内容创建 issue drafts 或准备好发布的 issue bundle：
1. selective install profiles
2. uninstall / doctor / repair lifecycle
3. generated skill placement 和 provenance policy
4. governance past the tool call
5. ECC 2.0 discovery doc / adapter contracts

任务：
1. 阅读 3月11日 mega plan 和 3月12日 handoff。
2. 与已开放的 issues 进行去重。
3. 起草 issue titles、problem statements、scope、non-goals、acceptance
   criteria 和 file/system areas affected。

约束：
- 不要创建 filler issues。
- 优先选择 4-6 个高价值 issues，而非大量 backlog dump。
- 保持每个 issue 的范围合理，使其可以在一个 focused PR series 中完成。

交付物：
- issue shortlist
- 准备好发布的 issue bodies
- 与现有 issues 的 duplication notes
\`\`\`

### Prompt C — ECC 2.0 Discovery 和 Adapter 规范

\`\`\`text
工作目录：<ECC_ROOT>/everything-claude-code

目标：
将现有的 ECC 2.0 vision 转化为第一份具体的 discovery doc，重点是
adapter contracts、session/task state、token accounting 和 security/policy
events。

任务：
1. 使用当前的 orchestration/session snapshot code 作为 baseline。
2. 为 Claude Code、Codex、OpenCode 以及
   后续的 Cursor / GitHub App integration 定义 normalized adapter contract。
3. 为 sessions、tasks、worktrees、events、findings 和
   approvals 定义初始的 SQLite-backed data model。
4. 定义什么保留在 ECC 1.x 中，什么属于 ECC 2.0。
5. 将 unresolved product decisions 与 implementation
   requirements 分开标注。

约束：
- 将当前的 tmux/worktree/session snapshot substrate 作为
   起点，而不是 blank slate。
- 保持文档 implementation-oriented。

交付物：
- discovery doc
- adapter contract sketch
- event model sketch
- unresolved questions list
\`\`\`

## 仓库：`agentshield`

### Prompt — False Positive Audit 和 Regression Plan

\`\`\`text
工作目录：<ECC_ROOT>/agentshield

目标：
推进 mega plan 中的 AgentShield Phase 2 workstream：减少 false
positives，特别是在 declarative deny rules、block hooks、docs examples
或 config snippets 被错误分类为 executable risk 的情况下。

重要仓库状态：
- branch 当前是 main
- CLAUDE.md 和 README.md 中存在 dirty files
- 在进行 broader changes 之前，对现有 edits 进行 classify 或 park

任务：
1. 检查当前围绕以下内容的 false-positive behavior：
   - .claude hook configs
   - AGENTS.md / CLAUDE.md
   - .cursor rules
   - .opencode plugin configs
   - sample deny-list patterns
2. Separate parser behavior for declarative patterns vs executable commands。
3. Propose regression coverage additions 和 exact fixture set needed。
4. 如果 branch setup 后 safe，则 implement classifier fix 的 first pass。

约束：
- 不要直接在 dirty main 上工作
- 保持 fixes parser/classifier-scoped
- 明确 document 任何 remaining ambiguity

交付物：
- branch recommendation
- false-positive taxonomy
- proposed 或 landed regression tests
- remaining edge cases
\`\`\`

## 仓库：`ECC-website`

### Prompt — Landing Rewrite 和 Product Framing

\`\`\`text
工作目录：<ECC_ROOT>/ECC-website

目标：
Execute website lane from the mega plan by rewriting the landing/product
framing away from "config repo" and toward "open agent harness system" plus
future control-plane direction。

重要仓库状态：
- branch 当前是 main
- favicon assets 和 multiple page/component files 中存在 dirty files
- 在 meaningful work 之前 branch，并 preserve existing edits，
   除非明确 classified as stale

任务：
1. Classify the dirty main worktree state。
2. Rewrite the landing page narrative around：
   - open agent harness system
   - runtime guardrails
   - cross-harness parity
   - operator visibility and security
3. Define 或 update the next key pages：
   - /skills
   - /security
   - /platforms
   - /system 或 /dashboard
4. Keep the page visually intentional 和 product-forward，not generic SaaS。

约束：
- 不要 silently overwrite existing dirty work
- preserve existing design system where it is coherent
- distinguish ECC 1.x toolkit from ECC 2.0 control plane clearly

交付物：
- branch recommendation
- landing-page rewrite diff 或 content spec
- follow-up page map
- deployment readiness notes
\`\`\`

## 仓库：`skill-creator-app`

### Prompt — Skill Import Pipeline 和 Product Fit

\`\`\`text
工作目录：<ECC_ROOT>/skill-creator-app

目标：
Align skill-creator-app with the mega-plan external skill sourcing 和 audited
import pipeline workstream。

重要仓库状态：
- branch 当前是 main
- README.md 和 src/lib/github.ts 中存在 dirty files
- 在 broader work 之前，对现有 changes 进行 classify 或 park

任务：
1. Assess whether the app should support：
   - inventorying external skills
   - provenance tagging
   - dependency/risk audit fields
   - ECC convention adaptation workflows
2. Review the existing GitHub integration surface in src/lib/github.ts。
3. Produce concrete product/technical scope for an audited import pipeline。
4. 如果 safe after branching，land the smallest enabling changes for metadata
   capture 或 GitHub ingestion。

约束：
- 不要 turn this into a generic prompt-builder
- keep the focus on audited skill ingestion 和 ECC-compatible output

交付物：
- product-fit summary
- recommended scope for v1
- data fields / workflow steps for the import pipeline
- code changes if they are small 和 clearly justified
\`\`\`

## 仓库：`ECC` Workspace（`applications/`、`knowledge/`、`tasks/`）

### Prompt — Example Apps 和 Workflow Reliability Proofs

\`\`\`text
工作目录：<ECC_ROOT>

目标：
Use the parent ECC workspace to support the mega-plan hosted/workflow lanes。
This is not a standalone applications repo；it is the umbrella workspace that
contains applications/、knowledge/、tasks/ 和 related planning assets。

任务：
1. Inventory what in applications/ is real product code vs placeholder。
2. Identify where example repos 或 demo apps should live for：
   - GitHub App workflow proofs
   - ECC 2.0 prototype spikes
   - example install / setup reliability checks
3. Propose a clean workspace structure so product code、research、和 planning
   stop bleeding into each other。
4. Recommend which proof-of-concept should be built first。

约束：
- 不要 move large directories blindly
- distinguish repo structure recommendations from immediate code changes
- keep recommendations compatible with the current multi-repo ECC setup

交付物：
- workspace inventory
- proposed structure
- first demo/app recommendation
- follow-up branch/worktree plan
\`\`\`

## Local Continuation

当前 worktree 应继续进行 ECC-native Phase 1 工作，不要触及
这里现有的 dirty skill-file changes。最佳的下一个本地任务是：

1. selective-install architecture
2. ECC 2.0 discovery doc
3. PR `#399` review
