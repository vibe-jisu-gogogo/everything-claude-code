# 用户级 CLAUDE.md 示例

这是用户级 CLAUDE.md 文件的示例。放置在 `~/.claude/CLAUDE.md`。

用户级配置全局应用于所有项目。用于：
- 个人代码偏好
- 您始终希望应用的通用规则
- 模块化规则的链接

---

## 核心理念

您是 Claude Code。我使用专门的 agents 和 skills 来处理复杂任务。

**关键原则：**
1. **Agent-First**：将复杂工作委托给专门的 agents
2. **并行执行**：尽可能使用 Task 工具与多个 agents
3. **先规划后执行**：对复杂操作使用 Plan Mode
4. **Test-Driven**：在实现之前编写测试
5. **Security-First**：绝不妥协安全性

---

## 模块化规则

`~/.claude/rules/` 中的详细指南：

| Rule File | Contents |
|-----------|----------|
| security.md | Security checks, secret management |
| coding-style.md | Immutability, file organization, error handling |
| testing.md | TDD workflow, 80% coverage requirement |
| git-workflow.md | Commit format, PR workflow |
| agents.md | Agent orchestration, when to use which agent |
| patterns.md | API response, repository patterns |
| performance.md | Model selection, context management |
| hooks.md | Hooks System |

---

## 可用 Agents

位于 `~/.claude/agents/`：

| Agent | Purpose |
|-------|---------|
| planner | Feature implementation planning |
| architect | System design and architecture |
| tdd-guide | Test-driven development |
| code-reviewer | Code review for quality/security |
| security-reviewer | Security vulnerability analysis |
| build-error-resolver | Build error resolution |
| e2e-runner | Playwright E2E testing |
| refactor-cleaner | Dead code cleanup |
| doc-updater | Documentation updates |

---

## 个人偏好

### 隐私
- 始终匿名化日志；绝不粘贴机密信息（API keys/tokens/passwords/JWTs）
- 在分享前检查输出 - 删除任何敏感数据

### 代码风格
- 代码、注释或文档中不使用 emoji
- 优先使用不可变性 - 永不修改对象或数组
- 多个小文件而非少量大文件
- 典型文件 200-400 行，最大 800 行

### Git
- Conventional commits: `feat:`, `fix:`, `refactor:`, `docs:`, `test:`
- 提交前始终在本地测试
- 小而专注的 commits

### 测试
- TDD: 先写测试
- 最低 80% 覆盖率
- 关键流程需要 Unit + integration + E2E

### 知识捕获
- 个人调试笔记、偏好和临时上下文 → auto memory
- 团队/项目知识（架构决策、API 变更、实施运行手册）→ 遵循项目中已有的文档结构
- 如果当前任务已经产生相关的文档/注释/示例，不要在其他地方重复相同知识
- 如果项目中没有明显的文档位置，在创建顶层新文档之前询问

---

## 编辑器集成

我使用 Zed 作为主要编辑器：
- Agent Panel 用于文件跟踪
- CMD+Shift+R 用于 command palette
- 启用 Vim mode

---

## 成功指标

您成功的条件：
- 所有测试通过（80%+ 覆盖率）
- 无安全漏洞
- 代码可读且可维护
- 满足用户需求

---

**理念**：Design agent-first, 并行执行, 先规划后行动, 先测试后编码, 始终安全。
