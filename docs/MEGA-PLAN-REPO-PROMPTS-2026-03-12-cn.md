# Mega Plan Repo Prompt List — 2026年3月12日

## Purpose

使用这些prompts按仓库拆分3月11日剩余的mega-plan工作。
它们是为parallel agents编写的，假设3月12日的orchestration和
Windows CI lane已通过`#417`合并。

## Current Snapshot

- `everything-claude-code`已完成orchestration、Codex baseline和
  Windows CI recovery lane。
- 下一个开放的ECC Phase 1项目是：
  - review `#399`
  - 将反复出现的discussion pressure转化为tracked issues
  - 定义selective-install architecture
  - 编写ECC 2.0 discovery doc
- `agentshield`、`ECC-website`和`skill-creator-app`都有dirty的
  `main` worktrees，不应直接在`main`上编辑。
- `applications/`不是独立的git repo。它位于parent workspace repo
  的`<ECC_ROOT>`内。

## Repo: `everything-claude-code`

### Prompt A — PR `#399` Review and Merge Readiness

```text
Work in: <ECC_ROOT>/everything-claude-code

Goal:
针对issue #398和3月11日mega plan中描述的实际loop problem，
review PR #399（"fix(observe): 5-layer automated session guard to prevent
self-loop observations"）。不要假设PR上旧的failing CI仍然有意义，
因为Windows baseline已在#417中修复。

Tasks:
1. 完整阅读issue #398和PR #399。
2. 在本地inspect observe hook implementation和tests。
3. 确定该PR是否真正防止了observer self-observation、
   automated-session observation和runaway recursive loops。
4. 识别任何缺失的env-based bypass、idle gating或session exclusion
   behavior。
5. 生成merge recommendation，按severity排序findings。

Constraints:
- 不要merge automatically。
- 不要rewrite unrelated hook behavior。
- 如果进行code changes，保持它们严格限定于
  observe behavior和tests。

Deliverables:
- review summary
- 带file references的确切findings
- 推荐的merge / rework decision
- 运行的test commands
```

### Prompt B — Roadmap Issues Extraction

```text
Work in: <ECC_ROOT>/everything-claude-code

Goal:
将mega plan中反复出现的discussion pressure转化为具体的GitHub
issues。专注于能为ECC 1.x和ECC 2.0扫清障碍的high-signal roadmap items。

为以下内容创建issue drafts或ready-to-post issue bundle：
1. selective install profiles
2. uninstall / doctor / repair lifecycle
3. generated skill placement和provenance policy
4. governance past the tool call
5. ECC 2.0 discovery doc / adapter contracts

Tasks:
1. 阅读3月11日的mega plan和3月12日的handoff。
2. 与already-open issues进行deduplicate。
3. Draft issue titles、problem statements、scope、non-goals、acceptance
   criteria以及受影响的file/system areas。

Constraints:
- 不要创建filler issues。
- 优先选择4-6个high-value issues，而不是大量的backlog dump。
- 保持每个issue的scope，使其可以通过一系列focused PR
  合理完成。

Deliverables:
- issue shortlist
- ready-to-post issue bodies
- 与existing issues的duplication notes
```

### Prompt C — ECC 2.0 Discovery and Adapter Spec

```text
Work in: <ECC_ROOT>/everything-claude-code

Goal:
将现有的ECC 2.0 vision转化为第一个具体的discovery doc，专注于
adapter contracts、session/task state、token accounting和security/policy
events。

Tasks:
1. 使用当前的orchestration/session snapshot code作为baseline。
2. 为Claude Code、Codex、OpenCode以及
   后来的Cursor / GitHub App integration定义normalized adapter contract。
3. 为sessions、tasks、worktrees、events、findings和approvals
   定义initial SQLite-backed data model。
4. 定义哪些内容保留在ECC 1.x中，哪些属于ECC 2.0。
5. 将unresolved product decisions与implementation
   requirements分开列出。

Constraints:
- 将当前的tmux/worktree/session snapshot substrate作为
   起点，而不是blank slate。
- 保持doc implementation-oriented。

Deliverables:
- discovery doc
- adapter contract sketch
- event model sketch
- unresolved questions list
```

## Repo: `agentshield`

### Prompt — False Positive Audit and Regression Plan

