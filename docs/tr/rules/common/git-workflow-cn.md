# Git 工作流程

## Commit 消息格式
```
<type>: <description>

<optional body>
```

Types: feat, fix, refactor, docs, test, chore, perf, ci

注意：Attribution 功能通过 ~/.claude/settings.json 已在全局范围内禁用。

## Pull Request 工作流程

创建 PR 时：
1. 分析完整的 commit 历史（不仅仅是最后一个 commit）
2. 使用 `git diff [base-branch]...HEAD` 查看所有变更
3. 起草全面的 PR 摘要
4. 添加包含 TODO 项的测试计划
5. 如果是新分支，使用 `-u` flag 进行 push

> Git 操作前的完整开发流程（规划、TDD、代码审查）请参见
> [development-workflow.md](./development-workflow.md) 文件。
