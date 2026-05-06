# Agent 编排

## 可用 Agents

位于 `~/.claude/agents/`：

| Agent | 用途 | 何时使用 |
|--------|-----------|-------------|
| planner | 实现规划 | 复杂功能、重构 |
| architect | 系统设计 | 架构决策 |
| tdd-guide | 测试驱动开发 | 新功能、bug 修复 |
| code-reviewer | 代码审查 | 编写代码后 |
| security-reviewer | 安全分析 | 提交前 |
| build-error-resolver | 构建错误修复 | 构建失败时 |
| e2e-runner | E2E 测试 | 关键用户流程 |
| refactor-cleaner | 死代码清理 | 代码维护 |
| doc-updater | 文档 | 文档更新 |
| rust-reviewer | Rust 代码审查 | Rust 项目 |

## Agent 即时使用

无需用户提示：
1. 复杂功能请求 - 使用 **planner** agent
2. 刚编写/修改完代码 - 使用 **code-reviewer** agent
3. Bug 修复或新功能 - 使用 **tdd-guide** agent
4. 架构决策 - 使用 **architect** agent

## 任务并行执行

对于独立操作，始终使用 Task 并行执行：

```markdown
# 推荐：并行执行
同时启动 3 个 agents：
1. Agent 1：认证模块安全分析
2. Agent 2：缓存系统性能审查
3. Agent 3：工具类型检查

# 不推荐：不必要的顺序执行
先 agent 1，再 agent 2，再 agent 3
```

## 多视角分析

对于复杂问题，使用具有不同角色的 subagents：
- 事实审查员
- 高级工程师
- 安全专家
- 一致性审查员
- 冗余检查器