```text
Work in: <ECC_ROOT>/agentshield

Goal:
推进mega plan中的AgentShield Phase 2 workstream：减少false
positives，特别是在declarative deny rules、block hooks、docs examples
或config snippets被错误分类为executable risk的情况下。

Important repo state:
- 当前branch是main
- CLAUDE.md和README.md存在dirty files
- 在进行broader changes之前，对existing edits进行classify或park

Tasks:
1. Inspect当前围绕以下内容的false-positive behavior：
   - .claude hook configs
   - AGENTS.md / CLAUDE.md
   - .cursor rules
   - .opencode plugin configs
   - sample deny-list patterns
2. Separate parser behavior for declarative patterns vs executable commands。
3. Propose regression coverage additions和所需的确切fixture set。
4. 如果在branch setup后safe，implement classifier fix的first pass。

Constraints:
- 不要直接在dirty main上工作
- 保持fixes parser/classifier-scoped
- 明确document any remaining ambiguity

Deliverables:
- branch recommendation
- false-positive taxonomy
- proposed或landed regression tests
- remaining edge cases
```

## Repo: `ECC-website`

### Prompt — Landing Rewrite and Product Framing

```text
Work in: <ECC_ROOT>/ECC-website

Goal:
通过rewriting landing/product framing来执行mega plan中的website lane，
从"config repo"转向"open agent harness system"加上
future control-plane direction。

Important repo state:
- 当前branch是main
- favicon assets和多个page/component files存在dirty files
- 在meaningful work之前branch，保留existing edits，除非
  明确classified为stale

Tasks:
1. Classify dirty main worktree state。
2. Rewrite landing page narrative，围绕：
   - open agent harness system
   - runtime guardrails
   - cross-harness parity
   - operator visibility和security
3. Define或update下一个key pages：
   - /skills
   - /security
   - /platforms
   - /system或/dashboard
4. 保持page visually intentional和product-forward，不是generic SaaS。

Constraints:
- 不要silently overwrite existing dirty work
- 在现有design system连贯的地方保留它
- 清楚区分ECC 1.x toolkit与ECC 2.0 control plane

Deliverables:
- branch recommendation
- landing-page rewrite diff或content spec
- follow-up page map
- deployment readiness notes
```

## Repo: `skill-creator-app`

### Prompt — Skill Import Pipeline and Product Fit

```text
Work in: <ECC_ROOT>/skill-creator-app

Goal:
使skill-creator-app与mega plan的external skill sourcing和audited
import pipeline workstream保持一致。

Important repo state:
- 当前branch是main
- README.md和src/lib/github.ts存在dirty files
- 在broader work之前对existing changes进行classify或park

Tasks:
1. Assess app是否应support：
   - inventorying external skills
   - provenance tagging
   - dependency/risk audit fields
   - ECC convention adaptation workflows
2. Review src/lib/github.ts中现有的GitHub integration surface。
3. Produce concrete product/technical scope for audited import pipeline。
4. 如果在branching后safe，land最小的enabling changes用于metadata
   capture或GitHub ingestion。

Constraints:
- 不要将其变成generic prompt-builder
- 保持focus on audited skill ingestion和ECC-compatible output

Deliverables:
- product-fit summary
- recommended scope for v1
- data fields / workflow steps for import pipeline
- code changes如果它们small且clearly justified
```

## Repo: `ECC` Workspace (`applications/`, `knowledge/`, `tasks/`)

### Prompt — Example Apps and Workflow Reliability Proofs

```text
Work in: <ECC_ROOT>

Goal:
使用parent ECC workspace支持mega plan的hosted/workflow lanes。
这不是独立的applications repo；它是包含
applications/、knowledge/、tasks/和相关planning assets的umbrella workspace。

Tasks:
1. Inventory applications/中哪些是real product code，哪些是placeholder。
2. Identify example repos或demo apps应该放置在哪里，用于：
   - GitHub App workflow proofs
   - ECC 2.0 prototype spikes
   - example install / setup reliability checks
3. Propose clean workspace structure，使product code、research和planning
   不再相互渗透。
4. Recommend应该首先build哪个proof-of-concept。

Constraints:
- 不要blindly move large directories
- 区分repo structure recommendations与immediate code changes
- 保持recommendations与current multi-repo ECC setup兼容

Deliverables:
- workspace inventory
- proposed structure
- first demo/app recommendation
- follow-up branch/worktree plan
```

## Local Continuation

Current worktree应保持在ECC-native Phase 1 work，不接触
此处现有的dirty skill-file changes。最佳的下一个local tasks是：

1. selective-install architecture
2. ECC 2.0 discovery doc
3. PR `#399` review
