---
name: everything-claude-code
description: everything-claude-code 的开发规范与模式。采用 conventional commits 的 JavaScript 项目。
---

# Everything Claude Code 规范

> 2026-03-20 生成自 [affaan-m/everything-claude-code](https://github.com/affaan-m/everything-claude-code)

## 概述

本 skill 向 Claude 传授 everything-claude-code 使用的开发模式与规范。

## 技术栈

- **主要语言**：JavaScript
- **架构**：混合模块组织
- **测试位置**：独立存放

## 何时使用本 Skill

在以下场景激活本 skill：
- 对本仓库进行修改时
- 按照已有模式添加新功能时
- 编写符合项目规范的测试时
- 创建符合规范格式的 commit 时

## Commit 规范

基于对 500 个已分析 commit，遵循以下 commit 消息规范。

### Commit 风格：Conventional Commits

### 使用的前缀

- `fix`
- `test`
- `feat`
- `docs`

### 消息指南

- 平均消息长度：~65 个字符
- 保持第一行简洁且具有描述性
- 使用祈使语气（用 "Add feature" 而非 "Added feature"）


*Commit 消息示例*

```text
feat(rules): add C# language support
```

*Commit 消息示例*

```text
chore(deps-dev): bump flatted (#675)
```

*Commit 消息示例*

```text
fix: auto-detect ECC root from plugin cache when CLAUDE_PLUGIN_ROOT is unset (#547) (#691)
```

*Commit 消息示例*

```text
docs: add Antigravity setup and usage guide (#552)
```

*Commit 消息示例*

```text
merge: PR #529 — feat(skills): add documentation-lookup, bun-runtime, nextjs-turbopack; feat(agents): add rust-reviewer
```

*Commit 消息示例*

```text
Revert "Add Kiro IDE support (.kiro/) (#548)"
```

*Commit 消息示例*

```text
Add Kiro IDE support (.kiro/) (#548)
```

*Commit 消息示例*

```text
feat: add block-no-verify hook for Claude Code and Cursor (#649)
```

## 架构

### 项目结构：单包

本项目采用**混合**模块组织。

### 配置文件

- `.github/workflows/ci.yml`
- `.github/workflows/maintenance.yml`
- `.github/workflows/monthly-metrics.yml`
- `.github/workflows/release.yml`
- `.github/workflows/reusable-release.yml`
- `.github/workflows/reusable-test.yml`
- `.github/workflows/reusable-validate.yml`
- `.opencode/package.json`
- `.opencode/tsconfig.json`
- `.prettierrc`
- `eslint.config.js`
- `package.json`

### 指南

- 本项目采用混合组织模式
- 添加新代码时遵循现有模式

## 代码风格

### 语言：JavaScript

### 命名规范

| 元素 | 规范 |
|---------|------------|
| 文件 | camelCase |
| 函数 | camelCase |
| 类 | PascalCase |
| 常量 | SCREAMING_SNAKE_CASE |

### 导入风格：相对导入

### 导出风格：混合风格


*推荐导入风格*

```typescript
// Use relative imports
import { Button } from '../components/Button'
import { useAuth } from './hooks/useAuth'
```

## 测试

### 测试框架

未检测到特定测试框架 —— 使用仓库现有测试模式。

### 文件模式：`*.test.js`

### 测试类型

- **单元测试**：隔离测试单个函数和组件
- **集成测试**：测试多个组件/服务之间的交互

### 覆盖率

本项目已配置覆盖率报告。目标覆盖率达到 80% 以上。


## 错误处理

### 错误处理风格：Try-Catch 块


*标准错误处理模式*

```typescript
try {
  const result = await riskyOperation()
  return result
} catch (error) {
  console.error('Operation failed:', error)
  throw new Error('User-friendly message')
}
```

## 常见 Workflows

这些 workflow 是通过分析 commit 模式检测到的。

### 数据库迁移

使用迁移文件修改数据库 schema

**频率**：每月约 2 次

**步骤**：
1. 创建迁移文件
2. 更新 schema 定义
3. 生成/更新类型

**通常涉及的文件**：
- `**/schema.*`
- `migrations/*`

**示例 commit 序列**：
```
feat: implement --with/--without selective install flags (#679)
fix: sync catalog counts with filesystem (27 agents, 113 skills, 58 commands) (#693)
feat(rules): add Rust language rules (rebased #660) (#686)
```

### 功能开发

标准功能实现 workflow

**频率**：每月约 22 次

**步骤**：
1. 完成功能实现
2. 为功能添加测试
3. 更新文档

**通常涉及的文件**：
- `manifests/*`
- `schemas/*`
- `**/*.test.*`
- `**/api/**`

**示例 commit 序列**：
```
feat(skills): add documentation-lookup, bun-runtime, nextjs-turbopack; feat(agents): add rust-reviewer
docs(skills): align documentation-lookup with CONTRIBUTING template; add cross-harness (Codex/Cursor) skill copies
fix: address PR review — skill template (When to use, How it works, Examples), bun.lock, next build note, rust-reviewer CI note, doc-lookup privacy/uncertainty
```

### 添加语言规则

向规则系统添加新的编程语言规范，包括编码风格、hooks、模式、安全和测试指南。

**频率**：每月约 2 次

**步骤**：
1. 在 rules/{language}/ 下创建新目录
2. 添加 coding-style.md、hooks.md、patterns.md、security.md 和 testing.md 文件，填入对应语言的专属内容
3. 可选择引用或链接到相关 skill

**通常涉及的文件**：
- `rules/*/coding-style.md`
- `rules/*/hooks.md`
- `rules/*/patterns.md`
- `rules/*/security.md`
- `rules/*/testing.md`

**示例 commit 序列**：
```
Create a new directory under rules/{language}/
Add coding-style.md, hooks.md, patterns.md, security.md, and testing.md files with language-specific content
Optionally reference or link to related skills
```

### 添加新 Skill

向系统添加新 skill，记录其 workflow、触发条件和使用方法，通常包含配套脚本。

**频率**：每月约 4 次

**步骤**：
1. 在 skills/{skill-name}/ 下创建新目录
2. 添加包含文档的 SKILL.md（何时使用、如何工作、示例等）
3. 可选择在 skills/{skill-name}/scripts/ 下添加脚本或配套文件
4. 处理评审反馈，迭代优化文档

**通常涉及的文件**：
- `skills/*/SKILL.md`
- `skills/*/scripts/*.sh`
- `skills/*/scripts/*.js`

**示例 commit 序列**：
```
Create a new directory under skills/{skill-name}/
Add SKILL.md with documentation (When to Use, How It Works, Examples, etc.)
Optionally add scripts or supporting files under skills/{skill-name}/scripts/
Address review feedback and iterate on documentation
```

### 添加新 Agent

向系统添加新 agent，用于代码评审、构建问题修复或其他自动化任务。

**频率**：每月约 2 次

**步骤**：
1. 在 agents/{agent-name}.md 下创建新的 agent markdown 文件
2. 在 AGENTS.md 中注册该 agent
3. 可选择更新 README.md 和 docs/COMMAND-AGENT-MAP.md

**通常涉及的文件**：
- `agents/*.md`
- `AGENTS.md`
- `README.md`
- `docs/COMMAND-AGENT-MAP.md`

**示例 commit 序列**：
```
Create a new agent markdown file under agents/{agent-name}.md
Register the agent in AGENTS.md
Optionally update README.md and docs/COMMAND-AGENT-MAP.md
```

### 添加新 Workflow 入口

添加或更新 workflow 入口。默认优先使用 skill 实现；仅当仍需要兼容旧版斜杠命令时才添加命令兼容层。

**频率**：每月约 1 次

**步骤**：
1. 在 skills/{skill-name}/SKILL.md 下创建或更新标准 workflow
2. 仅当需要时，在 commands/{command-name}.md 下添加或更新兼容层

**通常涉及的文件**：
- `skills/*/SKILL.md`
- `commands/*.md`（仅当有意保留旧兼容层时）

**示例 commit 序列**：
```
Create or update the canonical skill under skills/{skill-name}/SKILL.md
Only if needed, add or update commands/{command-name}.md as a compatibility shim
```

### 同步目录计数

将 AGENTS.md 和 README.md 中记录的 agent、skill 和 command 数量与仓库实际状态同步。

**频率**：每月约 3 次

**步骤**：
1. 更新 AGENTS.md 中的 agent、skill 和 command 数量
2. 更新 README.md 中的相同计数（快速入门、对比表格等位置）
3. 可选择更新其他文档文件

**通常涉及的文件**：
- `AGENTS.md`
- `README.md`

**示例 commit 序列**：
```
Update agent, skill, and command counts in AGENTS.md
Update the same counts in README.md (quick-start, comparison table, etc.)
Optionally update other documentation files
```

### 添加跨运行时 Skill 副本

为不同的 agent 运行时（例如 Codex、Cursor、Antigravity）添加 skill 副本，确保跨平台兼容性。

**频率**：每月约 2 次

**步骤**：
1. 复制或适配 SKILL.md 到 .agents/skills/{skill}/SKILL.md 和/或 .cursor/skills/{skill}/SKILL.md
2. 可选择添加运行时专属的 openai.yaml 或配置文件
3. 处理评审反馈，对齐 CONTRIBUTING 模板要求

**通常涉及的文件**：
- `.agents/skills/*/SKILL.md`
- `.cursor/skills/*/SKILL.md`
- `.agents/skills/*/agents/openai.yaml`

**示例 commit 序列**：
```
Copy or adapt SKILL.md to .agents/skills/{skill}/SKILL.md and/or .cursor/skills/{skill}/SKILL.md
Optionally add harness-specific openai.yaml or config files
Address review feedback to align with CONTRIBUTING template
```

### 添加或更新 Hook

添加或更新 git 或 bash hook，用于强制执行 workflow、质量或安全策略。

**频率**：每月约 1 次

**步骤**：
1. 在 hooks/ 或 scripts/hooks/ 下添加或更新 hook 脚本
2. 在 hooks/hooks.json 或类似配置中注册该 hook
3. 可选择在 tests/hooks/ 下添加或更新测试

**通常涉及的文件**：
- `hooks/*.hook`
- `hooks/hooks.json`
- `scripts/hooks/*.js`
- `tests/hooks/*.test.js`
- `.cursor/hooks.json`

**示例 commit 序列**：
```
Add or update hook scripts in hooks/ or scripts/hooks/
Register the hook in hooks/hooks.json or similar config
Optionally add or update tests in tests/hooks/
```

### 处理评审反馈

通过更新文档、脚本或配置来处理代码评审反馈，提升清晰度、正确性或规范对齐度。

**频率**：每月约 4 次

**步骤**：
1. 编辑 SKILL.md、agent 或 command 文件，响应评审意见
2. 按需更新示例、标题或配置
3. 迭代直到所有评审反馈得到解决

**通常涉及的文件**：
- `skills/*/SKILL.md`
- `agents/*.md`
- `commands/*.md`
- `.agents/skills/*/SKILL.md`
- `.cursor/skills/*/SKILL.md`

**示例 commit 序列**：
```
Edit SKILL.md, agent, or command files to address reviewer comments
Update examples, headings, or configuration as requested
Iterate until all review feedback is resolved
```


## 最佳实践

基于代码库分析，遵循以下实践：

### 推荐

- 使用 conventional commit 格式（feat:、fix: 等）
- 遵循 *.test.js 命名模式
- 文件名使用 camelCase
- 优先使用混合导出

### 不推荐

- 不要编写模糊的 commit 消息
- 不要跳过新功能的测试
- 未经讨论不要偏离已有模式

---

*本 skill 由 [ECC Tools](https://ecc.tools) 自动生成。请根据团队需要评审并自定义。*
