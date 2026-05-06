---
description: Code review — 本地未提交变更或GitHub PR (传入PR编号/URL进入PR模式)
argument-hint: [pr-number | pr-url | 留空进行本地审查]
---

# 代码审查

> PR审查模式改编自Wirasm的PRPs-agentic-eng，属于PRP工作流系列的一部分。

**输入**: $ARGUMENTS

---

## 模式选择

如果 `$ARGUMENTS` 包含PR编号、PR URL或 `--pr`：
→ 跳转至下方的**PR审查模式**。

否则：
→ 使用**本地审查模式**。

---

## 本地审查模式

对未提交变更进行全面的安全和质量审查。

### 阶段1 — 收集

```bash
git diff --name-only HEAD
```

如果没有变更文件，停止："Nothing to review."

### 阶段2 — 审查

完整读取每个变更文件，检查以下内容：

**安全问题 (CRITICAL):**
- 硬编码凭据、API keys、tokens
- SQL注入漏洞
- XSS漏洞
- 缺失输入校验
- 不安全的依赖
- 路径遍历风险

**代码质量 (HIGH):**
- 函数超过50行
- 文件超过800行
- 嵌套深度超过4层
- 缺失错误处理
- console.log 语句
- TODO/FIXME 注释
- 公共API缺失JSDoc

**最佳实践 (MEDIUM):**
- 可变模式（优先使用不可变模式）
- 代码/注释中使用emoji
- 新代码缺失测试
- 可访问性问题 (a11y)

### 阶段3 — 报告

生成报告包含以下内容：
- 严重级别：CRITICAL、HIGH、MEDIUM、LOW
- 文件位置和行号
- 问题描述
- 修复建议

如果发现CRITICAL或HIGH级别问题，阻止提交。
永远不要批准存在安全漏洞的代码。

---

## PR审查模式

全面的GitHub PR审查 — 获取diff、读取完整文件、运行验证、发布审查结果。

### 阶段1 — 获取

解析输入确定PR：

| 输入 | 操作 |
|---|---|
| 数字（例如 `42`） | 用作PR编号 |
| URL (`github.com/.../pull/42`) | 提取PR编号 |
| 分支名称 | 通过 `gh pr list --head <branch>` 查找PR |

```bash
gh pr view <NUMBER> --json number,title,body,author,baseRefName,headRefName,changedFiles,additions,deletions
gh pr diff <NUMBER>
```

如果未找到PR，报错停止。存储PR元数据供后续阶段使用。

### 阶段2 — 上下文

构建审查上下文：

1. **项目规则** — 读取 `CLAUDE.md`、`.claude/docs/` 和所有贡献指南
2. **PRP产物** — 检查 `.claude/PRPs/reports/` 和 `.claude/PRPs/plans/` 中与该PR相关的实现上下文
3. **PR意图** — 解析PR描述中的目标、关联issue、测试计划
4. **变更文件** — 列出所有修改文件并按类型分类（源码、测试、配置、文档）

### 阶段3 — 审查

**完整**读取每个变更文件（不只是diff片段 — 需要上下文环境）。

对于PR审查，获取PR head版本的完整文件内容：
```bash
gh pr diff <NUMBER> --name-only | while IFS= read -r file; do
  gh api "repos/{owner}/{repo}/contents/$file?ref=<head-branch>" --jq '.content' | base64 -d
done
```

应用7个类别的审查清单：

| 类别 | 检查内容 |
|---|---|
| **正确性** | 逻辑错误、off-by-one错误、空值处理、边界场景、竞态条件 |
| **类型安全** | 类型不匹配、不安全的类型转换、`any` 使用、缺失泛型 |
| **模式合规性** | 符合项目规范（命名、文件结构、错误处理、导入） |
| **安全** | 注入、认证漏洞、机密泄露、SSRF、路径遍历、XSS |
| **性能** | N+1查询、缺失索引、无边界循环、内存泄漏、大负载 |
| **完整性** | 缺失测试、缺失错误处理、不完整的迁移、缺失文档 |
| **可维护性** | 死代码、魔术数字、深层嵌套、命名不清晰、缺失类型 |

为每个发现分配严重级别：

| 严重级别 | 含义 | 操作 |
|---|---|---|
| **CRITICAL** | 安全漏洞或数据丢失风险 | 合并前必须修复 |
| **HIGH** | 可能导致问题的bug或逻辑错误 | 合并前应该修复 |
| **MEDIUM** | 代码质量问题或缺失最佳实践 | 建议修复 |
| **LOW** | 风格问题或次要建议 | 可选 |

