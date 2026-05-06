---
name: continuous-learning-v2
description: 通过 hooks 观察会话，创建带有置信度分数的原子本能，并将其进化为技能/命令/agent 的本能驱动学习系统。v2.1 增加了项目范围本能以防止跨项目污染。
origin: ECC
version: 2.1.0
---

# 持续学习 v2.1 - 本能驱动架构

一个高级学习系统，通过原子"本能 (instinct)"——带有置信度分数的小型学习行为——将 Claude Code 会话转化为可重用知识。

**v2.1** 新增**项目范围本能**——React 模式保留在 React 项目中，Python 规则保留在 Python 项目中，通用模式（如"始终验证输入"）可全局共享。

## 何时启用

- 在 Claude Code 会话中配置自动学习时
- 配置通过 hooks 进行的本能驱动行为提取时
- 调整学习行为的置信度阈值时
- 检查、导出或导入本能库时
- 将本能进化为完整的 skill、command 或 agent 时
- 管理项目范围与全局范围本能时
- 将本能从项目范围提升到全局范围时

## v2.1 新功能

| 功能 | v2.0 | v2.1 |
|---------|------|------|
| 存储 | 全局 (~/.claude/homunculus/) | 项目范围 (projects/<hash>/) |
| 范围 | 所有本能随处适用 | 项目范围 + 全局 |
| 检测 | 无 | git remote URL / 仓库路径 |
| 提升 | 不适用 | 在 2+ 项目中验证时，项目 -> 全局 |
| 命令 | 4 个 (status/evolve/export/import) | 6 个 (+promote/projects) |
| 跨项目 | 污染风险 | 默认隔离 |

## v2 新功能（与 v1 相比）

| 功能 | v1 | v2 |
|---------|----|----|
| 观察 | Stop hook（会话结束） | PreToolUse/PostToolUse（100% 可靠） |
| 分析 | 主上下文 | 后台 Agent (Haiku) |
| 粒度 | 完整技能 | 原子"本能" |
| 置信度 | 无 | 0.3-0.9 权重 |
| 进化 | 直接变为技能 | 本能 -> 集群 -> 技能/命令/Agent |
| 共享 | 无 | 本能导出/导入 |

## 本能模型

本能是小型的学习行为：

```yaml
---
id: prefer-functional-style
trigger: "when writing new functions"
confidence: 0.7
domain: "code-style"
source: "session-observation"
scope: project
project_id: "a1b2c3d4e5f6"
project_name: "my-react-app"
---

# Prefer Functional Style

## Action
Use functional patterns over classes when appropriate.

## Evidence
- Observed 5 instances of functional pattern preference
- User corrected class-based approach to functional on 2025-01-15
```

**属性：**
- **原子**——一个触发器，一个动作
- **置信度加权**——0.3 = 暂定，0.9 = 几乎确定
- **领域标签**——code-style, testing, git, debugging, workflow 等
- **基于证据**——跟踪哪些观察结果创建了它
- **范围感知**——`project`（默认）或 `global`

## 工作原理

```
会话活动（git 仓库内）
      |
      | hook 捕获提示 + 工具使用（100% 可靠）
      | + 项目上下文检测（git remote / 仓库路径）
      v
+---------------------------------------------+
|  projects/<project-hash>/observations.jsonl  |
|   (提示, 工具调用, 结果, 项目)                |
+---------------------------------------------+
      |
      | 由观察者 Agent 读取（后台, Haiku）
      v
+---------------------------------------------+
|          模式检测                             |
|   * 用户修正 -> 本能                          |
|   * 错误修复 -> 本能                          |
|   * 重复工作流 -> 本能                         |
|   * 范围判定：项目或全局？                      |
+---------------------------------------------+
      |
      | 创建/更新
      v
+---------------------------------------------+
|  projects/<project-hash>/instincts/personal/ |
|   * prefer-functional.yaml (0.7) [project]   |
|   * use-react-hooks.yaml (0.9) [project]     |
+---------------------------------------------+
|  instincts/personal/  (全局)                  |
|   * always-validate-input.yaml (0.85) [global]|
|   * grep-before-edit.yaml (0.6) [global]     |
+---------------------------------------------+
      |
      | /evolve 聚类 + /promote
      v
+---------------------------------------------+
|  projects/<hash>/evolved/ (项目范围)           |
|  evolved/ (全局)                              |
|   * commands/new-feature.md                  |
|   * skills/testing-workflow.md               |
|   * agents/refactor-specialist.md            |
+---------------------------------------------+
```

