# Mega Plan Repo Prompt List — 2026年3月12日

## 目的

使用这些 prompts 按仓库拆分剩余的3月11日 mega-plan 工作。
它们是为 parallel agents 编写的，假设3月12日的 orchestration 和
Windows CI lane 已通过 `#417` 合并。

## 当前快照

- `everything-claude-code` 已完成 orchestration、Codex baseline 和
  Windows CI recovery lane。
- 接下来的 ECC 第一阶段开放项目是：
  - review `#399`
  - 将反复出现的 discussion pressure 转化为 tracked issues
  - 定义 selective-install architecture
  - 编写 ECC 2.0 discovery doc
- `agentshield`、`ECC-website` 和 `skill-creator-app` 都有 dirty 的
  `main` worktrees，不应直接在 `main` 上编辑。
- `applications/` 不是独立的 git repo。它位于 parent workspace repo
  的 `<ECC_ROOT>` 内。

## 仓库：`everything-claude-code`

### Prompt A — PR `#399` 审查和合并准备

```text
工作目录：<ECC_ROOT>/everything-claude-code

目标：
对照 issue #398 和3月11日 mega plan 中描述的实际 loop problem，
review PR #399（"fix(observe): 5层 automated session guard 以防止自循环观察"）。
不要假设 PR 上旧的 failing CI 仍然有意义，因为 Windows baseline 在
#417 中稍后修复了。

任务：
1. 完整阅读 issue #398 和 PR #399。
2. 在本地 inspect observe hook implementation 和 tests。
3. 确定 PR 是否真正防止了 observer self-observation、
   automated-session observation 和 runaway recursive loops。
4. 识别任何缺失的 env-based bypass、idle gating 或 session exclusion behavior。
5. 生成 merge recommendation，按 severity 排序 findings。

约束：
- 不要 merge automatically。
- 不要 rewrite unrelated hook behavior。
- 如果进行 code changes，保持它们严格限定在 observe behavior
   和 tests 范围内。

交付物：
- review summary
- 带有 file references 的确切 findings
- 推荐的 merge / rework decision
- 运行的 test commands
```

### Prompt B — 路线图 Issues 提取

```text
工作目录：<ECC_ROOT>/everything-claude-code

目标：
将 mega plan 中反复出现的 discussion pressure 转化为具体的 GitHub
issues。专注于为 ECC 1.x 和 ECC 2.0 扫除障碍的 high-signal roadmap items。

为以下内容创建 issue drafts 或 ready-to-post issue bundle：
1. selective install profiles
2. uninstall / doctor / repair lifecycle
3. generated skill placement 和 provenance policy
4. governance past the tool call
5. ECC 2.0 discovery doc / adapter contracts

任务：
1. 阅读3月11日的 mega plan 和3月12日的 handoff。
2. 与 already-open issues 进行 deduplicate。
3. Draft issue titles、problem statements、scope、non-goals、acceptance
   criteria 以及受影响的 file/system areas。

约束：
- 不要创建 filler issues。
- 优先选择4-6个 high-value issues，而不是大量 backlog dump。
- 保持每个 issue 的 scope，使其可以通过一系列 focused PR
   合理完成。

交付物：
- issue shortlist
- ready-to-post issue bodies
- 与 existing issues 的 duplication notes
```

### Prompt C — ECC 2.0 Discovery 和 Adapter 规格

```text
工作目录：<ECC_ROOT>/everything-claude-code

目标：
将现有的 ECC 2.0 vision 转化为第一份具体的 discovery doc，重点关注
adapter contracts、session/task state、token accounting 和 security/policy events。

任务：
1. 使用当前的 orchestration/session snapshot code 作为 baseline。
2. 为 Claude Code、Codex、OpenCode 以及
   后来的 Cursor / GitHub App integration 定义 normalized adapter contract。
3. 为 sessions、tasks、worktrees、events、findings 和 approvals
   定义 initial SQLite-backed data model。
4. 定义哪些内容保留在 ECC 1.x 中，哪些属于 ECC 2.0。
5. 将 unresolved product decisions 与 implementation
   requirements 分开列出。

约束：
- 将当前的 tmux/worktree/session snapshot substrate 作为
   起点，而不是 blank slate。
- 保持 doc implementation-oriented。

交付物：
- discovery doc
- adapter contract sketch
- event model sketch
- unresolved questions list
```

## 仓库：`agentshield`

### Prompt — False Positive Audit 和 Regression Plan

