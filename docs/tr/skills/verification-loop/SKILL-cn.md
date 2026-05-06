---
name: verification-loop
description: "Claude Code sessions için kapsamlı doğrulama sistemi."
origin: ECC
---

# Verification Loop Skill

Claude Code 会话的综合验证系统。

## 何时使用

在以下情况下调用此 skill：
- 完成功能或重大代码更改后
- 创建 PR 之前
- 确保通过质量门禁时
- 重构之后

## 验证阶段

### 阶段 1：Build 验证
```bash
# 检查项目是否可以 build
npm run build 2>&1 | tail -20
# 或
pnpm build 2>&1 | tail -20
```

如果 build 失败，停止并修复后再继续。

### 阶段 2：类型检查
```bash
# TypeScript 项目
npx tsc --noEmit 2>&1 | head -30

# Python 项目
pyright . 2>&1 | head -30
```

报告所有类型错误。继续之前先修复关键错误。

### 阶段 3：Lint 检查
```bash
# JavaScript/TypeScript
npm run lint 2>&1 | head -30

# Python
ruff check . 2>&1 | head -30
```

### 阶段 4：测试套件
```bash
# 运行带 coverage 的测试
npm run test -- --coverage 2>&1 | tail -50

# 检查 coverage 阈值
# 目标：最低 80%
```

报告：
- 总测试数：X
- 通过：X
- 失败：X
- Coverage：%X

### 阶段 5：安全扫描
```bash
# 检查 secrets
grep -rn "sk-" --include="*.ts" --include="*.js" . 2>/dev/null | head -10
grep -rn "api_key" --include="*.ts" --include="*.js" . 2>/dev/null | head -10

# 检查 console.log
grep -rn "console.log" --include="*.ts" --include="*.tsx" src/ 2>/dev/null | head -10
```

### 阶段 6：Diff 审查
```bash
# 显示更改内容
git diff --stat
git diff HEAD~1 --name-only
```

审查每个更改的文件，检查：
- 不必要的更改
- 缺失的错误处理
- 潜在的 edge cases

## 输出格式

运行所有阶段后，生成验证报告：

```
验证报告
==================

Build:     [PASS/FAIL]
类型:      [PASS/FAIL] (X 错误)
Lint:      [PASS/FAIL] (X 警告)
测试:      [PASS/FAIL] (X/Y 通过，%Z coverage)
安全:      [PASS/FAIL] (X 问题)
Diff:      [X 文件已更改]

总体:      PR [就绪/未就绪]

需要修复的问题：
1. ...
2. ...
```

## 连续模式

对于长会话，每 15 分钟或重大更改后运行验证：

```markdown
设置心理检查点：
- 完成每个函数后
- 完成一个 component 后
- 进入下一个任务之前

运行：/verify
```

## 与 Hooks 集成

此 skill 补充了 PostToolUse hooks，但提供了更深入的验证。
Hooks 能即时捕获问题；此 skill 提供全面审查。
