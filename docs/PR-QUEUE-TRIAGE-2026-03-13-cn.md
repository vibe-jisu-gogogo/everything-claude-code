# PR 审查与队列分类 — 2026年3月13日

## 快照

本文档记录了 `everything-claude-code` 拉取请求队列截至 `2026-03-13T08:33:31Z` 的实时 GitHub 分类快照。

使用的数据源：

- `gh pr view`
- `gh pr checks`
- `gh pr diff --name-only`
- 针对已合并的 `#399` 提交头进行针对性本地验证

本次分类使用的过时阈值：

- `2026-02-11 之前最后更新`（2026年3月13日前超过30天）

## PR `#399` 回顾审查

PR 信息：

- `#399` — `fix(observe): 5层自动会话防护机制，防止自循环观察`
- 状态：`MERGED`
- 合并时间：`2026-03-13T06:40:03Z`
- 合并提交：`c52a28ace9e7e84c00309fc7b629955dfc46ecf9`

变更文件：

- `skills/continuous-learning-v2/hooks/observe.sh`
- `skills/continuous-learning-v2/agents/observer-loop.sh`

针对已合并提交头 `546628182200c16cc222b97673ddd79e942eacce` 执行的验证：

- 对两个变更的 shell 脚本执行 `bash -n` 语法检查
- `node tests/hooks/hooks.test.js`（204 个通过，0 个失败）
- 针对以下场景的针对性 hook 调用：
  - 交互式 CLI 会话
  - `CLAUDE_CODE_ENTRYPOINT=mcp`
  - `ECC_HOOK_PROFILE=minimal`
  - `ECC_SKIP_OBSERVE=1`
  - `agent_id` 负载
  - 经过裁剪的 `ECC_OBSERVE_SKIP_PATHS`

行为结果：

- 核心自循环修复有效
- 自动会话防护分支按预期抑制了观察写入
- 最终的 `非CLI => 退出` 入口逻辑是正确的失败关闭形态

剩余发现：

1. 中等级别：被跳过的自动会话在新防护退出前仍会创建 homunculus 项目状态。
   `observe.sh` 在到达自动会话防护块之前会解析 `cwd` 并加载项目检测，因此 `detect-project.sh` 仍会为后续提前退出的会话创建 `projects/<id>/...` 目录并更新 `projects.json`。
2. 低级别：新的防护矩阵在发布时没有直接的回归覆盖。
   Hook 测试套件仍然验证了相邻行为，但它没有直接断言新的 `CLAUDE_CODE_ENTRYPOINT`、`ECC_HOOK_PROFILE`、`ECC_SKIP_OBSERVE`、`agent_id` 或经过裁剪的跳过路径分支。

结论：

- `#399` 对于其主要目标在技术上是正确的，作为紧急循环停止修复进行合并不存在安全问题。
- 它仍然值得后续 issue 或补丁，将自动会话防护移到项目注册副作用之前，并添加明确的防护路径测试。

## 未关闭 PR 清单

当前有 `4` 个未关闭的 PR。

### 队列表格

| PR | 标题 | 草稿 | 可合并 | 合并状态 | 更新时间 | 过时 | 当前结论 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `#292` | `chore(config): 治理与配置基础 (PR #272 拆分 1/6)` | `false` | `MERGEABLE` | `UNSTABLE` | `2026-03-13T07:26:55Z` | `否` | `当前最佳合并候选` |
| `#298` | `feat(agents,skills,rules): 添加 Rust、Java、移动端、DevOps 和性能内容` | `false` | `CONFLICTING` | `DIRTY` | `2026-03-11T04:29:07Z` | `否` | `审查完成前需要变更` |
| `#336` | `Codex CLI 自定义 - 来自 Claude Code 和 OpenCode 的功能` | `true` | `MERGEABLE` | `UNSTABLE` | `2026-03-13T07:26:12Z` | `否` | `需要手动审查并退出草稿状态` |
| `#420` | `feat: 添加 Laravel 技能` | `true` | `MERGEABLE` | `UNSTABLE` | `2026-03-12T22:57:36Z` | `否` | `低风险草稿，退出草稿状态后审查` |

