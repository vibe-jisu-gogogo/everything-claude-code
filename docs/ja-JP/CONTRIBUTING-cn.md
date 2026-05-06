# Everything Claude Code に貢献する

感谢您的贡献！本仓库是为 Claude Code 用户打造的社区资源。

## 目录

- [探しているもの](#寻找的贡献类型)
- [クイックスタート](#快速开始)
- [スキルの貢献](#贡献技能)
- [エージェントの貢献](#贡献代理)
- [フックの貢献](#贡献钩子)
- [コマンドの貢献](#贡献命令)
- [プルリクエストプロセス](#Pull-Request-流程)

---

## 寻找的贡献类型

### 代理 (Agent)

能够出色处理特定任务的新代理：
- 语言专属评审员（Python、Go、Rust）
- 框架专家（Django、Rails、Laravel、Spring）
- DevOps 专家（Kubernetes、Terraform、CI/CD）
- 领域专家（ML 流水线、数据工程、移动端）

### 技能 (Skill)

工作流定义与领域知识：
- 语言最佳实践
- 框架模式
- 测试策略
- 架构指南

### 钩子 (Hook)

实用的自动化功能：
- Linting/Formatting 钩子
- 安全检查
- 验证钩子
- 通知钩子

### 命令 (Command)

调用实用工作流的斜杠命令：
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

# 3. 添加贡献（参考以下章节）

# 4. 本地测试
cp -r skills/my-skill ~/.claude/skills/  # 对于技能
# 然后在 Claude Code 中进行测试

# 5. 提交 PR
git add . && git commit -m "feat: add my-skill" && git push
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
---

# Your Skill Title

此技能涵盖内容的概述。

## Core Concepts

解释主要模式和指南。

## Code Examples

```typescript
// 包含经过验证的实际示例
function example() {
  // 注释详尽的代码
}
```

## Best Practices

- 可执行的指南
- 应该做和不应该做的事
- 需要避免的常见陷阱

## When to Use

描述此技能适用的场景。
```

### 技能检查清单

- [ ] 聚焦于单一领域/技术
- [ ] 包含实际代码示例
- [ ] 500 行以内
- [ ] 使用清晰的章节标题
- [ ] 已在 Claude Code 中测试

### 示例技能

| 技能 | 目的 |
|-------|---------|
| `coding-standards/` | TypeScript/JavaScript 模式 |
| `frontend-patterns/` | React 与 Next.js 最佳实践 |
| `backend-patterns/` | API 与数据库模式 |
| `security-review/` | 安全检查清单 |

---

## 贡献代理

代理是通过 Task 工具调用的专用助手。

### 文件位置

```
agents/your-agent-name.md
```

### 代理模板

```markdown
---
name: your-agent-name
description: 此代理执行的操作，以及 Claude 应该何时调用它。请具体！
tools: ["Read", "Write", "Edit", "Bash", "Grep", "Glob"]
model: sonnet
---

你是 [角色] 专家。

## Your Role

- 主要职责
- 次要职责
- 你不执行的操作（边界）

## Workflow

### Step 1: Understand
如何处理任务。

### Step 2: Execute
如何执行工作。

### Step 3: Verify
如何验证结果。

## Output Format

返回给用户的内容。

## Examples

### Example: [场景]
Input: [用户提供的内容]
Action: [执行的操作]
Output: [返回的内容]
```

### 代理字段

| 字段 | 描述 | 选项 |
|-------|-------------|---------|
| `name` | 小写、连字符分隔 | `code-reviewer` |
| `description` | 用于判断是否调用 | 请具体！ |
| `tools` | 仅包含必要的 | `Read, Write, Edit, Bash, Grep, Glob, WebFetch, Task` |
| `model` | 复杂度级别 | `haiku`（简单）、`sonnet`（编码）、`opus`（复杂） |

### 示例代理

| 代理 | 目的 |
|-------|---------|
| `tdd-guide.md` | 测试驱动开发 |
| `code-reviewer.md` | 代码评审 |
| `security-reviewer.md` | 安全扫描 |
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
|------|---------|----------|
| `PreToolUse` | 工具执行前 | 验证、警告、阻止 |
| `PostToolUse` | 工具执行后 | 格式化、检查、通知 |
| `SessionStart` | 会话开始 | 加载上下文 |
| `Stop` | 会话结束 | 清理、审计 |

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

### 匹配器语法

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
// 阻止在 tmux 外启动开发服务器
{
  "matcher": "tool == \"Bash\" && tool_input.command matches \"npm run dev\"",
  "hooks": [{"type": "command", "command": "echo 'Use tmux for dev servers' && exit 1"}],
  "description": "确保开发服务器在 tmux 中运行"
}

// 编辑 TypeScript 文件后自动格式化
{
  "matcher": "tool == \"Edit\" && tool_input.file_path matches \"\\.tsx?$\"",
  "hooks": [{"type": "command", "command": "npx prettier --write \"$file_path\""}],
  "description": "编辑后格式化 TypeScript 文件"
}

// git push 前发出警告
{
  "matcher": "tool == \"Bash\" && tool_input.command matches \"git push\"",
  "hooks": [{"type": "command", "command": "echo '[Hook] Review changes before pushing'"}],
  "description": "推送前提醒审查变更"
}
```

### 钩子检查清单

- [ ] 匹配器具体（不过于宽泛）
- [ ] 包含清晰的错误/信息消息
- [ ] 使用正确的退出码（`exit 1` 表示阻止，`exit 0` 表示允许）
- [ ] 经过彻底测试
- [ ] 包含描述

---

## 贡献命令

命令是通过 `/command-name` 调用的用户触发操作。

### 文件位置

```
commands/your-command.md
```

### 命令模板

```markdown
---
description: 显示在 /help 中的简短描述
---

# Command Name

## Purpose

此命令执行的操作。

## Usage

```
/your-command [args]
```

## Workflow

1. 第一步
2. 第二步
3. 最终步骤

## Output

用户将收到的内容。
```

### 示例命令

| 命令 | 目的 |
|---------|---------|
| `commit.md` | 创建 git 提交 |
| `code-review.md` | 评审代码变更 |
| `tdd.md` | TDD 工作流 |
| `e2e.md` | E2E 测试 |

---

## Pull-Request 流程

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
## Summary
添加了什么内容，以及为什么。

## Type
- [ ] Skill
- [ ] Agent
- [ ] Hook
- [ ] Command

## Testing
如何进行测试。

## Checklist
- [ ] 遵循格式指南
- [ ] 已在 Claude Code 中测试
- [ ] 无敏感信息（API 密钥、路径）
- [ ] 描述清晰
```

### 3. 评审流程

1. 维护者在 48 小时内评审
2. 根据要求回应反馈
3. 批准后合并到 main 分支

---

## 指南

### 应该做的事

- 保持贡献聚焦和模块化
- 包含清晰的描述
- 提交前进行测试
- 遵循现有模式
- 文档化依赖关系

### 不应该做的事

- 包含敏感数据（API 密钥、令牌、路径）
- 添加过于复杂或小众的配置
- 提交未经测试的贡献
- 创建与现有功能重复的内容

---

## 文件命名规则

- 使用小写和连字符：`python-reviewer.md`
- 具有描述性：使用 `tdd-workflow.md` 而非 `workflow.md`
- 名称与文件名保持一致

---

## 有问题？

- **Issues:** [github.com/affaan-m/everything-claude-code/issues](https://github.com/affaan-m/everything-claude-code/issues)
- **X/Twitter:** [@affaanmustafa](https://x.com/affaanmustafa)

---

感谢您的贡献。让我们一起打造出色的资源。
