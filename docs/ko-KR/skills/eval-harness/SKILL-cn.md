---
name: eval-harness
description: 实现 Eval-Driven Development (EDD) 原则的官方 Claude Code 会话评估框架
origin: ECC
tools: Read, Write, Edit, Bash, Grep, Glob
---

# 评估框架技能

官方的 Claude Code 会话评估框架，实现了评估驱动开发 (EDD) 原则。

## 何时使用

- 在 AI 辅助工作流中设置评估驱动开发 (EDD) 时
- 定义 Claude Code 任务完成的通过/失败标准时
- 使用 pass@k 指标测量 Agent 可靠性时
- 为提示词或 Agent 变更创建回归测试套件时
- 在模型版本之间基准测试 Agent 性能时

## 理念

评估驱动开发将评估视为"AI 开发的单元测试"：
- 实现前定义预期行为
- 开发期间持续运行评估
- 每次变更时追踪回归
- 使用 pass@k 指标衡量可靠性

## 评估类型

### 功能评估
测试 Claude 是否能做以前不能做的事情：
```markdown
[CAPABILITY EVAL: feature-name]
Task: Description of what Claude should accomplish
Success Criteria:
  - [ ] Criterion 1
  - [ ] Criterion 2
  - [ ] Criterion 3
Expected Output: Description of expected result
```

### 回归评估
确保变更不会破坏现有功能：
```markdown
[REGRESSION EVAL: feature-name]
Baseline: SHA or checkpoint name
Tests:
  - existing-test-1: PASS/FAIL
  - existing-test-2: PASS/FAIL
  - existing-test-3: PASS/FAIL
Result: X/Y passed (previously Y/Y)
```

## 评分者类型

### 1. 代码评分者
使用代码进行确定性检查：
```bash
# Check if file contains expected pattern
grep -q "export function handleAuth" src/auth.ts && echo "PASS" || echo "FAIL"

# Check if tests pass
npm test -- --testPathPattern="auth" && echo "PASS" || echo "FAIL"

# Check if build succeeds
npm run build && echo "PASS" || echo "FAIL"
```

### 2. 模型评分者
使用 Claude 评估开放式输出：
```markdown
[MODEL GRADER PROMPT]
Evaluate the following code change:
1. Does it solve the stated problem?
2. Is it well-structured?
3. Are edge cases handled?
4. Is error handling appropriate?

Score: 1-5 (1=poor, 5=excellent)
Reasoning: [explanation]
```

### 3. 人工评分者
标记需要人工审查：
```markdown
[HUMAN REVIEW REQUIRED]
Change: Description of what changed
Reason: Why human review is needed
Risk Level: LOW/MEDIUM/HIGH
```

## 指标

### pass@k
"k 次尝试中至少成功一次"
- pass@1：首次尝试成功率
- pass@3：3 次尝试内成功
- 典型目标：pass@3 > 90%

### pass^k
"k 次试验全部成功"
- 可靠性的更高标准
- pass^3：连续 3 次成功
- 用于关键路径

## 评估工作流

### 1. 定义（编码前）
```markdown
## EVAL DEFINITION: feature-xyz

### Capability Evals
1. Can create new user account
2. Can validate email format
3. Can hash password securely

### Regression Evals
1. Existing login still works
2. Session management unchanged
3. Logout flow intact

### Success Metrics
- pass@3 > 90% for capability evals
- pass^3 = 100% for regression evals
```

### 2. 实现
编写代码以通过定义的评估。

### 3. 评估
```bash
# Run capability evals
[Run each capability eval, record PASS/FAIL]

# Run regression evals
npm test -- --testPathPattern="existing"

# Generate report
```

### 4. 报告
```markdown
EVAL REPORT: feature-xyz
========================

Capability Evals:
  create-user:     PASS (pass@1)
  validate-email:  PASS (pass@2)
  hash-password:   PASS (pass@1)
  Overall:         3/3 passed

Regression Evals:
  login-flow:      PASS
  session-mgmt:    PASS
  logout-flow:     PASS
  Overall:         3/3 passed

Metrics:
  pass@1: 67% (2/3)
  pass@3: 100% (3/3)

Status: READY FOR REVIEW
```

## 集成模式

### 实现前
```
/eval define feature-name
```
在 `.claude/evals/feature-name.md` 中创建评估定义文件

### 实现中
```
/eval check feature-name
```
运行当前评估并报告状态

### 实现后
```
/eval report feature-name
```
生成完整评估报告

## 评估存储

在项目中存储评估：
```
.claude/
  evals/
    feature-xyz.md      # 评估定义
    feature-xyz.log     # 评估运行历史
    baseline.json       # 回归基线
```

## 最佳实践

1. **编码前定义评估** - 强制对成功标准进行清晰思考
2. **经常运行评估** - 尽早发现回归
3. **随时间追踪 pass@k** - 监控可靠性趋势
4. **尽可能使用代码评分者** - 确定性 > 概率性
5. **安全性需要人工审查** - 不要完全自动化安全检查
6. **保持评估快速** - 缓慢的评估不会被运行
7. **与代码一起版本化评估** - 评估是一等产出物

## 示例：添加认证

```markdown
## EVAL: add-authentication

### Phase 1: 定义 (10分钟)
Capability Evals:
- [ ] User can register with email/password
- [ ] User can login with valid credentials
- [ ] Invalid credentials rejected with proper error
- [ ] Sessions persist across page reloads
- [ ] Logout clears session

Regression Evals:
- [ ] Public routes still accessible
- [ ] API responses unchanged
- [ ] Database schema compatible

### Phase 2: 实现 (可变)
[Write code]

### Phase 3: 评估
Run: /eval check add-authentication

### Phase 4: 报告
EVAL REPORT: add-authentication
==============================
Capability: 5/5 passed (pass@3: 100%)
Regression: 3/3 passed (pass^3: 100%)
Status: SHIP IT
```

## 产品评估 (v1.8)

当行为质量无法仅通过单元测试捕获时，使用产品评估。

### 评分者类型

1. 代码评分者（确定性断言）
2. 规则评分者（正则表达式/模式约束）
3. 模型评分者（LLM 评审员评分标准）
4. 人工评分者（对模糊输出的人工判断）

### pass@k 指南

- `pass@1`：直接可靠性
- `pass@3`：受控重试下的实际可靠性
- `pass^3`：稳定性测试（必须全部通过 3 次）

推荐阈值：
- 功能评估：pass@3 >= 0.90
- 回归评估：发布关键路径的 pass^3 = 1.00

### 评估反模式

- 提示词过度拟合已知评估示例
- 仅测量正常路径输出
- 在追求通过率时忽略成本和延迟变化
- 允许不稳定的评分者进入发布门控

### 最小评估产出物布局

- `.claude/evals/<feature>.md` 定义
- `.claude/evals/<feature>.log` 运行历史
- `docs/releases/<version>/eval-summary.md` 发布快照
