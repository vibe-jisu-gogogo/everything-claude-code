---
name: eval-harness
description: 实现 eval-driven development (EDD) 原则的 Claude Code 会话正式评估框架
allowed-tools: Read, Write, Edit, Bash, Grep, Glob
---

# Eval Harness 技能

为 Claude Code 会话提供的正式评估框架，实现 eval-driven development (EDD) 原则。

## 激活时机

- 为 AI 辅助 workflow 设置 eval-driven development (EDD)
- 为 Claude Code 任务完成定义通过/失败标准
- 使用 pass@k 指标衡量 agent 可靠性
- 为 prompt 或 agent 变更创建回归测试套件
- 跨 model 版本 benchmarking agent 性能

## 设计理念

Eval-Driven Development 将评估视为"AI 开发的单元测试"：
- 实现前先定义预期行为
- 开发过程中持续运行评估
- 跟踪每次变更导致的回归问题
- 使用 pass@k 指标衡量可靠性

## 评估类型

### 能力评估
测试 Claude 是否能完成以前无法实现的任务：
```markdown
[CAPABILITY EVAL: feature-name]
任务：Claude 应该完成的任务描述
成功标准：
  - [ ] 标准 1
  - [ ] 标准 2
  - [ ] 标准 3
预期输出：预期结果描述
```

### 回归评估
确保变更不会破坏现有功能：
```markdown
[REGRESSION EVAL: feature-name]
基线：SHA 或 checkpoint 名称
测试：
  - existing-test-1: PASS/FAIL
  - existing-test-2: PASS/FAIL
  - existing-test-3: PASS/FAIL
结果：X/Y 项通过（之前为 Y/Y 项全部通过）
```

## 评分器类型

### 1. 基于代码的评分器
使用代码进行确定性检查：
```bash
# 检查文件是否包含预期模式
grep -q "export function handleAuth" src/auth.ts && echo "PASS" || echo "FAIL"

# 检查测试是否通过
npm test -- --testPathPattern="auth" && echo "PASS" || echo "FAIL"

# 检查构建是否成功
npm run build && echo "PASS" || echo "FAIL"
```

### 2. 基于模型的评分器
使用 Claude 评估开放输出结果：
```markdown
[MODEL GRADER PROMPT]
评估以下代码变更：
1. 是否解决了所述问题？
2. 结构是否合理？
3. 边缘情况是否处理？
4. 错误处理是否合适？

评分：1-5（1=差，5=优秀）
推理：[说明]
```

### 3. 人工评分器
标记内容以供人工审核：
```markdown
[需要人工审核]
变更：变更内容描述
原因：需要人工审核的原因
风险等级：LOW/MEDIUM/HIGH
```

## 指标

### pass@k
"k次尝试中至少成功一次"
- pass@1: 首次尝试成功率
- pass@3: 3次尝试内成功
- 典型目标：pass@3 > 90%

### pass^k
"k次尝试全部成功"
- 更高的可靠性标准
- pass^3: 连续3次成功
- 用于关键路径

## 评估 Workflow

### 1. 定义（编码前）
```markdown
## 评估定义：feature-xyz

### 能力评估
1. 可创建新用户账户
2. 可验证邮箱格式
3. 可安全加密密码

### 回归评估
1. 现有登录功能正常工作
2. Session 管理逻辑不变
3. 登出流程完整

### 成功指标
- 能力评估 pass@3 > 90%
- 回归评估 pass^3 = 100%
```

### 2. 实现
编写代码以通过已定义的评估。

### 3. 评估
```bash
# 运行能力评估
[运行每个能力评估，记录 PASS/FAIL]

# 运行回归评估
npm test -- --testPathPattern="existing"

# 生成报告
```

### 4. 报告
```markdown
评估报告：feature-xyz
========================

能力评估：
  create-user:     PASS (pass@1)
  validate-email:  PASS (pass@2)
  hash-password:   PASS (pass@1)
  总体：         3/3 项通过

回归评估：
  login-flow:      PASS
  session-mgmt:    PASS
  logout-flow:     PASS
  总体：         3/3 项通过

指标：
  pass@1: 67% (2/3)
  pass@3: 100% (3/3)

状态：待审核
```

## 集成模式

### 实现前
```
/eval define feature-name
```
在`.claude/evals/feature-name.md`创建评估定义文件。

### 实现过程中
```
/eval check feature-name
```
运行当前评估并报告状态。

### 实现后
```
/eval report feature-name
```
生成完整评估报告。

## 评估存储
将评估存储在项目中：
```
.claude/
  evals/
    feature-xyz.md      # 评估定义
    feature-xyz.log     # 评估运行历史
    baseline.json       # 回归基线
```

## 最佳实践

1. **编码前先定义评估** - 促使你清晰思考成功标准
2. **频繁运行评估** - 尽早发现回归问题
3. **长期跟踪 pass@k 指标** - 监控可靠性趋势
4. **尽可能使用代码评分器** - 确定性 > 概率性
5. **安全相关内容走人工审核** - 永远不要完全自动化安全检查
6. **保持评估运行速度快** - 没人会运行慢的评估
7. **评估随代码一起版本控制** - 评估是一等 artifact

## 示例：添加认证功能

```markdown
## 评估：add-authentication

### 阶段1：定义（10分钟）
能力评估：
- [ ] 用户可使用邮箱/密码注册
- [ ] 用户可使用有效凭证登录
- [ ] 无效凭证会被拒绝并返回正确错误
- [ ] Session 在页面刷新后仍然有效
- [ ] 登出会清空 session

回归评估：
- [ ] 公共路由仍然可访问
- [ ] API 响应不变
- [ ] 数据库 schema 兼容

### 阶段2：实现（时间不定）
[编写代码]

### 阶段3：评估
运行：/eval check add-authentication

### 阶段4：报告
评估报告：add-authentication
==============================
能力评估：5/5 项通过（pass@3: 100%）
回归评估：3/3 项通过（pass^3: 100%）
状态：可发布
```
