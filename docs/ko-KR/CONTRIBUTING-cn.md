# Everything Claude Code 贡献指南

感谢您对贡献感兴趣！此仓库是为 Claude Code 用户提供的社区资源。

## 目录

- [我们寻找的内容](#我们寻找的内容)
- [快速开始](#快速开始)
- [贡献技能](#贡献技能)
- [贡献代理](#贡献代理)
- [贡献钩子](#贡献钩子)
- [贡献命令](#贡献命令)
- [Pull Request 流程](#pull-request-流程)

---

## 我们寻找的内容

### 代理
擅长处理特定任务的新代理：
- 语言特定审查员 (Python, Go, Rust)
- 框架专家 (Django, Rails, Laravel, Spring)
- DevOps 专家 (Kubernetes, Terraform, CI/CD)
- 领域专家 (ML 管道、数据工程、移动端)

### 技能
工作流定义和领域知识：
- 语言最佳实践
- 框架模式
- 测试策略
- 架构指南

### 钩子
有用的自动化：
- Linting/格式化钩子
- 安全检查
- 验证钩子
- 通知钩子

### 命令
调用有用工作流的斜杠命令：
- 部署命令
- 测试命令
- 代码生成命令

---

## 快速开始

```bash
# 1. Fork 并克隆
gh repo fork affaan-m/everything-claude-code --clone
cd everything-claude-code

# 2. 创建分支
git checkout -b feat/my-contribution

# 3. 添加贡献内容（参见以下章节）

# 4. 本地测试
cp -r skills/my-skill ~/.claude/skills/  # 对于技能
# 然后在 Claude Code 中测试

# 5. 提交 PR
git add . && git commit -m "feat: add my-skill" && git push -u origin feat/my-contribution
```

---

## 贡献技能

技能是 Claude Code 根据上下文加载的知识模块。

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
description: 显示在技能列表中的简短描述
origin: ECC
---

# 技能标题

此技能涵盖内容的简要概述。

## 核心概念

解释主要模式和指南。

## 代码示例

```typescript
// 包含实用且经过测试的示例
function example() {
  // 注释良好的代码
}
```

## 最佳实践

- 可执行的指南
- 应该做和不应该做的事情
- 避免常见错误

## 使用时机

解释此技能适用的场景。
```

### 技能检查清单

- [ ] 专注于一个领域/技术
- [ ] 包含实用的代码示例
- [ ] 少于 500 行
- [ ] 使用清晰的章节标题
- [ ] 在 Claude Code 中测试完成

### 技能示例

| 技能 | 用途 |
|------|------|
| `coding-standards/` | TypeScript/JavaScript 模式 |
| `frontend-patterns/` | React 和 Next.js 最佳实践 |
| `backend-patterns/` | API 和数据库模式 |
| `security-review/` | 安全检查清单 |

---

## 贡献代理

代理是通过 Task 工具调用的专业助手。

### 文件位置

```
agents/your-agent-name.md
```

### 代理模板

```markdown
---
name: your-agent-name
description: 此代理的功能以及 Claude 何时应调用它。请具体！
tools: ["Read", "Write", "Edit", "Bash", "Grep", "Glob"]
model: sonnet
---

你是 [角色] 专家。

## 角色

- 主要职责
- 次要职责
- 不做的事情（边界）

## 工作流

### 步骤 1：理解
如何处理任务。

### 步骤 2：执行
如何执行任务。

### 步骤 3：验证
如何验证结果。

## 输出格式

返回给用户的内容。

## 示例

### 示例：[场景]
输入：[用户提供的内容]
行动：[执行的内容]
输出：[返回的内容]
```

### 代理字段

| 字段 | 描述 | 选项 |
|------|------|------|
| `name` | 小写，连字符分隔 | `code-reviewer` |
| `description` | 用于确定何时调用 | 请具体！ |
| `tools` | 只包含必要的 | `Read, Write, Edit, Bash, Grep, Glob, WebFetch, Task` |
| `model` | 复杂度级别 | `haiku`（简单），`sonnet`（编码），`opus`（复杂） |

### 示例代理

| 代理 | 用途 |
|----------|------|
| `tdd-guide.md` | 测试驱动开发 |
| `code-reviewer.md` | 代码审查 |
| `security-reviewer.md` | 安全检查 |
| `build-error-resolver.md` | 构建错误修复 |

---

## 贡献钩子

钩子是由 Claude Code 事件触发的自动操作。

### 文件位置

```
hooks/hooks.json
```

### 钩子类型

| 类型 | 触发时机 | 使用场景 |
|------|-----------|----------|
| `PreToolUse` | 工具执行前 | 验证、警告、阻止 |
| `PostToolUse` | 工具执行后 | 格式化、检查、通知 |
| `SessionStart` | 会话开始时 | 上下文加载 |
| `Stop` | 会话结束时 | 清理、审计 |

### 钩子格式

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "tool == \"Bash\" && tool_input.command matches \"rm -rf /\"",
        "hooks": [
          {
            "type": "command",
            "command": "echo '[Hook] BLOCKED: Dangerous command' && exit 1"
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

### 钩子示例

```json
// 在 tmux 外阻止 dev 服务器
{
  "matcher": "tool == \"Bash\" && tool_input.command matches \"npm run dev\"",
  "hooks": [{"type": "command", "command": "echo '请在 tmux 中运行开发服务器' && exit 1"}],
  "description": "强制在 tmux 中运行 dev 服务器"
}

// TypeScript 编辑后自动格式化
{
  "matcher": "tool == \"Edit\" && tool_input.file_path matches \"\\.tsx?$\"",
  "hooks": [{"type": "command", "command": "npx prettier --write \"$file_path\""}],
  "description": "TypeScript 文件编辑后格式化"
}

// git push 前警告
{
  "matcher": "tool == \"Bash\" && tool_input.command matches \"git push\"",
  "hooks": [{"type": "command", "command": "echo '[Hook] push 前请再次检查更改'"}],
  "description": "push 前检查提醒"
}
```

### 钩子检查清单

- [ ] Matcher 具体（不要太宽泛）
- [ ] 包含清晰的错误/信息消息
- [ ] 使用正确的退出代码（`exit 1` 阻止，`exit 0` 允许）
- [ ] 充分测试完成
- [ ] 包含描述

---

## 贡献命令

命令是用户以 `/command-name` 形式调用的操作。

### 文件位置

```
commands/your-command.md
```

### 命令模板

```markdown
---
description: 显示在 /help 中的简短描述
---

# 命令名称

## 目的

此命令执行的操作。

## 用法

```
/your-command [args]
```

## 工作流

1. 第一步
2. 第二步
3. 最后一步

## 输出

用户收到的结果。
```

### 命令示例

| 命令 | 用途 |
|--------|------|
| `commit.md` | 创建 Git 提交 |
| `code-review.md` | 审查代码更改 |
| `tdd.md` | TDD 工作流 |
| `e2e.md` | E2E 测试 |

---

## 跨 Harness 和翻译

### 技能子集（Codex 和 Cursor）

ECC 还为其他 harness 提供技能子集：

- **Codex:** `.agents/skills/` — `agents/openai.yaml` 中列出的技能将在 Codex 中加载。
- **Cursor:** `.cursor/skills/` — 包含单独的 Cursor 技能子集。

如果添加需要在 Codex 或 Cursor 中也提供的**新技能**：

1. 首先在 `skills/your-skill-name/` 下作为常规 ECC 技能添加。
2. 如果需要在 **Codex** 中提供，在 `.agents/skills/` 中反映，必要时在 `agents/openai.yaml` 中添加引用。
3. 如果需要在 **Cursor** 中提供，根据 Cursor 布局在 `.cursor/skills/` 下添加。

请检查现有目录的结构并遵循相同的模式。此子集同步是手动的，因此建议在 PR 描述中注明是否已反映。

### 翻译

翻译文档位于 `docs/` 下。例如：`docs/zh-CN`、`docs/zh-TW`、`docs/ja-JP`。

如果更改已翻译的代理、命令、技能：

- 同时更新相应的翻译文件，或
- 打开问题以便维护者/翻译者可以进行后续工作。

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
## 摘要
添加了什么以及为什么需要。

## 类型
- [ ] 技能
- [ ] 代理
- [ ] 钩子
- [ ] 命令

## 测试
如何测试的。

## 检查清单
- [ ] 遵循格式指南
- [ ] 在 Claude Code 中测试完成
- [ ] 无敏感信息（API 密钥、路径）
- [ ] 包含清晰的描述
```

### 3. 审查流程

1. 维护者将在 48 小时内审查
2. 如有反馈，应用修改
3. 批准后合并到 main

---

## 指南

### 应该做的事情
- 保持贡献集中和模块化
- 包含清晰的描述
- 提交前测试
- 遵循现有模式
- 记录依赖关系

### 不应该做的事情
- 包含敏感数据（API 密钥、令牌、路径）
- 添加过于复杂或特殊的配置
- 提交未经测试的贡献
- 创建与现有功能重复的内容

---

## 文件名规则

- 使用小写加连字符：`python-reviewer.md`
- 描述性命名：使用 `tdd-workflow.md` 而非 `workflow.md`
- 保持 name 与文件名一致

---

## 有问题吗？

- **问题:** [github.com/affaan-m/everything-claude-code/issues](https://github.com/affaan-m/everything-claude-code/issues)
- **X/Twitter:** [@affaanmustafa](https://x.com/affaanmustafa)

---

感谢您的贡献！让我们一起创造出色的资源。
