---
description: "使用自然语言指定文件的快速提交 — 用简单英语描述要提交的内容"
argument-hint: "[目标描述] (留空 = 所有变更)"
---

# 智能提交

> 改编自 Wirasm 的 PRPs-agentic-eng。属于 PRP 工作流系列的一部分。

**输入**: $ARGUMENTS

---

## 阶段 1 — 评估

```bash
git status --short
```

如果输出为空 → 停止："没有可提交的内容。"

向用户展示变更摘要（新增、修改、删除、未跟踪）。

---

## 阶段 2 — 解释与暂存

解释 `$ARGUMENTS` 以确定要暂存的内容：

| 输入 | 解释 | Git 命令 |
|---|---|---|
| *(留空/空)* | 暂存所有内容 | `git add -A` |
| `staged` | 使用已暂存的所有内容 | *(不执行 git add)* |
| `*.ts` 或 `*.py` 等 | 暂存匹配 glob 模式的文件 | `git add '*.ts'` |
| `except tests` | 暂存所有内容，然后取消暂存测试文件 | `git add -A && git reset -- '**/*.test.*' '**/*.spec.*' '**/test_*' 2>/dev/null || true` |
| `only new files` | 仅暂存未跟踪文件 | `git ls-files --others --exclude-standard | grep . && git ls-files --others --exclude-standard | xargs git add` |
| `the auth changes` | 从 status/diff 中解释 — 查找与 auth 相关的文件 | `git add <matched files>` |
| 特定文件名 | 暂存这些文件 | `git add <files>` |

对于自然语言输入（如 "the auth changes"），交叉参考 `git status` 输出和 `git diff` 来识别相关文件。向用户展示你正在暂存哪些文件以及原因。

```bash
git add <determined files>
```

暂存后验证：
```bash
git diff --cached --stat
```

如果没有暂存任何内容，停止："没有文件匹配你的描述。"

---

## 阶段 3 — 提交

编写命令式语气的单行提交信息：

```
{type}: {description}
```

类型：
- `feat` — 新功能或能力
- `fix` — Bug 修复
- `refactor` — 不改变行为的代码重构
- `docs` — 文档变更
- `test` — 添加或更新测试
- `chore` — 构建、配置、依赖
- `perf` — 性能优化
- `ci` — CI/CD 变更

规则：
- 命令式语气（"add feature" 而不是 "added feature"）
- 类型前缀后使用小写
- 末尾不加句号
- 长度不超过72个字符
- 描述变更内容，而不是变更方式

```bash
git commit -m "{type}: {description}"
```

---

## 阶段 4 — 输出

向用户报告：

```
已提交: {hash_short}
信息:   {type}: {description}
文件:     {count} 个文件已变更

下一步:
  - git push           → 推送到远程
  - /prp-pr            → 创建 pull request
  - /code-review       → 推送前进行代码审查
```

---

## 示例

| 你输入 | 执行结果 |
|---|---|
| `/prp-commit` | 暂存所有内容，自动生成提交信息 |
| `/prp-commit staged` | 仅提交已暂存的内容 |
| `/prp-commit *.ts` | 暂存所有 TypeScript 文件并提交 |
| `/prp-commit except tests` | 暂存除测试文件外的所有内容 |
| `/prp-commit the database migration` | 从 status 中查找数据库迁移文件并暂存 |
| `/prp-commit only new files` | 仅暂存未跟踪文件 |