```text
工作目录：<ECC_ROOT>/agentshield

目标：
推进 mega plan 中的 AgentShield Phase 2 workstream：减少 false positives，
特别是在 declarative deny rules、block hooks、docs examples 或 config snippets
被错误分类为 executable risk 的情况下。

重要仓库状态：
- 当前 branch 是 main
- CLAUDE.md 和 README.md 存在 dirty files
- 在进行 broader changes 之前，对 existing edits 进行 classify 或 park

任务：
1. Inspect 当前围绕以下内容的 false-positive behavior：
   - .claude hook configs
   - AGENTS.md / CLAUDE.md
   - .cursor rules
   - .opencode plugin configs
   - sample deny-list patterns
2. Separate parser behavior for declarative patterns vs executable commands。
3. Propose regression coverage additions 和所需的确切 fixture set。
4. 如果在 branch setup 后 safe，implement classifier fix 的 first pass。

约束：
- 不要直接在 dirty main 上工作
- 保持 fixes parser/classifier-scoped
- 明确 document any remaining ambiguity

交付物：
- branch recommendation
- false-positive taxonomy
- proposed 或 landed regression tests
- remaining edge cases
```

## 仓库：`ECC-website`

### Prompt — Landing Rewrite 和 Product Framing

```text
工作目录：<ECC_ROOT>/ECC-website

目标：
通过 rewriting landing/product framing 来执行 mega plan 中的 website lane，
从"config repo"转向"open agent harness system"加上 future control-plane direction。

重要仓库状态：
- 当前 branch 是 main
- favicon assets 和多个 page/component files 存在 dirty files
- 在 meaningful work 之前 branch，保留 existing edits，除非
   明确 classified 为 stale

任务：
1. Classify dirty main worktree state。
2. Rewrite landing page narrative，围绕：
   - open agent harness system
   - runtime guardrails
   - cross-harness parity
   - operator visibility 和 security
3. Define 或 update 下一个 key pages：
   - /skills
   - /security
   - /platforms
   - /system 或 /dashboard
4. 保持 page visually intentional 和 product-forward，不是 generic SaaS。

约束：
- 不要 silently overwrite existing dirty work
- 在现有 design system 连贯的地方保留它
- 清楚区分 ECC 1.x toolkit 与 ECC 2.0 control plane

交付物：
- branch recommendation
- landing-page rewrite diff 或 content spec
- follow-up page map
- deployment readiness notes
```

## 仓库：`skill-creator-app`

### Prompt — Skill Import Pipeline 和 Product Fit

```text
工作目录：<ECC_ROOT>/skill-creator-app

目标：
使 skill-creator-app 与 mega plan 的 external skill sourcing 和 audited
import pipeline workstream 保持一致。

重要仓库状态：
- 当前 branch 是 main
- README.md 和 src/lib/github.ts 存在 dirty files
- 在 broader work 之前对 existing changes 进行 classify 或 park

任务：
1. Assess app 是否应 support：
   - inventorying external skills
   - provenance tagging
   - dependency/risk audit fields
   - ECC convention adaptation workflows
2. Review src/lib/github.ts 中现有的 GitHub integration surface。
3. Produce concrete product/technical scope for audited import pipeline。
4. 如果在 branching 后 safe，land 最小的 enabling changes 用于 metadata
   capture 或 GitHub ingestion。

约束：
- 不要将其变成 generic prompt-builder
- 保持 focus on audited skill ingestion 和 ECC-compatible output

交付物：
- product-fit summary
- recommended scope for v1
- data fields / workflow steps for import pipeline
- code changes 如果它们 small 且 clearly justified
```

## 仓库：`ECC` Workspace (`applications/`、`knowledge/`、`tasks/`)

### Prompt — Example Apps 和 Workflow Reliability Proofs

```text
工作目录：<ECC_ROOT>

目标：
使用 parent ECC workspace 支持 mega plan 的 hosted/workflow lanes。
这不是独立的 applications repo；它是包含
applications/、knowledge/、tasks/ 和相关 planning assets 的 umbrella workspace。

任务：
1. Inventory applications/ 中哪些是 real product code，哪些是 placeholder。
2. Identify example repos 或 demo apps 应该放置在哪里，用于：
   - GitHub App workflow proofs
   - ECC 2.0 prototype spikes
   - example install / setup reliability checks
3. Propose clean workspace structure，使 product code、research 和 planning
   不再相互渗透。
4. Recommend 应该首先 build 哪个 proof-of-concept。

约束：
- 不要 blindly move large directories
- 区分 repo structure recommendations 与 immediate code changes
- 保持 recommendations 与 current multi-repo ECC setup 兼容

交付物：
- workspace inventory
- proposed structure
- first demo/app recommendation
- follow-up branch/worktree plan
```

## 本地继续

Current worktree 应保持在 ECC-native Phase 1 work，不接触
此处现有的 dirty skill-file changes。最佳的下一个 local tasks 是：

1. selective-install architecture
2. ECC 2.0 discovery doc
3. PR `#399` review
