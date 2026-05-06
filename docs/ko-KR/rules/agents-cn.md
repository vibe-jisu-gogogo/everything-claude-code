# 代理编排

## 可用代理

位于 `~/.claude/agents/`：

| Agent | 用途 | 使用时机 |
|---------|------|----------|
| planner | 实现计划 | 复杂功能、重构 |
| architect | 系统设计 | 架构决策 |
| tdd-guide | 测试驱动开发 | 新功能、Bug 修复 |
| code-reviewer | 代码审查 | 代码编写后 |
| security-reviewer | 安全分析 | 提交前 |
| build-error-resolver | 构建错误修复 | 构建失败时 |
| e2e-runner | E2E 测试 | 核心用户流程 |
| database-reviewer | 数据库 schema/query 审查 | Schema 设计、查询优化 |
| go-reviewer | Go 代码审查 | Go 代码编写或修改后 |
| go-build-resolver | Go 构建错误修复 | `go build` 或 `go vet` 失败时 |
| refactor-cleaner | 清理未使用代码 | 代码维护 |
| doc-updater | 文档管理 | 文档更新 |

## 立即使用 Agent

无需用户提示：
1. 复杂功能请求 - 使用 **planner** Agent
2. 代码编写/修改后 - 使用 **code-reviewer** Agent
3. Bug 修复或新功能 - 使用 **tdd-guide** Agent
4. 架构决策 - 使用 **architect** Agent

## 并行 Task 执行

独立任务始终使用并行 Task 执行：

```markdown
# 好：并行执行
并行运行 3 个 Agent：
1. Agent 1：认证模块安全分析
2. Agent 2：缓存系统性能审查
3. Agent 3：工具类型检查

# 差：不必要的顺序执行
先 Agent 1，然后 Agent 2，再然后 Agent 3
```

## 多视角分析

复杂问题使用角色分离 Subagent：
- 事实验证 Reviewer
- 资深工程师
- 安全专家
- 一致性检查者
- 重复检查者
