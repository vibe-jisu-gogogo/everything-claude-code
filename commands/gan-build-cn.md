---
description: 运行用于实现任务的 generator/evaluator 构建循环，包含限定迭代次数和评分功能。
---

从 $ARGUMENTS 中解析以下参数：
1. `brief` — 用户对要构建内容的一行描述
2. `--max-iterations N` — （可选，默认15）generator-evaluator 循环最大次数
3. `--pass-threshold N` — （可选，默认7.0）通过所需的加权评分
4. `--skip-planner` — （可选）跳过 planner，假设 spec.md 已存在
5. `--eval-mode MODE` — （可选，默认 "playwright"）可选值：playwright, screenshot, code-only

## GAN 风格的 Harness 构建

这个命令编排了一个三代理构建循环，灵感来自 Anthropic 2026年3月的 harness 设计论文。

### 阶段0：设置
1. 在项目根目录创建 `gan-harness/` 目录
2. 创建子目录：`gan-harness/feedback/`、`gan-harness/screenshots/`
3. 如果尚未初始化 git 则进行初始化
4. 记录开始时间和配置

### 阶段1：规划（Planner 代理）
除非设置了 `--skip-planner`：
1. 使用 Task 工具启动 `gan-planner` 代理，传入用户的 brief
2. 等待其生成 `gan-harness/spec.md` 和 `gan-harness/eval-rubric.md`
3. 向用户展示 spec 摘要
4. 进入阶段2

### 阶段2：Generator-Evaluator 循环
```
iteration = 1
while iteration <= max_iterations:

    # 生成
    使用 Task 工具启动 gan-generator 代理：
    - 读取 spec.md
    - 如果 iteration > 1：读取 feedback/feedback-{iteration-1}.md
    - 构建/改进应用
    - 确保 dev server 正在运行
    - 提交更改

    # 等待 generator 完成

    # 评估
    使用 Task 工具启动 gan-evaluator 代理：
    - 读取 eval-rubric.md 和 spec.md
    - 测试运行中的应用（模式：playwright/screenshot/code-only）
    - 按照评分标准打分
    - 将反馈写入 feedback/feedback-{iteration}.md

    # 等待 evaluator 完成

    # 检查评分
    读取 feedback/feedback-{iteration}.md
    提取加权总分

    if score >= pass_threshold:
        记录"在第 {iteration} 次迭代通过，得分 {score}"
        中断循环

    if iteration >= 3 and score has not improved in last 2 iterations:
        记录"检测到平台期 — 提前停止"
        中断循环

    iteration += 1
```

### 阶段3：总结
1. 读取所有反馈文件
2. 展示最终得分和迭代历史
3. 展示得分进度：`第1次迭代：4.2 → 第2次迭代：5.8 → ... → 第N次迭代：7.5`
4. 列出最终评估中剩余的所有问题
5. 报告总耗时和预估成本

### 输出

```markdown
## GAN Harness 构建报告

**需求说明：** [原始提示]
**结果：** 通过/失败
**迭代次数：** N / 最大次数
**最终得分：** X.X / 10

### 得分进度
| 迭代 | 设计 | 创意 | 工艺 | 功能性 | 总分 |
|------|--------|-------------|-------|---------------|-------|
| 1 | ... | ... | ... | ... | X.X |
| 2 | ... | ... | ... | ... | X.X |
| N | ... | ... | ... | ... | X.X |

### 剩余问题
- [来自最终评估的所有问题]

### 已创建文件
- gan-harness/spec.md
- gan-harness/eval-rubric.md
- gan-harness/feedback/feedback-001.md 至 feedback-NNN.md
- gan-harness/generator-state.md
- gan-harness/build-report.md
```

将完整报告写入 `gan-harness/build-report.md`。