根据「最后更新超过30天」规则，当前所有未关闭 PR 均不过时。

## 逐项 PR 评估

### `#292` — 治理 / 配置基础

实时状态：

- 未关闭
- 非草稿
- `MERGEABLE`
- 合并状态 `UNSTABLE`
- 可见检查：
  - `CodeRabbit` 通过
  - `GitGuardian Security Checks` 通过

范围：

- `.env.example`
- `.github/ISSUE_TEMPLATE/copilot-task.md`
- `.github/PULL_REQUEST_TEMPLATE.md`
- `.gitignore`
- `.markdownlint.json`
- `.tool-versions`
- `VERSION`

评估：

- 这是当前队列中最干净的合并候选。
- 该分支已经刷新到当前 `main` 分支。
- 当前可见的机器人反馈是次要/细节级别的，并非明显的合并阻塞项。
- 主要注意事项是目前只可见外部机器人检查；当前 PR 检查输出中没有出现 GitHub Actions 矩阵运行。

当前建议：

- `经过最后一次所有者检查后可合并。`
- 如果想要保守路径，在合并前对剩余的 `.env.example`、PR 模板和 `.tool-versions` 细节进行一次快速人工审查。

### `#298` — 大型多域内容扩展

实时状态：

- 未关闭
- 非草稿
- `CONFLICTING`
- 合并状态 `DIRTY`
- 可见检查：
  - `CodeRabbit` 通过
  - `GitGuardian Security Checks` 通过
  - `cubic · AI 代码审查器` 通过

范围：

- `35` 个文件
- 覆盖 Java、Rust、移动端、DevOps、性能、数据和 MLOps 的大型文档与技能/规则扩展

评估：

- 该 PR 尚未准备好合并。
- 它与当前 `main` 分支存在冲突，因此在分支层面甚至还不可合并。
- cubic 在当前审查中识别了跨 35 个文件的 34 个问题。
  这些发现是实质性和技术性的，不仅仅是风格清理，它们涵盖了多个新技能中存在的损坏或误导性示例。
- 即使没有冲突，其范围也足够大，需要有针对性的内容修复过程，而非快速合并决策。

当前建议：

- `需要变更。`
- 首先变基或重排，然后解决实质性的示例质量问题。
- 如果推进速度很重要，按领域拆分而不是维护一个非常大的 PR。

### `#336` — Codex CLI 自定义

实时状态：

- 未关闭
- 草稿
- `MERGEABLE`
- 合并状态 `UNSTABLE`
- 可见检查：
  - `CodeRabbit` 通过
  - `GitGuardian Security Checks` 通过

范围：

- `scripts/codex-git-hooks/pre-commit`
- `scripts/codex-git-hooks/pre-push`
- `scripts/codex/check-codex-global-state.sh`
- `scripts/codex/install-global-git-hooks.sh`
- `scripts/sync-ecc-to-codex.sh`

评估：

- 该 PR 不再冲突，但它仍然只是草稿状态，尚未进行有意义的第一方审查。
- 它修改了用户全局 Codex 设置行为和 git hook 安装，因此操作影响范围比仅文档 PR 更大。
- 可见检查仅为外部机器人；当前检查集中没有显示完整的 GitHub Actions 运行。
- 由于该分支来自贡献者 fork 的 `main`，在更改状态之前，它还值得对具体提议内容进行额外的合理性检查。

当前建议：

- `合并就绪前需要变更`，其中所需变更为流程和审查导向的，而非已证实的代码缺陷：
  - 完成手动审查
  - 运行或确认全局状态脚本的验证
  - 仅在审查完成后将其退出草稿状态

### `#420` — Laravel 技能

实时状态：

- 未关闭
- 草稿
- `MERGEABLE`
- 合并状态 `UNSTABLE`
- 可见检查：
  - `CodeRabbit` 通过
  - `GitGuardian Security Checks` 通过

