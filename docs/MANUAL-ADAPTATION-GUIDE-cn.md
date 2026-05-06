# Manual Adaptation Guide for Non-Native Harnesses

当你希望在不能原生加载 `.claude/`、`.codex/`、`.opencode/`、`.cursor/` 或 `.agent/` 布局的 harness 中使用 ECC 行为时，请使用本指南。

这是针对 Grok 和其他聊天式界面工具的备用路径，这些工具可以接受 system prompts、上传文件或粘贴指令，但不能直接执行仓库的原生安装界面。

## When to Use This

当目标 harness 满足以下条件时使用手动适配：

- 不自动加载仓库文件夹
- 不支持自定义 slash commands
- 不支持 hooks
- 不支持仓库本地 skill 激活
- 只有部分或完全没有文件系统/工具访问权限

只要存在一流的 ECC 目标，请优先选择：

- Claude Code
- Codex
- Cursor
- OpenCode
- CodeBuddy
- Antigravity

仅当你需要在非原生 harness 中使用 ECC 行为时才使用本指南。

## What You Are Reproducing

当你手动适配 ECC 时，你正在尝试保留四件事：

1. 专注的上下文，而不是转储整个仓库。
2. Skill 激活提示，而不是希望模型猜测工作流程。
3. 命令意图，即使 harness 没有 slash-command 系统。
4. Hook 纪律，即使 harness 没有原生自动化。

你不是在尝试镜像仓库中的每个文件。你正在尝试用尽可能小的上下文包来重建有用的行为。

## The ECC-Native Fallback

默认从仓库本身进行手动选择。

只从你真正需要的文件开始：

- 一个语言或框架 skill
- 一个工作流程 skill
- 如果任务是专门化的，一个领域 skill
- 只有当 harness 从显式编排中受益时，才需要一个 agent 或命令

好的最小示例：

- Python 功能开发：
  - `skills/python-patterns/SKILL.md`
  - `skills/tdd-workflow/SKILL.md`
  - `skills/verification-loop/SKILL.md`
- TypeScript API 开发：
  - `skills/backend-patterns/SKILL.md`
  - `skills/security-review/SKILL.md`
  - `skills/tdd-workflow/SKILL.md`
- 内容/对外工作：
  - `skills/brand-voice/SKILL.md`
  - `skills/content-engine/SKILL.md`
  - `skills/crosspost/SKILL.md`

如果 harness 支持文件上传，只上传那些文件。

如果 harness 只支持粘贴上下文，提取相关部分并粘贴压缩包而不是原始完整文件。

## Manual Context Packing

你不需要额外的工具来做这件事。

直接使用仓库：

```bash
cd /path/to/everything-claude-code

sed -n '1,220p' skills/tdd-workflow/SKILL.md > /tmp/ecc-context.md
printf '\n\n---\n\n' >> /tmp/ecc-context.md
sed -n '1,220p' skills/backend-patterns/SKILL.md >> /tmp/ecc-context.md
printf '\n\n---\n\n' >> /tmp/ecc-context.md
sed -n '1,220p' skills/security-review/SKILL.md >> /tmp/ecc-context.md
```

你也可以在打包前使用 `rg` 来识别正确的 skills：

```bash
rg -n "When to use|Use when|Trigger" skills -g 'SKILL.md'
```

可选：如果你已经使用像 `repomix` 这样的仓库打包器，它可以帮助将选定的文件压缩成一个交接文档。它是一个便利工具，不是规范的 ECC 路径。

## Compression Rules

当为另一个 harness 手动打包 ECC 时：

- 保留任务框架
- 保留激活条件
- 保留工作流程步骤
- 保留关键示例
- 首先删除重复的叙述
- 其次删除不相关的变体
- 当一两个 skill 就足够时，避免粘贴整个目录

如果你需要更紧凑的提示格式，将必要部分转换为紧凑的结构化块：

```xml
<skill name="tdd-workflow">
  <when>New feature, bug fix, or refactor that should be test-first.</when>
  <steps>
    <step>Write a failing test.</step>
    <step>Make it pass with the smallest change.</step>
    <step>Refactor and rerun validation.</step>
  </steps>
</skill>
```

## Reproducing Commands

如果 harness 没有 slash-command 系统，在 system prompt 或会话前言中定义一个小型命令注册表。

示例：

```text
Command registry:
- /plan -> use planner-style reasoning, produce a short execution plan, then act
- /tdd -> follow the tdd-workflow skill
- /review -> switch into code-review mode and enumerate findings first
- /verify -> run a verification loop before claiming completion
```

你不是在实现真正的命令。你是在给 harness 显式的调用句柄，映射到 ECC 行为。

## Reproducing Hooks

如果 harness 没有原生 hooks，将 hook 意图移到常设指令中。

示例：

```text
Before writing code:
1. Check whether a relevant skill should be activated.
2. Check for security-sensitive changes.
3. Prefer tests before implementation when feasible.

Before finalizing:
1. Re-read the user request.
2. Verify the main changed paths.
3. State what was actually validated and what was not.
```

这不会重建真正的自动化，但它捕捉了 ECC 的操作纪律。

## Harness Capability Matrix

| Capability | First-Class ECC Targets | Manual-Adaptation Targets |
| --- | --- | --- |
| Folder-based install | Native | No |
| Slash commands | Native | Simulated in prompt |
| Hooks | Native | Simulated in prompt |
| Skill activation | Native | Manual |
| Repo-local tooling | Native | Depends on harness |
| Context packing | Optional | Required |

## Practical Grok-Style Setup

1. 选择最小的有用包。
2. 将选定的 ECC skill 文件打包成一个上传或粘贴块。
3. 添加一个简短的命令注册表。
4. 添加常设的"hook intent"指令。
5. 从一个任务开始，在扩展之前验证 harness 遵循工作流程。

示例入门前言：

```text
You are operating with a manually adapted ECC bundle.

Active skills:
- backend-patterns
- tdd-workflow
- security-review

Command registry:
- /plan
- /tdd
- /verify

Before writing code, follow the active skill instructions.
Before finalizing, verify what changed and report any remaining gaps.
```

## Limitations

手动适配很有用，但与原生目标相比，它仍然是二流的。

你失去了：

- 自动安装和同步
- 原生 hook 执行
- 真正的命令管道
- 运行时可靠的 skill 发现
- 内置的多 agent/worktree 编排

所以规则很简单：

- 使用手动适配将 ECC 行为带入非原生 harnesses
- 当你想要完整系统时，使用原生 ECC 目标

## Related Work

- [Issue #1186](https://github.com/affaan-m/everything-claude-code/issues/1186)
- [Discussion #1077](https://github.com/affaan-m/everything-claude-code/discussions/1077)
- [Antigravity Guide](./ANTIGRAVITY-GUIDE-cn.md)
- [Troubleshooting](./TROUBLESHOOTING.md)
