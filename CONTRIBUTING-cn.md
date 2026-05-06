# 为 Everything Claude Code 贡献代码

感谢您愿意贡献代码！本仓库是面向 Claude Code 用户的社区资源。

## 目录

- [我们欢迎的贡献类型](#what-were-looking-for)
- [快速开始](#quick-start)
- [贡献 Skill](#contributing-skills)
- [Skill 适配政策](#skill-adaptation-policy)
- [贡献 Agent](#contributing-agents)
- [贡献 Hook](#contributing-hooks)
- [贡献 Command](#contributing-commands)
- [MCP 与文档（例如 Context7）](#mcp-and-documentation-eg-context7)
- [跨 Harness 适配与翻译](#cross-harness-and-translations)
- [Pull Request 流程](#pull-request-process)

---

## 我们欢迎的贡献类型

### Agent
擅长处理特定任务的新 Agent：
- 特定语言审阅者（Python、Go、Rust）
- 框架专家（Django、Rails、Laravel、Spring）
- DevOps 专家（Kubernetes、Terraform、CI/CD）
- 领域专家（ML 流水线、数据工程、移动端）

### Skill
工作流定义与领域知识：
- 语言最佳实践
- 框架模式
- 测试策略
- 架构指南

### Hook
实用的自动化功能：
- 代码检查/格式化 Hook
- 安全检查
- 验证 Hook
- 通知 Hook

### Command
可调用实用工作流的斜杠命令：
- 部署命令
- 测试命令
- 代码生成命令

---

## 快速开始

```bash
# 1. Fork 并克隆仓库
gh repo fork affaan-m/everything-claude-code --clone
cd everything-claude-code

# 2. 创建分支
git checkout -b feat/my-contribution

# 3. 添加您的贡献（参见下文各章节）

# 4. 本地测试
cp -r skills/my-skill ~/.claude/skills/  # 适用于 Skill
# 然后在 Claude Code 中测试

# 5. 提交 PR
git add . && git commit -m "feat: add my-skill" && git push -u origin feat/my-contribution
```

---

## 贡献 Skill

Skill 是 Claude Code 根据上下文加载的知识模块。

> **完整指南：** 关于如何创建高效 Skill 的详细指导，请参见 [Skill 开发指南](docs/SKILL-DEVELOPMENT-GUIDE.md)，其中包含：
> - Skill 架构与分类
> - 编写带示例的有效内容
> - 最佳实践与通用模式
> - 测试与验证
> - 完整示例库

### 目录结构

```
skills/
└── your-skill-name/
    └── SKILL.md
```

### SKILL.md 模板

```markdown
---
name: your-skill-name
description: Skill 列表中显示的简短描述，用于自动激活
origin: ECC
---

# 您的 Skill 标题

关于本 Skill 涵盖内容的简要概述。

## 何时激活

描述 Claude 应使用本 Skill 的场景。这对于自动激活至关重要。

## 核心概念

解释关键模式与指南。

## 代码示例

```typescript
// 包含实用的、经过测试的示例
function example() {
  // 注释完善的代码
}
```

## 反模式

通过示例说明不应该做什么。

## 最佳实践

- 可执行的指导原则
- 应该做和不应该做的事
- 需要避免的常见陷阱

## 相关 Skill

链接到互补的 Skill（例如 `related-skill-1`、`related-skill-2`）。
```

### Skill 分类

| 分类 | 用途 | 示例 |
|----------|---------|----------|
| **语言标准** | 惯用写法、约定、最佳实践 | `python-patterns`、`golang-patterns` |
| **框架模式** | 特定框架指导 | `django-patterns`、`nextjs-patterns` |
| **工作流** | 分步流程 | `tdd-workflow`、`refactoring-workflow` |
| **领域知识** | 专业领域 | `security-review`、`api-design` |
| **工具集成** | 工具/库使用方法 | `docker-patterns`、`supabase-patterns` |
| **模板** | 项目专属 Skill 模板 | `docs/examples/project-guidelines-template.md` |

### Skill 适配政策

如果您要从其他仓库、插件、harness 或个人提示词包迁移创意，请在提交 PR 前阅读 [Skill 适配政策](docs/skill-adaptation-policy.md)。

简短版本：
- 复制底层创意，而非外部产品标识
- 当 ECC 对功能进行实质性修改或扩展时，重命名 Skill
- 优先使用 ECC 原生的 rule、skill、script 和 MCP，而非新增默认第三方依赖
- 不要提交主要价值只是告知用户安装未审核包的 Skill

### Skill 检查清单

- [ ] 聚焦于单个领域/技术（不要过于宽泛）
- [ ] 包含用于自动激活的「何时激活」章节
- [ ] 包含实用的、可直接复制粘贴的代码示例
- [ ] 展示反模式（不应该做什么）
- [ ] 内容不超过 500 行（最多 800 行）
- [ ] 使用清晰的章节标题
- [ ] 已在 Claude Code 中测试
- [ ] 链接到相关 Skill
- [ ] 不包含敏感数据（API 密钥、令牌、路径）

### Skill 示例

| Skill | 分类 | 用途 |
|-------|----------|---------|
| `coding-standards/` | 语言标准 | TypeScript/JavaScript 模式 |
| `frontend-patterns/` | 框架模式 | React 和 Next.js 最佳实践 |
| `backend-patterns/` | 框架模式 | API 和数据库模式 |
| `security-review/` | 领域知识 | 安全检查清单 |
| `tdd-workflow/` | 工作流 | 测试驱动开发流程 |
| `docs/examples/project-guidelines-template.md` | 模板 | 项目专属 Skill 模板 |

---

## 贡献 Agent

Agent 是通过 Task 工具调用的专业助手。

### 文件位置

```
agents/your-agent-name.md
```

### Agent 模板

```markdown
---
name: your-agent-name
description: 本 Agent 的功能，以及 Claude 应何时调用它。请务必具体！
tools: ["Read", "Write", "Edit", "Bash", "Grep", "Glob"]
model: sonnet
---

你是一名 [角色] 专家。

## 你的角色

- 主要职责
- 次要职责
- 你不负责的内容（边界）

## 工作流

### 步骤 1：理解需求
你如何处理任务。

### 步骤 2：执行
你如何完成工作。

### 步骤 3：验证
你如何验证结果。

## 输出格式

你返回给用户的内容。

## 示例

### 示例：[场景]
输入：[用户提供的内容]
操作：[你执行的操作]
输出：[你返回的内容]
```

### Agent 字段

| 字段 | 描述 | 可选值 |
|-------|-------------|---------|
| `name` | 小写，使用连字符分隔 | `code-reviewer` |
| `description` | 用于决定何时调用 | 请务必具体！ |
| `tools` | 仅包含所需工具 | `Read、Write、Edit、Bash、Grep、Glob、WebFetch、Task`，如果 Agent 使用 MCP，还可包含 MCP 工具名称（例如 `mcp__context7__resolve-library-id`、`mcp__context7__query-docs`） |
| `model` | 复杂度级别 | `haiku`（简单任务）、`sonnet`（编码任务）、`opus`（复杂任务） |

### Agent 示例

| Agent | 用途 |
|-------|---------|
| `tdd-guide.md` | 测试驱动开发 |
| `code-reviewer.md` | 代码审阅 |
| `security-reviewer.md` | 安全扫描 |
| `build-error-resolver.md` | 修复构建错误 |

---

## 贡献 Hook

Hook 是由 Claude Code 事件触发的自动化行为。

### 文件位置

```
hooks/hooks.json
```

### Hook 类型

| 类型 | 触发时机 | 使用场景 |
|------|---------|----------|
| `PreToolUse` | 工具运行前 | 验证、警告、阻止 |
| `PostToolUse` | 工具运行后 | 格式化、检查、通知 |
| `SessionStart` | 会话开始时 | 加载上下文 |
| `Stop` | 会话结束时 | 清理、审计 |

### Hook 格式

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "tool == \"Bash\" && tool_input.command matches \"rm -rf /\"",
        "hooks": [
          {
            "type": "command",
            "command": "echo '[Hook] 已阻止：危险命令' && exit 1"
          }
        ],
        "description": "阻止危险的 rm 命令"
      }
    ]
  }
}
```

### Matcher 语法

```javascript
// 匹配特定工具
tool == "Bash"
tool == "Edit"
tool == "Write"

// 匹配输入模式
tool_input.command matches "npm install"
tool_input.file_path matches "\\.tsx?$"

// 组合条件
tool == "Bash" && tool_input.command matches "git push"
```

### Hook 示例

```json
// 阻止在 tmux 外运行开发服务器
{
  "matcher": "tool == \"Bash\" && tool_input.command matches \"npm run dev\"",
  "hooks": [{"type": "command", "command": "echo '请使用 tmux 运行开发服务器' && exit 1"}],
  "description": "确保开发服务器在 tmux 中运行"
}

// 编辑 TypeScript 后自动格式化
{
  "matcher": "tool == \"Edit\" && tool_input.file_path matches \"\\.tsx?$\"",
  "hooks": [{"type": "command", "command": "npx prettier --write \"$file_path\""}],
  "description": "编辑后自动格式化 TypeScript 文件"
}

// git push 前发出警告
{
  "matcher": "tool == \"Bash\" && tool_input.command matches \"git push\"",
  "hooks": [{"type": "command", "command": "echo '[Hook] 推送前请先检查代码变更'"}],
  "description": "推送前提醒检查变更"
}
```

### Hook 检查清单

- [ ] Matcher 规则具体（不过于宽泛）
- [ ] 包含清晰的错误/提示信息
- [ ] 使用正确的退出码（`exit 1` 阻止操作，`exit 0` 允许操作）
- [ ] 经过充分测试
- [ ] 包含描述信息

---

## 贡献 Command

Command 是用户通过 `/command-name` 调用的操作。

### 文件位置

```
commands/your-command.md
```

### Command 模板

```markdown
---
description: 在 /help 中显示的简短描述
---

# 命令名称

## 用途

本命令的功能。

## 用法

```
/your-command [参数]
```

## 工作流

1. 第一步
2. 第二步
3. 最后一步

## 输出

用户收到的内容。
```

### Command 示例

| Command | 用途 |
|---------|---------|
| `commit.md` | 创建 git commit |
| `code-review.md` | 审阅代码变更 |
| `tdd.md` | TDD 工作流 |
| `e2e.md` | E2E 测试 |

---

## MCP 与文档（例如 Context7）

Skill 和 Agent 可以使用 **MCP（Model Context Protocol）** 工具获取最新数据，而无需仅依赖训练数据。这对于文档场景尤其有用。

- **Context7** 是一个 MCP 服务器，提供 `resolve-library-id` 和 `query-docs` 接口。当用户询问关于库、框架或 API 的问题时使用它，这样回答就能反映最新的文档和代码示例。
- 贡献依赖实时文档的 **Skill** 时（例如安装配置、API 使用），请描述如何使用相关的 MCP 工具（例如先解析库 ID，再查询文档），并以 `documentation-lookup` Skill 或 Context7 作为模式参考。
- 贡献用于回答文档/API 问题的 **Agent** 时，请在 Agent 的 tools 列表中包含 Context7 MCP 工具名称（例如 `mcp__context7__resolve-library-id`、`mcp__context7__query-docs`），并记录解析 → 查询的工作流。
- **mcp-configs/mcp-servers.json** 包含 Context7 配置项；用户需要在其 harness（例如 Claude Code、Cursor）中启用它，才能使用 documentation-lookup Skill（位于 `skills/documentation-lookup/`）和 `/docs` 命令。

---

## 跨 Harness 适配与翻译

### Skill 子集（Codex 和 Cursor）

ECC 为其他 harness 提供 Skill 子集：
- **Codex:** `.agents/skills/` — `agents/openai.yaml` 中列出的 Skill 会被 Codex 加载。
- **Cursor:** `.cursor/skills/` — 为 Cursor 打包的 Skill 子集。

当您 **添加新 Skill** 并希望其在 Codex 或 Cursor 中可用时：
1. 照常将 Skill 添加到 `skills/your-skill-name/` 目录下。
2. 如果需要在 **Codex** 中可用，将其添加到 `.agents/skills/`（复制 Skill 目录或添加引用），并确保在需要时在 `agents/openai.yaml` 中引用它。
3. 如果需要在 **Cursor** 中可用，按照 Cursor 的目录结构将其添加到 `.cursor/skills/` 下。

请参考这些目录中现有的 Skill 来了解预期的结构。这些子集的同步是手动完成的；如果您更新了它们，请在 PR 中说明。

### 翻译

翻译文件位于 `docs/` 目录下（例如 `docs/zh-CN`、`docs/zh-TW`、`docs/ja-JP`）。如果您修改了已翻译的 Agent、Command 或 Skill，请考虑更新对应的翻译文件，或者提交 Issue 以便维护者或翻译人员进行更新。

---

## Pull Request 流程

### 1. PR 标题格式

```
feat(skills): add rust-patterns skill
feat(agents): add api-designer agent
feat(hooks): add auto-format hook
fix(skills): update React patterns
docs: improve contributing guide
```

### 2. PR 描述

```markdown
## 概述
您添加的内容及其原因。

## 类型
- [ ] Skill
- [ ] Agent
- [ ] Hook
- [ ] Command

## 测试说明
您如何测试本次贡献。

## 检查清单
- [ ] 遵循格式规范
- [ ] 已在 Claude Code 中测试
- [ ] 不包含敏感信息（API 密钥、路径）
- [ ] 描述清晰
```

### 3. 审阅流程

1. 维护者会在 48 小时内完成审阅
2. 根据要求处理反馈意见
3. 审批通过后合并到 main 分支

---

## 指导原则

### 应该做
- 保持贡献内容聚焦且模块化
- 包含清晰的描述
- 提交前进行测试
- 遵循现有模式
- 为依赖添加文档说明

### 不应该做
- 包含敏感数据（API 密钥、令牌、路径）
- 添加过于复杂或小众的配置
- 提交未测试的贡献
- 创建与现有功能重复的内容

---

## 文件命名

- 使用小写字母和连字符：`python-reviewer.md`
- 命名要描述清晰：使用 `tdd-workflow.md` 而非 `workflow.md`
- 名称与文件名保持一致

---

## 有问题？

- **Issues：** [github.com/affaan-m/everything-claude-code/issues](https://github.com/affaan-m/everything-claude-code/issues)
- **X/Twitter：** [@affaanmustafa](https://x.com/affaanmustafa)

---

感谢您的贡献！让我们共同打造一个优秀的资源库。