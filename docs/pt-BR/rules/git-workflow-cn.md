# Git 工作流程

## 提交消息格式
```
<tipo>: <descrição>

<corpo opcional>
```

类型：feat, fix, refactor, docs, test, chore, perf, ci

注意：通过 ~/.claude/settings.json 全局禁用了归因功能。

## 拉取请求工作流程

创建 PR 时：
1. 分析完整的提交历史（不仅是最后一次提交）
2. 使用 `git diff [branch-base]...HEAD` 查看所有变更
3. 起草全面的 PR 摘要
4. 包含带有 TODO 列表的测试计划
5. 如果是新分支，使用 `-u` 标志进行推送

> 关于 Git 操作之前的完整开发流程（规划、TDD、代码审查），
> 请参阅 [development-workflow.md](./development-workflow.md)。