### 阶段4 — 验证

运行可用的验证命令：

从配置文件（`package.json`、`Cargo.toml`、`go.mod`、`pyproject.toml` 等）检测项目类型，然后运行对应的命令：

**Node.js / TypeScript**（存在 `package.json`）:
```bash
npm run typecheck 2>/dev/null || npx tsc --noEmit 2>/dev/null  # Type check
npm run lint                                                    # Lint
npm test                                                        # Tests
npm run build                                                   # Build
```

**Rust**（存在 `Cargo.toml`）:
```bash
cargo clippy -- -D warnings  # Lint
cargo test                   # Tests
cargo build                  # Build
```

**Go**（存在 `go.mod`）:
```bash
go vet ./...    # Lint
go test ./...   # Tests
go build ./...  # Build
```

**Python**（存在 `pyproject.toml` / `setup.py`）:
```bash
pytest  # Tests
```

仅运行适用于检测到的项目类型的命令。记录每个命令的通过/失败状态。

### 阶段5 — 决策

基于发现形成建议：

| 条件 | 决策 |
|---|---|
| 无CRITICAL/HIGH级别问题，验证通过 | **APPROVE** |
| 仅有MEDIUM/LOW级别问题，验证通过 | **APPROVE** 并附评论 |
| 存在任何HIGH级别问题或验证失败 | **REQUEST CHANGES** |
| 存在任何CRITICAL级别问题 | **BLOCK** — 合并前必须修复 |

特殊场景：
- 草稿PR → 始终使用 **COMMENT**（不批准/阻止）
- 仅文档/配置变更 → 简化审查，重点关注正确性
- 明确的 `--approve` 或 `--request-changes` 标志 → 覆盖决策（但仍需报告所有发现）

### 阶段6 — 报告

在 `.claude/PRPs/reviews/pr-<NUMBER>-review.md` 创建审查产物：

```markdown
# PR Review: #<NUMBER> — <TITLE>

**Reviewed**: <date>
**Author**: <author>
**Branch**: <head> → <base>
**Decision**: APPROVE | REQUEST CHANGES | BLOCK

## Summary
<1-2 sentence overall assessment>

## Findings

### CRITICAL
<findings or "None">

### HIGH
<findings or "None">

### MEDIUM
<findings or "None">

### LOW
<findings or "None">

## Validation Results

| Check | Result |
|---|---|
| Type check | Pass / Fail / Skipped |
| Lint | Pass / Fail / Skipped |
| Tests | Pass / Fail / Skipped |
| Build | Pass / Fail / Skipped |

## Files Reviewed
<list of files with change type: Added/Modified/Deleted>
```

### 阶段7 — 发布

将审查结果发布到GitHub：

```bash
# If APPROVE
gh pr review <NUMBER> --approve --body "<summary of review>"

# If REQUEST CHANGES
gh pr review <NUMBER> --request-changes --body "<summary with required fixes>"

# If COMMENT only (draft PR or informational)
gh pr review <NUMBER> --comment --body "<summary>"
```

针对特定行的内联评论，使用GitHub审查评论API：
```bash
gh api "repos/{owner}/{repo}/pulls/<NUMBER>/comments" \
  -f body="<comment>" \
  -f path="<file>" \
  -F line=<line-number> \
  -f side="RIGHT" \
  -f commit_id="$(gh pr view <NUMBER> --json headRefOid --jq .headRefOid)"
```

或者，一次性发布包含多个内联评论的单个审查：
```bash
gh api "repos/{owner}/{repo}/pulls/<NUMBER>/reviews" \
  -f event="COMMENT" \
  -f body="<overall summary>" \
  --input comments.json  # [{"path": "file", "line": N, "body": "comment"}, ...]
```

### 阶段8 — 输出

向用户报告：

```
PR #<NUMBER>: <TITLE>
Decision: <APPROVE|REQUEST_CHANGES|BLOCK>

Issues: <critical_count> critical, <high_count> high, <medium_count> medium, <low_count> low
Validation: <pass_count>/<total_count> checks passed

Artifacts:
  Review: .claude/PRPs/reviews/pr-<NUMBER>-review.md
  GitHub: <PR URL>

Next steps:
  - <contextual suggestions based on decision>
```

---

## 边缘场景

- **无 `gh` CLI**: 回退到仅本地审查（读取diff，跳过GitHub发布）。警告用户。
- **分支分叉**: 审查前建议执行 `git fetch origin && git rebase origin/<base>`。
- **大型PR（>50个文件）**: 警告审查范围。先重点关注源码变更，然后是测试，最后是配置/文档。
