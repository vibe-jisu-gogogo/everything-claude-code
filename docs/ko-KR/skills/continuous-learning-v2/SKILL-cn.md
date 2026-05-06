---
name: continuous-learning-v2
description: 通过 hooks 观察会话，创建带有置信度分数的原子本能，并将其进化为 skills/commands/agents 的本能驱动学习系统。v2.1 新增了项目范围本能以防止跨项目污染。
origin: ECC
version: 2.1.0
---

# 持续学习 v2.1 - 本能驱动架构

一个高级学习系统，通过原子化的"本能(instinct)"——带有置信度分数的小型学习行为——将 Claude Code 会话转化为可重用的知识。

**v2.1** 新增了 **项目范围本能**——React 模式保留在 React 项目中，Python 规则保留在 Python 项目中，通用模式（如"始终验证输入"）在全局共享。

## 何时使用

- 在 Claude Code 会话中配置自动学习时
- 配置基于本能的 hooks 行为提取时
- 调整学习行为的置信度阈值时
- 审查、导出、导入本能库时
- 将本能进化为完整的 skill、command 或 agent 时
- 管理项目范围与全局范围本能时
- 将本能从项目提升到全局范围时

## v2.1 新功能

| 功能 | v2.0 | v2.1 |
|---------|------|------|
| 存储 | 全局 (~/.claude/homunculus/) | 项目范围 (projects/<hash>/) |
| 范围 | 所有本能在任何地方应用 | 项目范围 + 全局 |
| 检测 | 无 | git remote URL / 仓库路径 |
| 提升 | 不适用 | 在 2+ 个项目中验证后从项目 -> 全局 |
| 命令 | 4 个 (status/evolve/export/import) | 6 个 (+promote/projects) |
| 跨项目 | 污染风险 | 默认隔离 |

## v2 新功能（对比 v1）

| 功能 | v1 | v2 |
|---------|----|----|
| 观察 | Stop hook（会话结束） | PreToolUse/PostToolUse（100% 可靠） |
| 分析 | 主上下文 | 后台代理 (Haiku) |
| 粒度 | 完整技能 | 原子化"本能" |
| 置信度 | 无 | 0.3-0.9 权重 |
| 进化 | 直接到技能 | 本能 -> 集群 -> 技能/命令/代理 |
| 共享 | 无 | 本能导出/导入 |

## 本能模型

本能是小型学习行为：

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
- **原子化**——一个触发，一个动作
- **置信度权重**——0.3 = 暂定，0.9 = 几乎确定
- **领域标签**——code-style, testing, git, debugging, workflow 等
- **基于证据**——跟踪哪些观察创建了它
- **范围感知**——`project`（默认）或 `global`

## 工作原理

```
会话活动（在 git 仓库内）
      |
      | hooks 捕获 prompt + 工具使用（100% 可靠）
      | + 项目上下文检测（git remote / 仓库路径）
      v
+---------------------------------------------+
|  projects/<project-hash>/observations.jsonl  |
|   (prompt, 工具调用, 结果, 项目)              |
+---------------------------------------------+
      |
      | 观察者代理读取（后台, Haiku）
      v
+---------------------------------------------+
|          模式检测                             |
|   * 用户纠正 -> 本能                          |
|   * 错误修复 -> 本能                          |
|   * 重复工作流 -> 本能                         |
|   * 范围确定: 项目还是全局?                    |
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
|  projects/<hash>/evolved/ (项目范围)          |
|  evolved/ (全局)                              |
|   * commands/new-feature.md                  |
|   * skills/testing-workflow.md               |
|   * agents/refactor-specialist.md            |
+---------------------------------------------+
```

## 项目检测

系统自动检测当前项目：

1. **`CLAUDE_PROJECT_DIR` 环境变量**（最高优先级）
2. **`git remote get-url origin`**——哈希生成可移植项目 ID（不同机器上的同一仓库获得相同 ID）
3. **`git rev-parse --show-toplevel`**——使用仓库路径的回退方案（机器特定）
4. **全局回退**——如果未检测到项目，本能移至全局范围

每个项目接收 12 字符哈希 ID（例如：`a1b2c3d4e5f6`）。`~/.claude/homunculus/projects.json` 中的注册表文件将 ID 映射到人类可读的名称。

## 快速开始

### 1. 启用观察 Hooks

添加到 `~/.claude/settings.json`。

**如果作为插件安装**（推荐）：

不要向 `~/.claude/settings.json` 添加 hook 块。Claude Code v2.1+ 会自动加载插件的 `hooks/hooks.json`，`observe.sh` 已在那里注册。

如果您之前已将 `observe.sh` 复制到 `~/.claude/settings.json`，请删除重复的 `PreToolUse` / `PostToolUse` 块。重复注册会导致双重执行和 `${CLAUDE_PLUGIN_ROOT}` 解析错误。此变量仅在插件拥有的 `hooks/hooks.json` 条目中展开。

