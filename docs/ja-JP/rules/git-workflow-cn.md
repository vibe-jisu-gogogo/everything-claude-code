# Git 工作流程

## 提交消息格式

```
<type>: <description>

<optional body>
```

类型: feat, fix, refactor, docs, test, chore, perf, ci

注意: Attribution 已在 ~/.claude/settings.json 中全局禁用。

## Pull Request 工作流程

创建 PR 时:
1. 分析完整的提交历史（不仅仅是最新提交）
2. 使用 `git diff [base-branch]...HEAD` 确认所有变更
3. 创建全面的 PR 摘要
4. 包含带 TODO 的测试计划
5. 如为新分支，使用 `-u` 标志 push

## 功能实现工作流程

1. **首先计划**
   - 使用 **planner** agent 创建实施计划
   - 确定依赖关系和风险
   - 分阶段实施

2. **TDD 方法**
   - 使用 **tdd-guide** agent
   - 先写测试（RED）
   - 实现代码通过测试（GREEN）
   - 重构（IMPROVE）
   - 确认 80%+ 覆盖率

3. **代码审查**
   - 编写代码后立即使用 **code-reviewer** agent
   - 处理 CRITICAL 和 HIGH 级别问题
   - 尽可能修复 MEDIUM 级别问题

4. **提交 & Push**
   - 详细的提交消息
   - 遵循 Conventional Commits 格式
