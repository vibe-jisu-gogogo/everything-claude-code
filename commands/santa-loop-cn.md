---
description: 对抗式双评审收敛循环 — 两个独立的模型评审员必须都批准后代码才能发布。
---

# Santa 循环

基于 santa-method 技能实现的对抗式双评审收敛循环。两个独立的评审员 — 使用不同模型，无共享上下文 — 必须都返回 NICE 后代码才能发布。

## 用途

针对当前任务输出运行两个独立评审员（Claude Opus + 一个外部模型）。两者都必须返回 NICE 后才能推送代码。如果任何一个返回 NAUGHTY，修复所有标记的问题，提交，然后重新运行全新的评审员 — 最多 3 轮。

## 用法

```
/santa-loop [file-or-glob | description]
```

## 工作流

### 步骤 1：确定评审范围

从 `$ARGUMENTS` 中确定范围，或者回退到未提交的变更：

```bash
git diff --name-only HEAD
```

读取所有变更的文件以构建完整的评审上下文。如果 `$ARGUMENTS` 指定了路径、文件或描述，则使用该内容作为范围。

### 步骤 2：构建评审标准

为正在评审的文件类型构建合适的评审标准。每个标准都必须有客观的 PASS/FAIL 条件。至少包含以下内容：

| 标准 | 通过条件 |
|-----------|---------------|
| 正确性 | 逻辑合理，无 bug，处理边缘情况 |
| 安全性 | 无密钥、注入、XSS 或 OWASP Top 10 问题 |
| 错误处理 | 显式处理错误，无静默吞错 |
| 完整性 | 满足所有需求，无缺失情况 |
| 内部一致性 | 文件或章节之间无矛盾 |
| 无回归 | 变更不会破坏现有行为 |

根据文件类型添加特定领域的标准（例如：TS 的类型安全，Rust 的内存安全，SQL 的迁移安全性）。

### 步骤 3：双独立评审

使用 Agent 工具**并行**启动两个评审员（在同一条消息中发送以实现并发执行）。两者都完成后再进入 verdict  gate（判决门）。

每个评审员将每个评审标准评估为 PASS 或 FAIL，然后返回结构化 JSON：

```json
{
  "verdict": "PASS" | "FAIL",
  "checks": [
    {"criterion": "...", "result": "PASS|FAIL", "detail": "..."}
  ],
  "critical_issues": ["..."],
  "suggestions": ["..."]
}
```

判决门（步骤 4）将这些映射为 NICE/NAUGHTY：两者都 PASS → NICE，任意一个 FAIL → NAUGHTY。

#### 评审员 A：Claude Agent（始终运行）

启动一个 Agent（subagent_type: `code-reviewer`，model: `opus`），传入完整的评审标准 + 所有待评审文件。提示词必须包含：
- 完整的评审标准
- 所有待评审文件内容
- "你是独立的质量评审员。你没有看过任何其他评审结果。你的工作是发现问题，而不是批准。"
- 返回上述结构化 JSON 判决结果

#### 评审员 B：外部模型（仅当没有安装外部 CLI 时使用 Claude 作为 fallback）

首先检测可用的 CLI：
```bash
command -v codex >/dev/null 2>&1 && echo "codex" || true
command -v gemini >/dev/null 2>&1 && echo "gemini" || true
```

构建评审提示词（与评审员 A 的评审标准 + 指令完全相同）并写入唯一的临时文件：
```bash
PROMPT_FILE=$(mktemp /tmp/santa-reviewer-b-XXXXXX.txt)
cat > "$PROMPT_FILE" << 'EOF'
... 完整的评审标准 + 文件内容 + 评审员指令 ...
EOF
```

使用第一个可用的 CLI：

**Codex CLI**（如果已安装）
```bash
codex exec --sandbox read-only -m gpt-5.4 -C "$(pwd)" - < "$PROMPT_FILE"
rm -f "$PROMPT_FILE"
```

**Gemini CLI**（如果已安装且 codex 未安装）
```bash
gemini -p "$(cat "$PROMPT_FILE")" -m gemini-2.5-pro
rm -f "$PROMPT_FILE"
```

**Claude Agent fallback**（仅当 `codex` 和 `gemini` 都未安装时使用）
启动第二个 Claude Agent（subagent_type: `code-reviewer`，model: `opus`）。记录警告：两个评审员使用相同的模型系列 — 未实现真正的模型多样性，但仍强制执行上下文隔离。

在所有情况下，评审员必须返回与评审员 A 相同的结构化 JSON 判决结果。

### 步骤 4：判决门

- **两者都 PASS** → **NICE** — 进入步骤 6（推送）
- **任意一个 FAIL** → **NAUGHTY** — 合并两个评审员的所有严重问题，去重，进入步骤 5

### 步骤 5：修复循环（NAUGHTY 路径）

1. 显示两个评审员的所有严重问题
2. 修复每个标记的问题 — 仅修改被标记的内容，不要进行顺路的重构
3. 将所有修复提交到单个 commit：
   ```
   fix: address santa-loop review findings (round N)
   ```
4. 使用**全新的评审员**重新运行步骤 3（没有之前轮次的记忆）
5. 重复直到两者都返回 PASS

**最多 3 次迭代。** 如果 3 轮后仍然是 NAUGHTY，停止并展示剩余问题：

```
SANTA LOOP ESCALATION (exceeded 3 iterations)

Remaining issues after 3 rounds:
- [列出两个评审员所有未解决的严重问题]

Manual review required before proceeding.
```

不要推送。

### 步骤 6：推送（NICE 路径）

当两个评审员都返回 PASS 时：

```bash
git push -u origin HEAD
```

### 步骤 7：最终报告

打印输出报告（参见下面的输出部分）。

## 输出

```
SANTA VERDICT: [NICE / NAUGHTY (escalated)]

Reviewer A (Claude Opus):   [PASS/FAIL]
Reviewer B ([model used]):  [PASS/FAIL]

Agreement:
  Both flagged:      [两个评审员都发现的问题]
  Reviewer A only:   [仅评审员 A 发现的问题]
  Reviewer B only:   [仅评审员 B 发现的问题]

Iterations: [N]/3
Result:     [PUSHED / ESCALATED TO USER]
```

## 注意事项

- 评审员 A（Claude Opus）始终运行 — 无论工具链如何，都能保证至少有一个强大的评审员。
- 评审员 B 的目标是模型多样性。GPT-5.4 或 Gemini 2.5 Pro 能提供真正的独立性 — 不同的训练数据，不同的偏见，不同的盲点。仅使用 Claude 的 fallback 仍然通过上下文隔离提供价值，但失去了模型多样性。
- 使用最强的可用模型：评审员 A 使用 Opus，评审员 B 使用 GPT-5.4 或 Gemini 2.5 Pro。
- 外部评审员使用 `--sandbox read-only`（Codex）运行，以防止评审过程中仓库被修改。
- 每轮使用全新的评审员可以防止先前发现导致的锚定偏差。
- 评审标准是最重要的输入。如果评审员盖章通过或标记主观风格问题，收紧评审标准。
- 在 NAUGHTY 轮次中进行提交，这样即使循环被中断，修复也会被保留。
- 仅在 NICE 后推送 — 永远不要在循环中间推送。