**如果手动安装到 `~/.claude/skills`**，请将以下内容添加到 `~/.claude/settings.json`：

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
/instinct-export     # 将本能导出到文件
/instinct-import     # 导入他人的本能
/promote             # 将项目本能提升到全局范围
/projects            # 列出所有已知项目及本能计数
```

## 命令

| 命令 | 描述 |
|---------|-------------|
| `/instinct-status` | 显示所有本能（项目范围 + 全局）及置信度 |
| `/evolve` | 将相关本能聚类为技能/命令，提出提升建议 |
| `/instinct-export` | 导出本能（可按范围/领域过滤） |
| `/instinct-import <file>` | 带范围控制导入本能 |
| `/promote [id]` | 将项目本能提升到全局范围 |
| `/projects` | 列出所有已知项目及本能计数 |

## 配置

编辑 `config.json` 以控制后台观察者：

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
| `observer.enabled` | `false` | 启用后台观察者代理 |
| `observer.run_interval_minutes` | `5` | 观察者分析观察结果的频率 |
| `observer.min_observations_to_analyze` | `20` | 运行分析前的最小观察次数 |

其他行为（观察捕获、本能阈值、项目范围、提升标准）在 `instinct-cli.py` 和 `observe.sh` 的代码默认值中配置。

## 文件结构

```
~/.claude/homunculus/
+-- identity.json           # 个人资料，技能水平
+-- projects.json           # 注册表: 项目哈希 -> 名称/路径/远程
+-- observations.jsonl      # 全局观察结果（回退）
+-- instincts/
|   +-- personal/           # 全局自动学习的本能
|   +-- inherited/          # 全局导入的本能
+-- evolved/
|   +-- agents/             # 全局生成的代理
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
    +-- f6e5d4c3b2a1/       # 其他项目
        +-- ...
```

## 范围确定指南

| 模式类型 | 范围 | 示例 |
|-------------|-------|---------|
| 语言/框架规则 | **project** | "使用 React hooks", "遵循 Django REST 模式" |
| 文件结构偏好 | **project** | "测试放在 `__tests__`/", "组件放在 src/components/" |
| 代码风格 | **project** | "使用函数式风格", "偏好 dataclasses" |
| 错误处理策略 | **project** | "错误使用 Result 类型" |
| 安全实践 | **global** | "验证用户输入", "SQL 安全处理" |
| 通用最佳实践 | **global** | "先写测试", "始终处理错误" |
| 工具工作流偏好 | **global** | "编辑前 Grep", "写入前 Read" |
| Git 实践 | **global** | "Conventional commits", "小而专注的提交" |

## 本能提升（项目 -> 全局）

当同一个本能以高置信度出现在多个项目中时，它成为提升到全局范围的候选。

**自动提升标准：**
- 在 2+ 个项目中存在相同的本能 ID
- 平均置信度 >= 0.8

**提升方法：**

```bash
# Promote a specific instinct
python3 instinct-cli.py promote prefer-explicit-errors

# Auto-promote all qualifying instincts
python3 instinct-cli.py promote

# Preview without changes
python3 instinct-cli.py promote --dry-run
```

`/evolve` 命令也会提出提升候选建议。

## 置信度分数

置信度随时间演变：

| 分数 | 含义 | 行为 |
|-------|---------|----------|
| 0.3 | 暂定 | 建议但不强制执行 |
| 0.5 | 中等 | 相关时应用 |
| 0.7 | 强烈 | 应用自动批准 |
| 0.9 | 几乎确定 | 核心行为 |

**置信度增加时：**
- 模式被重复观察到
- 用户未纠正建议的行为
- 其他来源的类似本能一致

**置信度减少时：**
- 用户明确纠正行为
- 模式长时间未被观察到
- 出现矛盾证据

## 为什么使用 Hooks 而不是 Skills 进行观察？

> "v1 依赖 skills 进行观察。Skills 是概率性的——根据 Claude 的判断，大约有 50-80% 的概率被执行。"

Hooks 以 **100% 的概率** 确定性执行。这意味着：
- 所有工具调用都被观察到
- 不会遗漏模式
- 学习是全面的

## 向后兼容性

v2.1 与 v2.0 和 v1 完全兼容：
- `~/.claude/homunculus/instincts/` 中的现有全局本能继续作为全局本能工作
- v1 的现有 `~/.claude/skills/learned/` 技能继续工作
- Stop hooks 仍在执行（但现在也为 v2 提供数据）
- 渐进式迁移：两者可以并行运行

## 隐私

- 观察结果 **本地** 保留在用户机器上
- 项目范围本能按项目隔离
- 只有 **本能**（模式）可以导出——而非原始观察结果
- 不共享实际代码或对话内容
- 用户控制导出和提升的内容

## 相关资源

- [Skill Creator](https://skill-creator.app) - 从仓库历史生成本能
- Homunculus - 启发 v2 本能驱动架构的社区项目（原子观察、置信度分数、本能进化管道）
- [The Longform Guide](https://x.com/affaanmustafa/status/2014040193557471352) - 持续学习部分

---

*本能驱动学习：一次一个项目，教 Claude 你的模式。*
