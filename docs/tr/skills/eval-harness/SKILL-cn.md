---
name: eval-harness
description: 为应用 eval-driven development (EDD) 原则的 Claude Code 会话提供正式评估框架
origin: ECC
tools: Read, Write, Edit, Bash, Grep, Glob
---

# Eval Harness Skill

为 Claude Code 会话提供正式的评估框架，应用 eval-driven development (EDD) 原则。

## 何时激活

- 为 AI 驱动的工作流建立 eval-driven development (EDD) 时
- 为 Claude Code 任务完成定义通过/失败标准时
- 使用 pass@k 指标测量 agent 可靠性时
- 为 prompt 或 agent 变更创建回归测试套件时
- 在模型版本之间基准测试 agent 性能时

## 理念

Eval-Driven Development 将 eval 视为 "AI 开发的单元测试"：
- 在实现**之前**定义预期行为
- 在开发过程中持续运行 eval
- 跟踪每次变更的回归
- 使用 pass@k 指标进行可靠性测量

## Eval 类型

### Capability Eval
测试 Claude 是否能做以前做不到的事情：
```markdown
[CAPABILITY EVAL: feature-name]
任务：描述 Claude 应该完成的事情
成功标准：
  - [ ] 标准 1
  - [ ] 标准 2
  - [ ] 标准 3
预期输出：描述预期结果
```

### Regression Eval
确保变更不破坏现有功能：
```markdown
[REGRESSION EVAL: feature-name]
基线：SHA 或检查点名称
测试：
  - existing-test-1: PASS/FAIL
  - existing-test-2: PASS/FAIL
  - existing-test-3: PASS/FAIL
结果：X/Y 通过（之前 Y/Y）
```

## Grader 类型

### 1. Code-Based Grader
使用代码进行确定性检查：
```bash
# 检查文件是否包含预期模式
grep -q "export function handleAuth" src/auth.ts && echo "PASS" || echo "FAIL"

# 检查测试是否通过
npm test -- --testPathPattern="auth" && echo "PASS" || echo "FAIL"

# 检查构建是否成功
npm run build && echo "PASS" || echo "FAIL"
```

### 2. Model-Based Grader
使用 Claude 评估开放式输出：
```markdown
[MODEL GRADER PROMPT]
评估以下代码变更：
1. 它是否解决了指定的问题？
2. 它是否结构良好？
3. 是否处理了边界情况？
4. 错误处理是否恰当？

评分：1-5（1=差，5=完美）
理由：[说明]
```

### 3. Human Grader
标记为人工审查：
```markdown
[HUMAN REVIEW REQUIRED]
变更：描述发生了什么变化
原因：为什么需要人工审查
风险等级：低/中/高
```

## 指标

### pass@k
"k 次尝试中至少一次成功"
- pass@1：首次尝试成功率
- pass@3：3 次尝试内成功
- 典型目标：pass@3 > 90%

### pass^k
"所有 k 次尝试都成功"
- 可靠性的更高标准
- pass^3：连续 3 次成功
- 用于关键路径

## Eval 工作流

### 1. 定义（编码前）
```markdown
## EVAL DEFINITION: feature-xyz

### Capability Eval
1. 可以创建新用户账户
2. 可以验证邮箱格式
3. 可以安全地哈希密码

### Regression Eval
1. 现有登录仍然有效
2. 会话管理未更改
3. 登出流程保持完整

### 成功指标
- capability eval 的 pass@3 > 90%
- regression eval 的 pass^3 = 100%
```

### 2. 实现
编写代码以通过定义的 eval。

### 3. 评估
```bash
# 运行 capability eval
[运行每个 capability eval，记录 PASS/FAIL]

# 运行 regression eval
npm test -- --testPathPattern="existing"

# 生成报告
```

### 4. 报告
```markdown
EVAL REPORT: feature-xyz
========================

Capability Eval:
  create-user:     PASS (pass@1)
  validate-email:  PASS (pass@2)
  hash-password:   PASS (pass@1)
  总计：           3/3 通过

Regression Eval:
  login-flow:      PASS
  session-mgmt:    PASS
  logout-flow:     PASS
  总计：           3/3 通过

指标：
  pass@1: 67% (2/3)
  pass@3: 100% (3/3)

状态：准备审查
```

## 集成模式

### 实现前
```
/eval define feature-name
```
在 `.claude/evals/feature-name.md` 创建 eval 定义文件

### 实现期间
```
/eval check feature-name
```
运行现有 eval 并报告状态

### 实现后
```
/eval report feature-name
```
生成完整的 eval 报告

## Eval 存储

在项目中存储 eval：
```
.claude/
  evals/
    feature-xyz.md      # Eval 定义
    feature-xyz.log     # Eval 运行历史
    baseline.json       # Regression 基线
```

## 最佳实践

1. **在编码前定义 eval** - 强制明确思考成功标准
2. **频繁运行 eval** - 及早发现回归
3. **随时间跟踪 pass@k** - 观察可靠性趋势
4. **尽可能使用 code grader** - 确定性 > 概率性
5. **安全关键路径需要人工审查** - 永远不要完全自动化安全检查
6. **保持 eval 快速** - 慢速 eval 不会被运行
7. **将 eval 与代码一起版本化** - Eval 是一等工件

## 示例：添加身份验证

```markdown
## EVAL: add-authentication

### 阶段 1：定义（10 分钟）
Capability Eval:
- [ ] 用户可以使用邮箱/密码注册
- [ ] 用户可以使用有效凭据登录
- [ ] 无效凭据被适当的错误拒绝
- [ ] 会话在页面刷新后保持持久
- [ ] 登出清理会话

Regression Eval:
- [ ] 公共路由仍然可访问
- [ ] API 响应未更改
- [ ] 数据库模式兼容

### 阶段 2：实现（可变）
[编写代码]

### 阶段 3：评估
运行：/eval check add-authentication

### 阶段 4：报告
EVAL REPORT: add-authentication
==============================
Capability: 5/5 通过 (pass@3: 100%)
Regression: 3/3 通过 (pass^3: 100%)
状态：发布
```

## Product Eval (v1.8)

当行为质量无法通过单元测试单独捕获时，使用 product eval。

### Grader 类型

1. Code grader（确定性断言）
2. Rule grader（正则/模式约束）
3. Model grader（LLM-as-judge 规则）
4. Human grader（用于模糊输出的人工判断）

### pass@k 指南

- `pass@1`：直接可靠性
- `pass@3`：在受控重试下的实际可靠性
- `pass^3`：稳定性测试（所有 3 次运行必须通过）

建议阈值：
- Capability eval：pass@3 >= 0.90
- Regression eval：发布关键路径的 pass^3 = 1.00

### Eval 反模式

- 将 prompt 过度拟合到已知 eval 示例
- 仅测量快乐路径输出
- 在追求通过率时忽略成本和延迟偏移
- 在发布门中允许不稳定的 grader

### 最小 Eval 工件布局

- `.claude/evals/<feature>.md` 定义
- `.claude/evals/<feature>.log` 运行历史
- `docs/releases/<version>/eval-summary.md` 发布快照
