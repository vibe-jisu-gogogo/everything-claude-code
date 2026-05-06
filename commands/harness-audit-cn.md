---
description: 运行确定性的仓库 harness 审计并返回优先级评分卡。
---

# Harness Audit 命令

运行确定性的仓库 harness 审计并返回优先级评分卡。

## 使用方法

`/harness-audit [scope] [--format text|json] [--root path]`

- `scope`（可选）：`repo`（默认）、`hooks`、`skills`、`commands`、`agents`
- `--format`：输出样式（默认 `text`，`json` 用于自动化场景）
- `--root`：审计指定路径而非当前工作目录

## 确定性引擎

始终运行：

```bash
node scripts/harness-audit.js <scope> --format <text|json> [--root <path>]
```

该脚本是评分和检查的唯一事实来源。请勿自行添加额外的评估维度或临时评分项。

评分规则版本：`2026-03-30`。

脚本计算7个固定类别（每个类别标准化为`0-10`分）：

1. Tool Coverage
2. Context Efficiency
3. Quality Gates
4. Memory Persistence
5. Eval Coverage
6. Security Guardrails
7. Cost Efficiency

分数基于明确的文件/规则检查得出，对同一提交而言结果可复现。
脚本默认审计当前工作目录，并自动检测目标是ECC仓库本身还是使用ECC的消费项目。

## 输出约定

返回内容：

1. `overall_score` 总分对比 `max_score`（`repo` 范围为70分；范围更小的审计分值更低）
2. 各分类得分及具体发现
3. 未通过检查的具体文件路径
4. 确定性输出中的前3项行动建议（`top_actions`）
5. 下一步建议应用的ECC技能

## 检查清单

- 直接使用脚本输出；请勿手动重新评分。
- 如果请求 `--format json`，原样返回脚本输出的JSON。
- 如果请求文本格式，汇总未通过检查项和顶部行动建议。
- 包含来自 `checks[]` 和 `top_actions[]` 的精确文件路径。

## 结果示例

```text
Harness Audit (repo): 66/70
- Tool Coverage: 10/10 (10/10 pts)
- Context Efficiency: 9/10 (9/10 pts)
- Quality Gates: 10/10 (10/10 pts)

Top 3 Actions:
1) [Security Guardrails] Add prompt/tool preflight security guards in hooks/hooks.json. (hooks/hooks.json)
2) [Tool Coverage] Sync commands/harness-audit.md and .opencode/commands/harness-audit.md. (.opencode/commands/harness-audit.md)
3) [Eval Coverage] Increase automated test coverage across scripts/hooks/lib. (tests/)
```

## 参数

$ARGUMENTS:
- `repo|hooks|skills|commands|agents`（可选范围）
- `--format text|json`（可选输出格式）
