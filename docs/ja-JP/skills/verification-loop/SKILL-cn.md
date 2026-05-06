# 验证循环技能

Claude Code 会话的综合验证系统。

## 使用时机

调用此技能：
- 完成功能或重要代码变更后
- 创建 PR 之前
- 需要确认质量门禁通过时
- 重构之后

## 验证阶段

### 阶段 1：构建验证
```bash
# 确认项目可以构建
npm run build 2>&1 | tail -20
# 或者
pnpm build 2>&1 | tail -20
```

如果构建失败，停止并在继续前修复。

### 阶段 2：类型检查
```bash
# TypeScript 项目
npx tsc --noEmit 2>&1 | head -30

# Python 项目
pyright . 2>&1 | head -30
```

报告所有类型错误。在继续前修复重要的错误。

### 阶段 3：Lint 检查
```bash
# JavaScript/TypeScript
npm run lint 2>&1 | head -30

# Python
ruff check . 2>&1 | head -30
```

### 阶段 4：测试套件
```bash
# 运行带覆盖率的测试
npm run test -- --coverage 2>&1 | tail -50

# 确认覆盖率阈值
# 目标：最低 80%
```

报告：
- 总测试数：X
- 成功：X
- 失败：X
- 覆盖率：X%

### 阶段 5：安全扫描
```bash
# 检查密钥
grep -rn "sk-" --include="*.ts" --include="*.js" . 2>/dev/null | head -10
grep -rn "api_key" --include="*.ts" --include="*.js" . 2>/dev/null | head -10

# 检查 console.log
grep -rn "console.log" --include="*.ts" --include="*.tsx" src/ 2>/dev/null | head -10
```

### 阶段 6：差异审查
```bash
# 显示变更内容
git diff --stat
git diff HEAD~1 --name-only
```

审查每个变更文件：
- 非预期变更
- 缺失的错误处理
- 潜在的边界情况

## 输出格式

执行所有阶段后，创建验证报告：

```
验证报告
==================

构建:     [成功/失败]
类型:     [成功/失败] (X 错误)
Lint:     [成功/失败] (X 警告)
测试:     [成功/失败] (X/Y 成功，Z% 覆盖率)
安全:     [成功/失败] (X 问题)
差异:     [X 文件变更]

综合:     PR 准备[完成/未完成]

需要修复的问题：
1. ...
2. ...
```

## 持续模式

对于长时间会话，每 15 分钟或主要变更后执行验证：

```markdown
设置心理检查点：
- 完成每个函数后
- 完成组件后
- 转移到下一个任务前

执行：/verify
```

## 与钩子的集成

此技能补充 PostToolUse 钩子，但提供更深入的验证。
钩子即时捕获问题；此技能提供综合审查。
