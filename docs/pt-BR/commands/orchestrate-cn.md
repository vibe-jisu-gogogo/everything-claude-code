---
description: 面向多代理流程的顺序编排和 tmux/worktree 指导。
---

# Orchestrate 命令

复杂任务的代理顺序流程。

## 用法

`/orchestrate [workflow-type] [task-description]`

## Workflow 类型

### feature
完整的功能实现工作流：
```
planner -> tdd-guide -> code-reviewer -> security-reviewer
```

### bugfix
Bug 调查与修复工作流：
```
planner -> tdd-guide -> code-reviewer
```

### refactor
安全重构工作流：
```
architect -> code-reviewer -> tdd-guide
```

### security
专注于安全的审查：
```
security-reviewer -> code-reviewer -> architect
```

## 执行模式

对于工作流中的每个代理：

1. **调用代理**，传入前一个代理的上下文
2. **收集输出**作为结构化的交接文档
3. **传递给链中的下一个代理**
4. **聚合结果**到最终报告中

## 交接文档格式

在代理之间，创建交接文档：

```markdown
## HANDOFF: [previous-agent] -> [next-agent]

### Context
[已完成工作的摘要]

### Findings
[关键发现或决策]

### Files Modified
[涉及的文件列表]

### Open Questions
[留给下一个代理的未解决项]

### Recommendations
[建议的下一步]
```

## 示例：功能工作流

```
/orchestrate feature "Add user authentication"
```

执行：

1. **Planner Agent**
   - 分析需求
   - 创建实施计划
   - 识别依赖
   - 输出：`HANDOFF: planner -> tdd-guide`

2. **TDD Guide Agent**
   - 读取 planner 的交接
   - 先写测试
   - 实现代码以通过测试
   - 输出：`HANDOFF: tdd-guide -> code-reviewer`

3. **Code Reviewer Agent**
   - 审查实现
   - 检查问题
   - 建议改进
   - 输出：`HANDOFF: code-reviewer -> security-reviewer`

4. **Security Reviewer Agent**
   - 安全审计
   - 漏洞检查
   - 最终批准
   - 输出：最终报告

## 最终报告格式

```
ORCHESTRATION REPORT
====================
Workflow: feature
Task: Add user authentication
Agents: planner -> tdd-guide -> code-reviewer -> security-reviewer

SUMMARY
-------
[一段摘要]

AGENT OUTPUTS
-------------
Planner: [summary]
TDD Guide: [summary]
Code Reviewer: [summary]
Security Reviewer: [summary]

FILES CHANGED
-------------
[列出所有修改的文件]

TEST RESULTS
------------
[测试通过/失败摘要]

SECURITY STATUS
---------------
[安全发现]

RECOMMENDATION
--------------
[SHIP / NEEDS WORK / BLOCKED]
```

## 并行执行

对于独立检查，并行运行代理：

```markdown
### 并行阶段
同时执行：
- code-reviewer (质量)
- security-reviewer (安全)
- architect (设计)

### 合并结果
将输出合并到单个报告中

对于在独立 git worktrees 中的 tmux 窗格中运行的外部 worker，使用 `node scripts/orchestrate-worktrees.js plan.json --execute`。内置的编排模式保持在当前进程中；该辅助工具用于长会话或跨 harness 的场景。

当 worker 需要查看主检出中本地的脏文件或未跟踪文件时，在计划文件中添加 `seedPaths`。ECC 在 `git worktree add` 后，仅将这些选定路径覆盖到每个 worker 的 worktree 中，保持分支隔离，同时仍暴露正在进行的脚本、计划或文档。

```json
{
  "sessionName": "workflow-e2e",
  "seedPaths": [
    "scripts/orchestrate-worktrees.js",
    "scripts/lib/tmux-worktree-orchestrator.js",
    ".claude/plan/workflow-e2e-test.json"
  ],
  "workers": [
    { "name": "docs", "task": "Update orchestration docs." }
  ]
}
```

要导出控制平面的快照到活动的 tmux/worktree 会话中，运行：

```bash
node scripts/orchestration-status.js .claude/plan/workflow-visual-proof.json
```

快照包括会话活动、tmux 窗格元数据、worker 状态、目标、种子覆盖和最近交接摘要，格式为 JSON。

## 操作员命令中心交接

当工作流跨越多个会话、worktrees 或 tmux 窗格时，在最终交接中添加一个控制平面块：

```markdown
CONTROL PLANE
-------------
Sessions:
- 活动会话 ID 或别名
- 每个活动 worker 的分支 + worktree 路径
- tmux 窗格或分离会话名称（如适用）

Diffs:
- git status 摘要
- 涉及文件的 git diff --stat
- 合并/冲突风险说明

Approvals:
- 待处理的用户批准
- 等待确认的阻塞步骤

Telemetry:
- 最后活动时间戳或空闲信号
- 预估 token 或成本偏差
- 钩子或审查器触发的策略事件
```

这使得 planner、实现者、审查器和循环 worker 在操作层面上都可读。

## 参数

$ARGUMENTS:
- `feature <description>` - 完整功能工作流
- `bugfix <description>` - Bug 修复工作流
- `refactor <description>` - 重构工作流
- `security <description>` - 安全审查工作流
- `custom <agents> <description>` - 自定义代理序列

## 自定义工作流示例

```
/orchestrate custom "architect,tdd-guide,code-reviewer" "Redesign caching layer"
```

## 提示

1. **从 planner 开始**处理复杂功能
2. **始终包含 code-reviewer**在合并前
3. **使用 security-reviewer**处理 auth/支付/PII
4. **保持交接简洁** - 专注于下一个代理需要知道的内容
5. **必要时在代理之间运行验证**
