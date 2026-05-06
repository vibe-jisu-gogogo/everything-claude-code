# 用户级别 CLAUDE.md 示例

这是用户级别的 CLAUDE.md 文件示例。请放置在 `~/.claude/CLAUDE.md`。

用户级别设置会全局应用到所有项目。请用于以下用途：
- 个人编码偏好设置
- 希望始终应用的通用规则
- 模块化规则文件链接

---

## 核心哲学

你是 Claude Code。我使用针对复杂任务的 specialized agent 和 skill。

**核心原则：**
1. **Agent 优先**：复杂任务委托给 specialized agent
2. **并行执行**：尽可能使用 Task tool 同时运行多个 agent
3. **执行前计划**：复杂任务使用 Plan Mode
4. **测试驱动**：实现前编写测试
5. **安全优先**：在安全方面绝不妥协

---

## 模块化规则

详细指南位于 `~/.claude/rules/`：

| 规则文件 | 内容 |
|-----------|------|
| security.md | 安全检查，密钥管理 |
| coding-style.md | 不变性，文件结构，错误处理 |
| testing.md | TDD workflow，80% 覆盖率要求 |
| git-workflow.md | 提交格式，PR workflow |
| agents.md | Agent orchestration，根据情况选择 agent |
| patterns.md | API 响应，repository 模式 |
| performance.md | 模型选择，上下文管理 |
| hooks.md | Hooks 系统 |

---

## 可用 Agent

位于 `~/.claude/agents/`：

| Agent | 用途 |
|-------|------|
| planner | 功能实现计划制定 |
| architect | 系统设计和架构 |
| tdd-guide | 测试驱动开发 |
| code-reviewer | 质量/安全代码审查 |
| security-reviewer | 安全漏洞分析 |
| build-error-resolver | 构建错误解决 |
| e2e-runner | Playwright E2E 测试 |
| refactor-cleaner | 不必要代码清理 |
| doc-updater | 文档更新 |

---

## 个人偏好设置

### 隐私保护
- 始终删除日志，绝不粘贴密钥（API key/token/password/JWT）
- 共享前检查输出内容，删除敏感数据

### 代码风格
- 代码、注释、文档中禁止使用 emoji
- 偏好不变性 - 不直接修改对象或数组
- 偏好多个小文件而非少数大文件
- 通常 200-400 行，每个文件最多 800 行

### Git
- Conventional commits：`feat:`、`fix:`、`refactor:`、`docs:`、`test:`
- 提交前始终在本地测试
- 小而集中的提交

### 测试
- TDD：先编写测试
- 最低 80% 覆盖率
- 对核心流程进行单元 + 集成 + E2E 测试

### 知识积累
- 个人调试笔记、偏好设置、临时上下文 → auto memory
- 团队/项目知识（架构决策、API 变更、实现 runbook）→ 遵循项目现有文档结构
- 如果当前任务中已创建相关文档、注释、示例，不要在其他地方重复相同知识
- 如果没有合适的项目文档位置，在创建新的顶级文档前先询问

---

## 编辑器集成

我使用 Zed 作为默认编辑器：
- 用于文件跟踪的 Agent Panel
- 使用 CMD+Shift+R 打开命令面板
- 启用 Vim 模式

---

## 成功标准

满足以下条件即为成功：
- 所有测试通过（80% 以上覆盖率）
- 无安全漏洞
- 代码易于阅读和维护
- 满足用户需求

---

**哲学**：Agent 优先设计、并行执行、执行前计划、代码前测试、始终安全优先。
