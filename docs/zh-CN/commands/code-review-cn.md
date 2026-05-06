---
description: 代码审查 — 本地未提交更改或 GitHub PR（传入 PR 编号/URL 启用 PR 模式）
argument-hint: [pr-number | pr-url | blank for local review]
---

# 代码审查

> PR 审查模式改编自 Wirasm 的 PRPs-agentic-eng。PRP 工作流系列的一部分。

**输入**：$ARGUMENTS

---

## 模式选择

如果 `$ARGUMENTS` 包含 PR 编号、PR URL 或 `--pr`：
→ 跳转到下面的 **PR 审查模式**。

否则：
→ 使用 **本地审查模式**。

---

## 本地审查模式

对未提交的更改进行全面的安全性和质量审查。

### 阶段 1 — 收集

```bash
git diff --name-only HEAD
```

如果没有更改的文件，停止："没有需要审查的内容。"

### 阶段 2 — 审查

完整读取每个更改的文件。检查：

**安全问题（严重）：**
- 硬编码的凭据、API 密钥、令牌
- SQL 注入漏洞
- XSS 漏洞
- 缺少输入验证
- 不安全的依赖项
- 路径遍历风险

**代码质量（高）：**
- 函数超过 50 行
- 文件超过 800 行
- 嵌套深度超过 4 层
- 缺少错误处理
- console.log 语句
- TODO/FIXME 注释
- 公共 API 缺少 JSDoc

**最佳实践（中）：**
- 可变模式（应使用不可变模式）
- 代码/注释中使用表情符号
- 新代码缺少测试
- 无障碍性问题（a11y）

### 阶段 3 — 报告

生成报告，包含：
- 严重性：严重、高、中、低
- 文件位置和行号
- 问题描述
- 建议的修复方法

如果发现严重或高优先级问题，阻止提交。
绝不批准包含安全漏洞的代码。

---

## PR 审查模式

全面的 GitHub PR 审查 — 获取 diff、读取完整文件、运行验证、发布审查。

### 阶段 1 — 获取

解析输入以确定 PR：

| 输入 | 操作 |
|---|---|
| 编号（例如 `42`） | 用作 PR 编号 |
| URL（`github.com/.../pull/42`） | 提取 PR 编号 |
| 分支名称 | 通过 `gh pr list --head <branch>` 查找 PR |

```bash
gh pr view <NUMBER> --json number,title,body,author,baseRefName,headRefName,changedFiles,additions,deletions
gh pr diff <NUMBER>
```

如果未找到 PR，停止并报错。保存 PR 元数据供后续阶段使用。

### 阶段 2 — 上下文

构建审查上下文：

1. **项目规则** — 读取 `CLAUDE.md`、`.claude/docs/` 和任何贡献指南
2. **PRP 工件** — 检查 `.claude/PRPs/reports/` 和 `.claude/PRPs/plans/` 以获取与此 PR 相关的实现上下文
3. **PR 意图** — 解析 PR 描述以了解目标、关联的 issue、测试计划
4. **更改的文件** — 列出所有修改的文件并按类型分类（源代码、测试、配置、文档）

### 阶段 3 — 审查

**完整**读取每个更改的文件（不仅仅是 diff 片段 — 你需要周围的上下文）。

对于 PR 审查，获取 PR 头版本的完整文件内容：
```bash
gh pr diff <NUMBER> --name-only | while IFS= read -r file; do
  gh api "repos/{owner}/{repo}/contents/$file?ref=<head-branch>" --jq '.content' | base64 -d
done
```

在 7 个类别中应用审查清单：

| 类别 | 检查内容 |
|---|---|
| **正确性** | 逻辑错误、差一错误、空值处理、边界情况、竞态条件 |
| **类型安全** | 类型不匹配、不安全的类型转换、`any` 使用、缺少泛型 |
| **模式合规性** | 符合项目约定（命名、文件结构、错误处理、导入） |
| **安全性** | 注入、认证漏洞、秘密暴露、SSRF、路径遍历、XSS |
| **性能** | N+1 查询、缺少索引、无边界循环、内存泄漏、大负载 |
| **完整性** | 缺少测试、缺少错误处理、不完整的迁移、缺少文档 |
| **可维护性** | 死代码、魔术数字、深层嵌套、命名不清晰、缺少类型 |

为每个发现分配严重性：

| 严重性 | 含义 | 操作 |
|---|---|---|
| **严重** | 安全漏洞或数据丢失风险 | 合并前必须修复 |
| **高** | 可能导致问题的 bug 或逻辑错误 | 合并前应该修复 |
| **中** | 代码质量问题或缺少最佳实践 | 建议修复 |
| **低** | 样式细节或次要建议 | 可选 |

