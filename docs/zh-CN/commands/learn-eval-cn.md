---
description: "从会话中提取可重用模式，在保存前自我评估质量，并确定正确的保存位置（Global 与 Project）。"
---

# /learn-eval - 提取、评估、然后保存

扩展 `/learn`，在编写任何技能文件之前，加入质量门控、保存位置决策和知识放置意识。

## 提取内容

寻找：

1. **错误解决模式** — 根本原因 + 修复方法 + 可重用性
2. **调试技术** — 非显而易见的步骤、工具组合
3. **变通方法** — 库的 quirks、API 限制、特定版本的修复
4. **项目特定模式** — 约定、架构决策、集成模式

## 流程

1. 回顾会话，寻找可提取的模式

2. 识别最有价值/可重用的见解

3. **确定保存位置：**
   - 提问："这个模式在其他项目中会有用吗？"
   - **Global** (`~/.claude/skills/learned/`)：可在 2 个以上项目中使用的通用模式（bash 兼容性、LLM API 行为、调试技术等）
   - **Project** (当前项目中的 `.claude/skills/learned/`)：项目特定的知识（特定配置文件的 quirks、项目特定的架构决策等）
   - 不确定时，选择 Global（将 Global → Project 移动比反向操作更容易）

4. 使用此格式起草技能文件：

```markdown
---
name: pattern-name
description: "Under 130 characters"
user-invocable: false
origin: auto-extracted
---

# [描述性模式名称]

**Extracted:** [日期]
**Context:** [简要描述此模式适用的场景]

## Problem
[此模式解决的具体问题 - 请详细说明]

## Solution
[模式/技术/变通方案 - 附带代码示例]

## When to Use
[触发条件]
```

5. **质量门控 — Checklist + 整体裁决**

   ### 5a. 必需 checklist（通过实际阅读文件进行验证）

   在评估草案**之前**，执行以下所有操作：

   - [ ] 使用关键字在 `~/.claude/skills/` 和相关项目的 `.claude/skills/` 文件中进行 grep 搜索，检查内容重叠
   - [ ] 检查 MEMORY.md（项目级和全局级）以查找重叠内容
   - [ ] 考虑是否追加到现有技能即可满足需求
   - [ ] 确认这是一个可复用的模式，而非一次性修复

   ### 5b. 整体裁决

   综合 checklist 结果和草案质量，然后选择**以下一项**：

   | 裁决 | 含义 | 下一步行动 |
   |---------|---------|-------------|
   | **Save** | 独特、具体、范围明确 | 进行到步骤 6 |
   | **Improve then Save** | 有价值但需要改进 | 列出改进项 → 修订 → 重新评估（一次） |
   | **Absorb into [X]** | 应追加到现有技能 | 显示目标技能和添加内容 → 步骤 6 |
   | **Drop** | 琐碎、冗余或过于抽象 | 解释原因并停止 |

   **指导维度**（用于告知裁决，不进行评分）：

   - **Specificity & Actionability**：包含可立即使用的代码示例或命令
   - **Scope Fit**：名称、触发条件和内容保持一致，并专注于单一模式
   - **Uniqueness**：提供现有技能未涵盖的价值（基于 checklist 结果）
   - **Reusability**：在未来的会话中存在现实的触发场景

6. **裁决特定的确认流程**

   - **Improve then Save**：呈现必需的改进项 + 修订后的草案 + 一次重新评估后的更新 checklist/裁决；如果修订后的裁决是 **Save**，则在用户确认后保存，否则遵循新的裁决
   - **Save**：呈现保存路径 + checklist 结果 + 1 行裁决理由 + 完整草案 → 在用户确认后保存
   - **Absorb into [X]**：呈现目标路径 + 添加内容（diff 格式） + checklist 结果 + 裁决理由 → 在用户确认后追加
   - **Drop**：仅显示 checklist 结果 + 推理（无需确认）

7. Save / Absorb 到确定的位置

## 步骤 5 的输出格式

```
### Checklist
- [x] skills/ grep: 无重叠 (或: 发现重叠 → 详情)
- [x] MEMORY.md: 无重叠 (或: 发现重叠 → 详情)
- [x] Existing skill append: 新文件合适 (或: 应追加到 [X])
- [x] Reusability: 已确认 (或: one-off → Drop)

### Verdict: Save / Improve then Save / Absorb into [X] / Drop

**Rationale:** (用 1-2 句话解释裁决)
```

## 设计原理

此版本用基于 checklist 的整体裁决系统取代了之前的 5 维度数字评分标准（Specificity、Actionability、Scope Fit、Non-redundancy、Coverage 评分 1-5）。现代前沿模型（Opus 4.6+）具有强大的情境判断能力 —— 将丰富的定性信号强行压缩为数字评分会丢失细微差别，并可能产生误导性的总分。整体方法让模型自然地权衡所有因素，产生更准确的 save/drop 决策，同时明确的 checklist 确保不会跳过任何关键检查。

## 注意事项

- 不要提取琐碎的修复（拼写错误、简单的语法错误）
- 不要提取一次性问题（特定的 API 中断等）
- 专注于那些将在未来会话中节省时间的模式
- 保持技能聚焦 —— 每个技能一个模式
- 当裁决为 Absorb 时，追加到现有技能，而不是创建新文件
