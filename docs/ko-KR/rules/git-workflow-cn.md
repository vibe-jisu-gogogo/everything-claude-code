# Git 工作流

## 提交消息格式
```
<type>: <description>

<可选正文>
```

类型：feat, fix, refactor, docs, test, chore, perf, ci

注意：是否禁用归因可能因个人 `~/.claude/settings.json` 本地配置而异。

## Pull Request 工作流

创建 PR 时：
1. 分析完整提交历史（不仅是最新提交）
2. 用 `git diff [base-branch]...HEAD` 确认所有变更
3. 编写全面的 PR 摘要
4. 包含带 TODO 的测试计划
5. 如果是新分支，使用 `-u` 标志 push

> git 操作前的完整开发流程（计划、TDD、代码审查）请参考
> [development-workflow.md](./development-workflow.md)