### 阶段 4 — 验证

运行可用的验证命令：

从配置文件（`package.json`、`Cargo.toml`、`go.mod`、`pyproject.toml` 等）检测项目类型，然后运行适当的命令：

**Node.js / TypeScript**（有 `package.json`）：
```bash
npm run typecheck 2>/dev/null || npx tsc --noEmit 2>/dev/null  # 类型检查
npm run lint                                                    # Lint
npm test                                                        # 测试
npm run build                                                   # 构建
```

**Rust**（有 `Cargo.toml`）：
```bash
cargo clippy -- -D warnings  # Lint
cargo test                   # 测试
cargo build                  # 构建
```

**Go**（有 `go.mod`）：
```bash
go vet ./...    # Lint
go test ./...   # 测试
go build ./...  # 构建
```

**Python**（有 `pyproject.toml` / `setup.py`）：
```bash
pytest  # 测试
```

只运行适用于检测到的项目类型的命令。记录每个命令的通过/失败状态。

### 阶段 5 — 决定

根据发现形成建议：

| 条件 | 决定 |
|---|---|
| 零个严重/高优先级问题，验证通过 | **批准** |
| 只有中/低优先级问题，验证通过 | **批准** 附带评论 |
| 任何高优先级问题或验证失败 | **请求更改** |
| 任何严重问题 | **阻止** — 合并前必须修复 |

特殊情况：
- 草稿 PR → 始终使用 **评论**（不批准/阻止）
- 只有文档/配置更改 → 较轻的审查，专注于正确性
- 显式 `--approve` 或 `--request-changes` 标志 → 覆盖决定（但仍报告所有发现）

### 阶段 6 — 报告

在 `.claude/PRPs/reviews/pr-<NUMBER>-review.md` 创建审查工件：

```markdown
# PR 审查：#<NUMBER> — <TITLE>

**审查日期**：<date>
**作者**：<author>
**分支**：<head> → <base>
**决定**：批准 | 请求更改 | 阻止

## 摘要
<1-2 句整体评估>

## 发现

### 严重
<发现或"无">

### 高
<发现或"无">

### 中
<发现或"无">

### 低
<发现或"无">

## 验证结果

| 检查 | 结果 |
|---|---|
| 类型检查 | 通过 / 失败 / 跳过 |
| Lint | 通过 / 失败 / 跳过 |
| 测试 | 通过 / 失败 / 跳过 |
| 构建 | 通过 / 失败 / 跳过 |

## 已审查文件
<文件列表及更改类型：添加/修改/删除>
```

### 阶段 7 — 发布

将审查发布到 GitHub：

```bash
# 如果批准
gh pr review <NUMBER> --approve --body "<审查摘要>"

# 如果请求更改
gh pr review <NUMBER> --request-changes --body "<包含必需修复的摘要>"

# 如果仅评论（草稿 PR 或信息性）
gh pr review <NUMBER> --comment --body "<摘要>"
```

对于特定行的内联评论，使用 GitHub 评论 API：
```bash
gh api "repos/{owner}/{repo}/pulls/<NUMBER>/comments" \
  -f body="<评论>" \
  -f path="<文件>" \
  -F line=<行号> \
  -f side="RIGHT" \
  -f commit_id="$(gh pr view <NUMBER> --json headRefOid --jq .headRefOid)"
```

或者，一次性发布包含多个内联评论的单个审查：
```bash
gh api "repos/{owner}/{repo}/pulls/<NUMBER>/reviews" \
  -f event="COMMENT" \
  -f body="<整体摘要>" \
  --input comments.json  # [{"path": "file", "line": N, "body": "comment"}, ...]
```

### 阶段 8 — 输出

向用户报告：

```
PR #<NUMBER>: <TITLE>
决定：<批准|请求更改|阻止>

问题：<critical_count> 严重，<high_count> 高，<medium_count> 中，<low_count> 低
验证：<pass_count>/<total_count> 项检查通过

工件：
  审查：.claude/PRPs/reviews/pr-<NUMBER>-review.md
  GitHub：<PR URL>

下一步：
  - <基于决定的上下文建议>
```

---

## 边缘情况

- **没有 `gh` CLI**：回退到仅本地审查（读取 diff，跳过 GitHub 发布）。警告用户。
- **分支分歧**：建议在审查前执行 `git fetch origin && git rebase origin/<base>`。
- **大型 PR（>50 个文件）**：警告审查范围。首先专注于源代码更改，然后是测试，最后是配置/文档。
