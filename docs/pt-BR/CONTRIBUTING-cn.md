# 为 Everything Claude Code 贡献

感谢您想要贡献！这个仓库是 Claude Code 用户的社区资源。

## 目录

- [我们寻求的贡献](#我们寻求的贡献)
- [快速开始](#快速开始)
- [贡献 Skills](#贡献-skills)
- [贡献 Agentes](#贡献-agentes)
- [贡献 Hooks](#贡献-hooks)
- [贡献 Comandos](#贡献-comandos)
- [MCP 和文档（例如：Context7）](#mcp-和文档例如context7)
- [多平台和翻译](#多平台和翻译)
- [Pull Request 流程](#pull-request-流程)

---

## 我们寻求的贡献

### Agentes
擅长处理特定任务的新代理：
- 语言特定审查员（Python, Go, Rust）
- 框架专家（Django, Rails, Laravel, Spring）
- DevOps 专家（Kubernetes, Terraform, CI/CD）
- 领域专家（ML pipelines, 数据工程, 移动端）

### Skills
工作流定义和领域知识：
- 语言最佳实践
- 框架模式
- 测试策略
- 架构指南

### Hooks
有用的自动化：
- lint/格式化 hooks
- 安全检查
- 验证 hooks
- 通知 hooks

### Comandos
调用有用工作流的斜杠命令：
- 部署命令
- 测试命令
- 代码生成命令

---

## 快速开始

```bash
# 1. Fork 和 clone
gh repo fork affaan-m/everything-claude-code --clone
cd everything-claude-code

# 2. 创建分支
git checkout -b feat/minha-contribuicao

# 3. 添加您的贡献（请参阅以下部分）

# 4. 本地测试
cp -r skills/minha-skill ~/.claude/skills/  # 用于 skills
# 然后使用 Claude Code 测试

# 5. 提交 PR
git add . && git commit -m "feat: adicionar minha-skill" && git push -u origin feat/minha-contribuicao
```

---

## 贡献 Skills

Skills 是 Claude Code 根据上下文加载的知识模块。

### 目录结构

```
skills/
└── nome-da-sua-skill/
    └── SKILL.md
```

### SKILL.md 模板

```markdown
---
name: nome-da-sua-skill
description: 在技能列表中显示的简短描述
origin: ECC
---

# 您的技能标题

本技能涵盖内容的简要概述。

## 核心概念

解释关键模式和指南。

## 代码示例

```typescript
// 包含经过测试的实用示例
function exemplo() {
  // 注释良好的代码
}
```

## 最佳实践

- 可操作的指南
- 该做什么和不该做什么
- 需要避免的常见陷阱

## 何时使用

描述此技能适用的场景。
```

### Skill 检查清单

- [ ] 专注于一个领域/技术
- [ ] 包含实用的代码示例
- [ ] 500行以下
- [ ] 使用清晰的章节标题
- [ ] 已使用 Claude Code 测试

### Skills 示例

| Skill | 用途 |
|-------|------|
| `coding-standards/` | TypeScript/JavaScript 标准 |
| `frontend-patterns/` | React 和 Next.js 最佳实践 |
| `backend-patterns/` | API 和数据库模式 |
| `security-review/` | 安全检查清单 |

---

## 贡献 Agentes

Agentes 是通过 Task 工具调用的专业助手。

### 文件位置

```
agents/nome-do-seu-agente.md
```

### Agente 模板

```markdown
---
name: nome-do-seu-agente
description: 此代理的功能以及 Claude 何时应调用它。请具体说明！
tools: ["Read", "Write", "Edit", "Bash", "Grep", "Glob"]
model: sonnet
---

您是 [角色] 专家。

## 您的角色

- 主要职责
- 次要职责
- 您不做的事情（限制）

## 工作流程

### 步骤 1：理解
您如何处理任务。

### 步骤 2：执行
您如何完成工作。

### 步骤 3：验证
您如何验证结果。

## 输出格式

您返回给用户的内容。

## 示例

### 示例：[场景]
输入：[用户提供的内容]
操作：[您要做的事情]
输出：[您返回的内容]
```

### Agente 字段

| 字段 | 描述 | 选项 |
|------|------|------|
| `name` | 小写，使用连字符 | `code-reviewer` |
| `description` | 用于决定何时调用 | 请具体说明！ |
| `tools` | 仅使用必要的工具 | `Read, Write, Edit, Bash, Grep, Glob, WebFetch, Task` |
| `model` | 复杂程度级别 | `haiku` (简单), `sonnet` (编码), `opus` (复杂) |

### Agentes 示例

| Agente | 用途 |
|--------|------|
| `tdd-guide.md` | 测试驱动开发 |
| `code-reviewer.md` | 代码审查 |
| `security-reviewer.md` | 安全扫描 |
| `build-error-resolver.md` | 构建错误修复 |

---

## 贡献 Hooks

Hooks 是由 Claude Code 事件触发的自动行为。

### 文件位置

```
hooks/hooks.json
```

### Hook 类型

| 类型 | 触发器 | 用例 |
|------|--------|------|
| `PreToolUse` | 工具执行前 | 验证、警告、阻止 |
| `PostToolUse` | 工具执行后 | 格式化、检查、通知 |
| `SessionStart` | 会话开始 | 加载上下文 |
| `Stop` | 会话结束 | 清理、审计 |

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

### Hook 检查清单

- [ ] Matcher 具体（不过于宽泛）
- [ ] 包含清晰的错误/信息消息
- [ ] 使用正确的退出代码（`exit 1` 阻止，`exit 0` 允许）
- [ ] 经过全面测试
- [ ] 有描述

---

## 贡献 Comandos

Comandos 是用户使用 `/nome-do-comando` 调用的操作。

### 文件位置

```
commands/seu-comando.md
```

### Comando 模板

```markdown
---
description: 在 /help 中显示的简短描述
---

# 命令名称

## 用途

此命令的功能。

## 使用

```
/seu-comando [args]
```

## 工作流程

1. 第一步
2. 第二步
3. 最后一步

## 输出

用户收到的内容。
```

---

## MCP 和文档（例如：Context7）

Skills 和 agentes 可以使用 **MCP (Model Context Protocol)** 工具来获取最新数据，而不是仅依赖训练数据。这对于文档特别有用。

- **Context7** 是一个 MCP 服务器，提供 `resolve-library-id` 和 `query-docs`。当用户询问有关库、框架或 API 的问题时使用，以便响应反映最新文档。
- 当贡献依赖实时文档的 **skills** 时，请描述如何使用相关的 MCP 工具。
- 当贡献回答文档/API 问题的 **agentes** 时，请在代理的工具中包含 Context7 的 MCP 工具名称。

---

## 多平台和翻译

### Skills 子集（Codex 和 Cursor）

ECC 附带用于其他 harnesse 的 skills 子集：

- **Codex:** `.agents/skills/` — 在 `agents/openai.yaml` 中列出的 skills 由 Codex 加载。
- **Cursor:** `.cursor/skills/` — 包含用于 Cursor 的 skills 子集。

当 **添加应该在 Codex 或 Cursor 中可用的新 skill** 时：

1. 像往常一样在 `skills/nome-da-sua-skill/` 中添加 skill。
2. 如果应该在 **Codex** 中可用，请在 `.agents/skills/` 中添加，并确保在必要时在 `agents/openai.yaml` 中引用。
3. 如果应该在 **Cursor** 中可用，请在 `.cursor/skills/` 中添加。

### 翻译

翻译位于 `docs/` 中（例如：`docs/zh-CN`, `docs/zh-TW`, `docs/ja-JP`, `docs/ko-KR`, `docs/pt-BR`）。如果您修改了已翻译的 agentes、comandos 或 skills，请考虑更新相应的翻译文件或创建 issue。

---

## Pull Request 流程

### 1. PR 标题格式

```
feat(skills): 添加 skill rust-patterns
feat(agents): 添加代理 api-designer
feat(hooks): 添加 hook auto-format
fix(skills): 更新 React 模式
docs: 改进贡献指南
docs(pt-BR): 添加巴西葡萄牙语翻译
```

### 2. PR 描述

```markdown
## 摘要
您正在添加什么以及为什么。

## 类型
- [ ] Skill
- [ ] Agente
- [ ] Hook
- [ ] Comando
- [ ] 文档 / 翻译

## 测试
您如何测试的。

## 检查清单
- [ ] 遵循格式指南
- [ ] 已使用 Claude Code 测试
- [ ] 无敏感信息（API 密钥，路径）
- [ ] 清晰的描述
```

### 3. 审查流程

1. 维护者在 48 小时内审查
2. 如果有反馈请处理
3. 一旦批准，合并到 main

---

## 指南

### 应该做
- 保持贡献专注和模块化
- 包含清晰的描述
- 提交前测试
- 遵循现有模式
- 记录依赖项

### 不应该做
- 包含敏感数据（API 密钥，令牌，路径）
- 添加过于复杂或小众的配置
- 提交未测试的贡献
- 创建现有功能的重复项

---

## 文件命名

- 使用小写和连字符：`python-reviewer.md`
- 描述性：`tdd-workflow.md` 而非 `workflow.md`
- 名称与文件名匹配

---

## 有疑问？

- **Issues:** [github.com/affaan-m/everything-claude-code/issues](https://github.com/affaan-m/everything-claude-code/issues)
- **X/Twitter:** [@affaanmustafa](https://x.com/affaanmustafa)

---

感谢您的贡献！让我们一起构建一个出色的资源。