## 项目检测

系统自动检测当前项目：

1. **`CLAUDE_PROJECT_DIR` 环境变量**（最高优先级）
2. **`git remote get-url origin`**——哈希化以生成可移植项目 ID（不同机器上的同一仓库获得相同 ID）
3. **`git rev-parse --show-toplevel`**——使用仓库路径回退（特定于机器）
4. **全局回退**——如果未检测到项目，本能会移至全局范围

每个项目收到 12 字符哈希 ID（例如 `a1b2c3d4e5f6`）。`~/.claude/homunculus/projects.json` 中的注册表文件将 ID 映射到人类可读的名称。

## 快速开始

### 1. 启用观察 Hook

添加到 `~/.claude/settings.json`。

**作为插件安装时**（推荐）：

不要在 `~/.claude/settings.json` 中添加额外的 hook 块。Claude Code v2.1+ 会自动加载插件的 `hooks/hooks.json`，且 `observe.sh` 已在其中注册。

如果之前已将 `observe.sh` 复制到 `~/.claude/settings.json`，请移除重复的 `PreToolUse` / `PostToolUse` 块。重复注册会导致双重执行和 `${CLAUDE_PLUGIN_ROOT}` 解析错误。此变量仅在插件拥有的 `hooks/hooks.json` 条目中扩展。

**手动安装到 `~/.claude/skills` 时**，将以下内容添加到 `~/.claude/settings.json`：

```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "~/.claude/skills/continuous-learning-v2/hooks/observe.sh"
      }]
    }],
    "PostToolUse": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "~/.claude/skills/continuous-learning-v2/hooks/observe.sh"
      }]
    }]
  }
}
```

### 2. 初始化目录结构

系统会在首次使用时自动创建目录，但您也可以手动创建：

```bash
# Global directories
mkdir -p ~/.claude/homunculus/{instincts/{personal,inherited},evolved/{agents,skills,commands},projects}

# Project directories are auto-created when the hook first runs in a git repo
```

### 3. 使用本能命令

```bash
/instinct-status     # 显示学习到的本能（项目 + 全局）
/evolve              # 将相关本能聚类为技能/命令
/instinct-export     # 将本能导出为文件
/instinct-import     # 导入其他人的本能
/promote             # 将项目本能提升到全局范围
/projects            # 列出所有已知项目及本能计数
```

## 命令

| 命令 | 描述 |
|---------|-------------|
| `/instinct-status` | 显示所有本能（项目范围 + 全局）及置信度 |
| `/evolve` | 将相关本能聚类为技能/命令，建议提升 |
| `/instinct-export` | 导出本能（可按范围/领域过滤） |
| `/instinct-import <file>` | 带范围控制导入本能 |
| `/promote [id]` | 将项目本能提升到全局范围 |
| `/projects` | 列出所有已知项目及本能计数 |

## 配置

要控制后台观察者，编辑 `config.json`：

```json
{
  "version": "2.1",
  "observer": {
    "enabled": false,
    "run_interval_minutes": 5,
    "min_observations_to_analyze": 20
  }
}
```

| 键 | 默认值 | 描述 |
|-----|---------|-------------|
| `observer.enabled` | `false` | 启用后台观察者 Agent |
| `observer.run_interval_minutes` | `5` | 观察者分析观察结果的频率 |
| `observer.min_observations_to_analyze` | `20` | 运行分析前的最小观察次数 |

其他行为（观察捕获、本能阈值、项目范围、提升标准）在 `instinct-cli.py` 和 `observe.sh` 的代码默认值中配置。

## 文件结构

