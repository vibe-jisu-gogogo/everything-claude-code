---
name: eval-harness
description: Claude Code 会话的正式评估框架，实现评估驱动开发（EDD）原则
tools: Read, Write, Edit, Bash, Grep, Glob
---

# Eval Harness 技能

Claude Code 会话的正式评估框架，实现评估驱动开发（EDD）原则。

## 哲学

评估驱动开发将评估视为"AI开发的单元测试"：
- 实现前定义预期行为
- 开发过程中持续运行评估
- 跟踪每次变更的回归
- 使用 pass@k 指标进行可靠性测量

## 评估类型

### 能力评估
测试 Claude 是否能完成以前无法完成的任务：
```markdown
[CAPABILITY EVAL: feature-name]
任务: Claude 应完成的任务描述
成功标准:
  - [ ] 标准1
  - [ ] 标准2
  - [ ] 标准3
预期输出: 预期结果的描述
```

### 回归评估
确保变更不会破坏现有功能：
```markdown
[REGRESSION EVAL: feature-name]
基线: SHA 或检查点名称
测试:
  - existing-test-1: PASS/FAIL
  - existing-test-2: PASS/FAIL
  - existing-test-3: PASS/FAIL
结果: X/Y 成功（之前为 Y/Y）
```

## 评估者类型

### 1. 基于代码的评估者
使用代码进行确定性检查：
```bash
# 检查文件是否包含预期模式
grep -q "export function handleAuth" src/auth.ts && echo "PASS" || echo "FAIL"

# 检查测试是否通过
npm test -- --testPathPattern="auth" && echo "PASS" || echo "FAIL"

# 检查构建是否成功
npm run build && echo "PASS" || echo "FAIL"
```

### 2. 基于模型的评估者
使用 Claude 评估自由格式输出：
```markdown
[MODEL GRADER PROMPT]
请评估以下代码变更：
1. 是否解决了描述的问题？
2. 是否结构化？
3. 边缘情况是否处理？
4. 错误处理是否适当？

评分: 1-5（1=差，5=优秀）
理由: [说明]
```

### 3. 人类评估者
标记需要手动审查的内容：
```markdown
[HUMAN REVIEW REQUIRED]
变更内容: 变更内容的描述
理由: 需要人类审查的原因
风险等级: LOW/MEDIUM/HIGH
```

## 指标

### pass@k
"k次尝试中至少成功一次"
- pass@1: 首次尝试的成功率
- pass@3: 3次内成功
- 一般目标: pass@3 > 90%

### pass^k
"k次尝试全部成功"
- 更高的可靠性标准
- pass^3: 连续3次成功
- 用于关键路径

## 评估工作流程

### 1. 定义（编码前）
```markdown
## 评估定义: feature-xyz

### 能力评估
1. 能创建新用户账户
2. 能验证邮箱格式
3. 能安全哈希密码

### 回归评估
1. 现有登录继续正常工作
2. 会话管理未变更
3. 登出流程保持不变

### 成功指标
- 能力评估 pass@3 > 90%
- 回归评估 pass^3 = 100%
```

### 2. 实现
编写通过定义评估的代码。

### 3. 评估
```bash
# 运行能力评估
[运行各能力评估并记录 PASS/FAIL]

# 运行回归评估
npm test -- --testPathPattern="existing"

# 生成报告
```

### 4. 报告
```markdown
评估报告: feature-xyz
========================

能力评估:
  create-user:     PASS (pass@1)
  validate-email:  PASS (pass@2)
  hash-password:   PASS (pass@1)
  总体:            3/3 成功

回归评估:
  login-flow:      PASS
  session-mgmt:    PASS
  logout-flow:     PASS
  总体:            3/3 成功

指标:
  pass@1: 67% (2/3)
  pass@3: 100% (3/3)

状态: 准备好审查
```

## 集成模式

### 实现前
```
/eval define feature-name
```
在 `.claude/evals/feature-name.md` 创建评估定义文件

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

## 保存评估

在项目内保存评估：
```
.claude/
  evals/
    feature-xyz.md      # 评估定义
    feature-xyz.log     # 评估执行历史
    baseline.json       # 回归基线
```

## 最佳实践

1. **编码前定义评估** - 强制明确思考成功标准
2. **频繁运行评估** - 及早发现回归
3. **随时间跟踪 pass@k** - 监控可靠性趋势
4. **尽可能使用代码评估者** - 确定性 > 概率性
5. **安全性需要人类审查** - 不要完全自动化安全检查
6. **保持评估快速** - 慢的评估不会被执行
7. **评估与代码一起版本控制** - 评估是一等工件

## 示例：添加认证

```markdown
## EVAL: add-authentication

### 阶段 1: 定义（10分钟）
能力评估:
- [ ] 用户可以通过邮箱/密码注册
- [ ] 用户可以使用有效凭证登录
- [ ] 无效凭证被适当的错误拒绝
- [ ] 会话在页面刷新后持续存在
- [ ] 登出清除会话

回归评估:
- [ ] 公开路由仍然可访问
- [ ] API 响应未变更
- [ ] 数据库架构兼容

### 阶段 2: 实现（可变）
[编写代码]

### 阶段 3: 评估
运行: /eval check add-authentication

### 阶段 4: 报告
评估报告: add-authentication
==============================
能力: 5/5 成功（pass@3: 100%）
回归: 3/3 成功（pass^3: 100%）
状态: 可发布
```
