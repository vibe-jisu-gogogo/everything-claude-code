---
description: 运行用于前端或视觉工作的生成器/评估器设计循环，包含有限迭代次数和评分机制。
---

从 $ARGUMENTS 中解析以下参数：
1. `brief` — 用户对要创建设计的描述
2. `--max-iterations N` — （可选，默认 10）最大设计-评估循环次数
3. `--pass-threshold N` — （可选，默认 7.5）通过所需的加权分数（设计场景默认值更高）

## GAN-Style Design Harness

一个双智能体循环（Generator + Evaluator）专注于前端设计质量。无需规划器 — 需求描述本身就是规范。

这与 Anthropic 在其前端设计实验中使用的模式相同，他们在这些实验中取得了创造性突破，比如使用 CSS perspective 和门道导航实现的 3D 荷兰艺术博物馆。

### 配置步骤
1. 创建 `gan-harness/` 目录
2. 将需求描述直接写入 `gan-harness/spec.md`
3. 编写一个专注于设计的 `gan-harness/eval-rubric.md`，额外侧重 Design Quality 和 Originality

### 设计专用评估标准
```markdown
### Design Quality (权重: 0.35)
### Originality (权重: 0.30)
### Craft (权重: 0.25)
### Functionality (权重: 0.10)
```

注意：Originality 权重更高（0.30 对比常规的 0.20）以推动创造性突破。Functionality 权重更低，因为设计模式专注于视觉质量。

### 循环流程
与 `/project:gan-build` 第二阶段相同，但有以下区别：
- 跳过规划器
- 使用专注于设计的评估标准
- Generator 提示词更强调视觉质量而非功能完整性
- Evaluator 提示词更强调 "这个设计是否能赢得设计奖项？" 而非 "所有功能是否都正常工作？"

### 与 gan-build 的核心区别
Generator 会收到指示："你的首要目标是视觉卓越。一个惊艳的半成品应用胜过一个功能完整但丑陋的应用。追求创造性的飞跃 — 不寻常的布局、自定义动画、独特的色彩设计。"