```
~/.claude/homunculus/
+-- identity.json           # 个人资料，技能水平
+-- projects.json           # 注册表：项目哈希 -> 名称/路径/远程
+-- observations.jsonl      # 全局观察（回退）
+-- instincts/
|   +-- personal/           # 全局自动学习的本能
|   +-- inherited/          # 全局导入的本能
+-- evolved/
|   +-- agents/             # 全局生成的 Agent
|   +-- skills/             # 全局生成的技能
|   +-- commands/           # 全局生成的命令
+-- projects/
    +-- a1b2c3d4e5f6/       # 项目哈希（来自 git remote URL）
    |   +-- observations.jsonl
    |   +-- observations.archive/
    |   +-- instincts/
    |   |   +-- personal/   # 项目特定自动学习
    |   |   +-- inherited/  # 项目特定导入
    |   +-- evolved/
    |       +-- skills/
    |       +-- commands/
    |       +-- agents/
    +-- f6e5d4c3b2a1/       # 另一个项目
        +-- ...
```

## 范围判定指南

| 模式类型 | 范围 | 示例 |
|-------------|-------|---------|
| 语言/框架规则 | **project** | "使用 React hooks"，"遵循 Django REST 模式" |
| 文件结构偏好 | **project** | "`__tests__`/ 中的测试"，"src/components/ 中的组件" |
| 代码风格 | **project** | "使用函数式风格"，"偏好 dataclasses" |
| 错误处理策略 | **project** | "对错误使用 Result 类型" |
| 安全实践 | **global** | "验证用户输入"，"SQL 清理" |
| 通用最佳实践 | **global** | "先写测试"，"始终处理错误" |
| 工具工作流偏好 | **global** | "编辑前 Grep"，"写入前 Read" |
| Git 实践 | **global** | "Conventional commits"，"小型、专注的提交" |

## 本能提升（项目 -> 全局）

当相同的本能以高置信度出现在多个项目中时，它成为提升到全局范围的候选者。

**自动提升标准：**
- 2+ 项目中相同的本能 ID
- 平均置信度 >= 0.8

**如何提升：**

```bash
# Promote a specific instinct
python3 instinct-cli.py promote prefer-explicit-errors

# Auto-promote all qualifying instincts
python3 instinct-cli.py promote

# Preview without changes
python3 instinct-cli.py promote --dry-run
```

`/evolve` 命令也会建议提升候选者。

## 置信度分数

置信度随时间演变：

| 分数 | 含义 | 行为 |
|-------|---------|----------|
| 0.3 | 暂定 | 建议但不强制 |
| 0.5 | 中等 | 相关时应用 |
| 0.7 | 强 | 应用自动批准 |
| 0.9 | 几乎确定 | 核心行为 |

**置信度增加时：**
- 模式被反复观察
- 用户未修正建议的行为
- 来自其他来源的类似本能一致

**置信度降低时：**
- 用户明确修正了行为
- 模式长时间未被观察
- 出现矛盾证据

## 为什么使用 Hooks 而非技能进行观察？

> "v1 依赖技能进行观察。技能是概率性的——Claude 判断，约 50-80% 的几率会被调用。"

Hooks **100%** 确定性地执行。这意味着：
- 所有工具调用都被观察
- 模式不会被遗漏
- 学习是全面的

## 向后兼容性

v2.1 与 v2.0 和 v1 完全兼容：
- `~/.claude/homunculus/instincts/` 中的现有全局本能继续作为全局本能工作
- v1 在 `~/.claude/skills/learned/` 中的现有技能继续工作
- Stop hooks 仍在运行（但现在也为 v2 提供数据）
- 渐进式迁移：两者可并行运行

## 隐私

- 观察结果**本地**保存在用户机器上
- 项目范围本能按项目隔离
- 只有**本能**（模式）可导出——不是原始观察结果
- 无实际代码或对话内容被共享
- 用户控制导出和提升的内容

## 相关资源

- [Skill Creator](https://skill-creator.app) - 从仓库历史生成本能
- Homunculus - 启发 v2 本能架构的社区项目（原子观察、置信度分数、本能进化管道）
- [The Longform Guide](https://x.com/affaanmustafa/status/2014040193557471352) - 持续学习部分

---

*本能驱动学习：一次一个项目地教 Claude 你的模式。*
