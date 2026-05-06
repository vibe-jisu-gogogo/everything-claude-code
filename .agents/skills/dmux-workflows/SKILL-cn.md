---
name: dmux-workflows
description: 使用 dmux（面向 AI agents 的 tmux 窗格管理器）实现多 agent 编排。包含跨 Claude Code、Codex、OpenCode 和其他 harness 的并行 agent workflow 模式。适用于并行运行多个 agent 会话或协调多 agent 开发 workflow 的场景。
---

# dmux Workflows

使用 dmux（面向 agent harness 的 tmux 窗格管理器）编排并行 AI agent 会话。

## 激活场景

- 并行运行多个 agent 会话
- 协调跨 Claude Code、Codex 和其他 harness 的工作
- 适合采用分治并行策略的复杂任务
- 用户提及"并行运行"、"拆分工作"、"使用 dmux"或"多 agent"

## 什么是 dmux

dmux 是一款基于 tmux 的编排工具，用于管理 AI agent 窗格：
- 按 `n` 可创建带 prompt 的新窗格
- 按 `m` 可将窗格输出合并回主会话
- 支持：Claude Code、Codex、OpenCode、Cline、Gemini、Qwen

**安装：** `npm install -g dmux` 或参考 [github.com/standardagents/dmux](https://github.com/standardagents/dmux)

## 快速开始

```bash
# 启动 dmux 会话
dmux

# 创建 agent 窗格（在 dmux 中按 'n'，然后输入 prompt）
# 窗格 1: "在 src/auth/ 中实现认证中间件"
# 窗格 2: "为用户服务编写测试用例"
# 窗格 3: "更新 API 文档"

# 每个窗格运行独立的 agent 会话
# 按 'm' 合并返回结果
```

## Workflow 模式

### 模式 1：调研 + 实现

将调研和实现拆分为并行路径：

```
窗格 1（调研）："调研 Node.js 限流的最佳实践。
  检查现有库、对比不同实现方案，并将结果写入
  /tmp/rate-limit-research.md"

窗格 2（实现）："为我们的 Express API 实现限流中间件。
  先从基础的令牌桶实现开始，我们会在调研完成后进行优化。"

# 窗格 1 完成后，将调研结果合并到窗格 2 的上下文中
```

### 模式 2：多文件功能开发

跨独立文件并行开展工作：

```
窗格 1："为计费功能创建数据库 schema 和迁移文件"
窗格 2："在 src/api/billing/ 中构建计费 API 端点"
窗格 3："创建计费仪表盘 UI 组件"

# 合并所有结果，然后在主窗格中完成集成
```

### 模式 3：测试 + 修复循环

在一个窗格中运行测试，另一个窗格中修复问题：

```
窗格 1（监视器）："以监听模式运行测试套件。测试失败时，
  总结失败原因。"

窗格 2（修复者）："根据窗格 1 的错误输出修复失败的测试用例"
```

### 模式 4：跨 Harness 协作

为不同任务使用不同的 AI 工具：

```
窗格 1（Claude Code）："评审认证模块的安全性"
窗格 2（Codex）："重构工具函数以提升性能"
窗格 3（Claude Code）："为结账流程编写 E2E 测试"
```

### 模式 5：代码评审流水线

并行开展多视角评审：

```
窗格 1："评审 src/api/ 的安全漏洞"
窗格 2："评审 src/api/ 的性能问题"
窗格 3："评审 src/api/ 的测试覆盖缺口"

# 将所有评审结果合并为一份报告
```

## 最佳实践

1. **仅并行独立任务。** 不要并行化依赖彼此输出的任务。
2. **边界清晰。** 每个窗格应处理不同的文件或业务领域。
3. **策略性合并。** 合并前先评审窗格输出，避免冲突。
4. **使用 git worktree。** 对于容易产生文件冲突的工作，每个窗格使用独立的 worktree。
5. **资源感知。** 每个窗格都会消耗 API token — 总窗格数保持在 5-6 个以内。

## Git Worktree 集成

适用于涉及重叠文件的任务：

```bash
# 创建 worktree 实现隔离
git worktree add ../feature-auth feat/auth
git worktree add ../feature-billing feat/billing

# 在独立 worktree 中运行 agents
# 窗格 1: cd ../feature-auth && claude
# 窗格 2: cd ../feature-billing && claude

# 完成后合并分支
git merge feat/auth
git merge feat/billing
```

## 配套工具

| 工具 | 功能 | 使用场景 |
|------|------|----------|
| **dmux** | 面向 agents 的 tmux 窗格管理 | 并行 agent 会话 |
| **Superset** | 支持 10+ 并行 agents 的终端 IDE | 大规模编排 |
| **Claude Code Task tool** | 进程内子 agent 生成 | 单个会话内的可编程并行处理 |
| **Codex multi-agent** | 内置 agent 角色 | Codex 专属的并行工作 |

## 问题排查

- **窗格无响应：** 检查 agent 会话是否在等待输入。使用 `m` 读取输出。
- **合并冲突：** 使用 git worktree 隔离每个窗格的文件变更。
- **token 用量过高：** 减少并行窗格数量。每个窗格都是一个完整的 agent 会话。
- **未找到 tmux：** 执行 `brew install tmux`（macOS）或 `apt install tmux`（Linux）安装。
