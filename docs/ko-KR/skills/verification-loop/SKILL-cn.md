---
name: verification-loop
description: "Claude Code 会话的综合验证系统。"
origin: ECC
---

# 验证循环技能

Claude Code 会话的综合验证系统。

## 使用时机

在以下情况下调用此技能：
- 完成功能或主要代码变更后
- 创建 PR 之前
- 需要确认质量门是否通过时
- 重构后

## 验证步骤

### 步骤 1: 构建验证
```bash
# Check if project builds
npm run build 2>&1 | tail -20
# OR
pnpm build 2>&1 | tail -20
```

如果构建失败，停止并修复后再继续。

### 步骤 2: 类型检查
```bash
# TypeScript projects
npx tsc --noEmit 2>&1 | head -30

# Python projects
pyright . 2>&1 | head -30
```

报告所有类型错误。重要错误需在继续前修复。

### 步骤 3: Lint 检查
```bash
# JavaScript/TypeScript
npm run lint 2>&1 | head -30

# Python
ruff check . 2>&1 | head -30
```

### 步骤 4: 测试套件
```bash
# Run tests with coverage
npm run test -- --coverage 2>&1 | tail -50

# Check coverage threshold
# Target: 80% minimum
```

报告项目：
- 总测试数：X
- 通过：X
- 失败：X
- 覆盖率：X%

### 步骤 5: 安全扫描
```bash
# Check for secrets
grep -rn "sk-" --include="*.ts" --include="*.js" . 2>/dev/null | head -10
grep -rn "api_key" --include="*.ts" --include="*.js" . 2>/dev/null | head -10

# Check for console.log
grep -rn "console.log" --include="*.ts" --include="*.tsx" src/ 2>/dev/null | head -10
```

### 步骤 6: Diff 审查
```bash
# Show what changed
git diff --stat
git diff --name-only
git diff --cached --name-only
```

在每个变更文件中审查以下内容：
- 非预期的变更
- 缺失的错误处理
- 潜在的边界情况

## 输出格式

执行所有步骤后生成验证报告：

```
VERIFICATION REPORT
==================

Build:     [PASS/FAIL]
Types:     [PASS/FAIL] (X errors)
Lint:      [PASS/FAIL] (X warnings)
Tests:     [PASS/FAIL] (X/Y passed, Z% coverage)
Security:  [PASS/FAIL] (X issues)
Diff:      [X files changed]

Overall:   [READY/NOT READY] for PR

Issues to Fix:
1. ...
2. ...
```

## 连续模式

在长会话中，每 15 分钟或主要变更后执行验证：

```markdown
设置心理检查点：
- 完成每个函数后
- 完成一个组件后
- 进入下一个任务前

执行：/verify
```

## 与 Hook 集成

此技能补充了 PostToolUse Hook，但提供了更深入的验证。
Hook 立即捕获问题，此技能提供全面审查。