范围：

- `README.md`
- `examples/laravel-api-CLAUDE.md`
- `rules/php/patterns.md`
- `rules/php/security.md`
- `rules/php/testing.md`
- `skills/configure-ecc/SKILL.md`
- `skills/laravel-patterns/SKILL.md`
- `skills/laravel-security/SKILL.md`
- `skills/laravel-tdd/SKILL.md`
- `skills/laravel-verification/SKILL.md`

评估：

- 该 PR 内容密集，操作风险低于 `#336`。
- 它仍然是草稿状态，尚未进行实质性的人工审查。
- 可见检查仅为外部机器人。
- 实时 PR 状态中还没有发现合并阻塞项，但由于它仍然是草稿且审查不足，尚不能简单地进行合并。

当前建议：

- `在最高优先级非草稿工作完成后进行审查。`
- 一旦作者准备退出草稿状态，很可能是良好的审查候选。

## 可合并性分类

### 现在可合并或经过最后所有者检查后可合并

- `#292`

### 合并前需要变更

- `#298`
- `#336`

### 草稿 / 任何合并决策前需要审查

- `#420`

### 过时 `>30天`

- 无

## 建议顺序

1. `#292`
   这是当前最干净的实时合并候选。
2. `#420`
   运行时风险低，但等待退出草稿状态和实际审查过程。
3. `#336`
   仔细审查，因为它更改了全局 Codex 同步和 hook 行为。
4. `#298`
   在花费更多审查时间之前，先变基并修复实质性内容问题。

## 总结

- `#399`：安全的 bug 修复合并，仍值得进行一次后续清理
- `#292`：当前未关闭队列中的最高优先级合并候选
- `#298`：不可合并；冲突加上实质性内容缺陷
- `#336`：不再冲突，但由于仍是草稿且验证不足，尚未就绪
- `#420`：草稿、低风险内容通道，在非草稿队列之后审查

## 实时刷新

刷新时间：`2026-03-13T22:11:40Z`。

### 主分支

- `origin/main` 当前是绿色状态，包括 Windows 测试矩阵。
- 主线 CI 修复不是当前瓶颈。

### 更新后的队列读取

#### `#292` — 治理 / 配置基础

- 未关闭
- 非草稿
- `MERGEABLE`
- 可见检查：
  - `CodeRabbit` 通过
  - `GitGuardian Security Checks` 通过
- 最高价值的剩余工作不是 CI 修复；而是合并前对 `.env.example` 的小正确性检查和 PR 模板对齐

当前建议：

- `下一个可操作的 PR。`
- 要么修补剩余的文档/配置正确性问题，要么进行最后一次所有者检查，如果您接受当前权衡则进行合并。

#### `#420` — Laravel 技能

- 未关闭
- 草稿
- `MERGEABLE`
- 可见检查：
  - 由于 PR 是草稿状态，`CodeRabbit` 已跳过
  - `GitGuardian Security Checks` 通过
- 尚未可见实质性的人工审查

当前建议：

- `在非草稿队列之后审查。`
- 实现风险低，但由于仍是草稿且审查不足，尚未准备好合并。

#### `#336` — Codex CLI 自定义

- 未关闭
- 草稿
- `MERGEABLE`
- 可见检查：
  - `CodeRabbit` 通过
  - `GitGuardian Security Checks` 通过
- 仍然需要有针对性的手动审查，因为它涉及全局 Codex 同步和 git hook 安装行为

当前建议：

- `手动审查通道，非立即合并通道。`

#### `#298` — 大型内容扩展

- 未关闭
- 非草稿
- `CONFLICTING`
- 仍然是队列中剩余的最难处理的 PR

当前建议：

- `当前未关闭 PR 中的最低优先级。`
- 首先变基，然后处理实质性的内容/示例更正。

### 当前顺序

1. `#292`
2. `#420`
3. `#336`
4. `#298`
