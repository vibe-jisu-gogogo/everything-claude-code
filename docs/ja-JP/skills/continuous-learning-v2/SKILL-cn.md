---
name: continuous-learning-v2
description: 通过hooks观察会话，创建具有置信度评分的原子instinct，并进化为skills/commands/agents的基于instinct的学习系统。
version: 2.0.0
---

# Continuous Learning v2 - 基于Instinct的架构

将Claude Code会话通过带有置信度评分的小型已学习行为——"instinct"，转化为可复用知识的高级学习系统。

## v2的新功能

| 功能 | v1 | v2 |
|---------|----|----|
| 观察 | Stop hook（会话结束） | PreToolUse/PostToolUse（100%可靠性） |
| 分析 | 主上下文 | 后台代理（Haiku） |
| 粒度 | 完整skill | 原子"instinct" |
| 置信度 | 无 | 0.3-0.9加权 |
| 进化 | 直接转为skill | instinct → 集群 → skill/command/agent |
| 共享 | 无 | instinct的导出/导入 |

## Instinct模型

Instinct是小型的已学习行为：

```yaml
---
id: prefer-functional-style
trigger: "when writing new functions"
confidence: 0.7
domain: "code-style"
source: "session-observation"
---

# 优先函数式风格

## Action
适当时使用函数式模式而非类。

## Evidence
- 观察到5次优先使用函数式模式
- 用户在2025-01-15将基于类的方法修正为函数式
```

**属性：**
- **原子** — 1个触发器，1个动作
- **置信度加权** — 0.3 = 暂定，0.9 = 几乎确定
- **领域标记** — code-style、testing、git、debugging、workflow等
- **基于证据** — 追踪创建它的观察结果

## 工作原理

```
会话活动
      │
      │ hook捕获prompt + 工具使用（100%可靠性）
      ▼
┌─────────────────────────────────────────┐
│         observations.jsonl              │
│   （prompt、工具调用、结果）               │
└─────────────────────────────────────────┘
      │
      │ Observer代理读取（后台、Haiku）
      ▼
┌─────────────────────────────────────────┐
│          模式检测                        │
│   • 用户修正 → instinct                 │
│   • 错误解决 → instinct                 │
│   • 重复工作流 → instinct               │
└─────────────────────────────────────────┘
      │
      │ 创建/更新
      ▼
┌─────────────────────────────────────────┐
│         instincts/personal/             │
│   • prefer-functional.md (0.7)          │
│   • always-test-first.md (0.9)          │
│   • use-zod-validation.md (0.6)         │
└─────────────────────────────────────────┘
      │
      │ /evolve集群
      ▼
┌─────────────────────────────────────────┐
│              evolved/                   │
│   • commands/new-feature.md             │
│   • skills/testing-workflow.md          │
│   • agents/refactor-specialist.md       │
└─────────────────────────────────────────┘
```

## 快速开始

### 1. 启用观察hook

添加到`~/.claude/settings.json`。

**作为插件安装时**（推荐）：

```json
插件的`hooks/hooks.json`会在Claude Code v2.1+中自动加载，因此无需在`~/.claude/settings.json`中添加额外的hook配置。`observe.sh`已在那里注册。

如果之前已将`observe.sh`复制到`~/.claude/settings.json`，请删除重复的`PreToolUse`/`PostToolUse`块。重复注册会导致双重执行和`${CLAUDE_PLUGIN_ROOT}`解析错误。该变量仅在插件管理的`hooks/hooks.json`中展开。
```

**手动安装到`~/.claude/skills`时**：

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

Python CLI会自动创建，也可以手动创建：

```bash
mkdir -p ~/.claude/homunculus/{instincts/{personal,inherited},evolved/{agents,skills,commands}}
touch ~/.claude/homunculus/observations.jsonl
```

### 3. 使用instinct命令

```bash
/instinct-status     # 显示带有置信度评分的已学习instinct
/evolve              # 将相关instinct集群化为skills/commands
/instinct-export     # 导出instinct以进行共享
/instinct-import     # 从其他人导入instinct
```

## 命令

| 命令 | 说明 |
|---------|-------------|
| `/instinct-status` | 显示所有已学习的instinct及其置信度 |
| `/evolve` | 将相关的instinct集群化为skills/commands |
| `/instinct-export` | 导出instinct以进行共享 |
| `/instinct-import <file>` | 从其他人导入instinct |

## 配置

编辑`config.json`：

```json
{
  "version": "2.0",
  "observation": {
    "enabled": true,
    "store_path": "~/.claude/homunculus/observations.jsonl",
    "max_file_size_mb": 10,
    "archive_after_days": 7
  },
  "instincts": {
    "personal_path": "~/.claude/homunculus/instincts/personal/",
    "inherited_path": "~/.claude/homunculus/instincts/inherited/",
    "min_confidence": 0.3,
    "auto_approve_threshold": 0.7,
    "confidence_decay_rate": 0.05
  },
  "observer": {
    "enabled": true,
    "model": "haiku",
    "run_interval_minutes": 5,
    "patterns_to_detect": [
      "user_corrections",
      "error_resolutions",
      "repeated_workflows",
      "tool_preferences"
    ]
  },
  "evolution": {
    "cluster_threshold": 3,
    "evolved_path": "~/.claude/homunculus/evolved/"
  }
}
```

## 文件结构

```
~/.claude/homunculus/
├── identity.json           # 配置文件、技术水平
├── observations.jsonl      # 当前会话观察
├── observations.archive/   # 已处理观察
├── instincts/
│   ├── personal/           # 自动学习的instinct
│   └── inherited/          # 从其他人导入
└── evolved/
    ├── agents/             # 生成的专业代理
    ├── skills/             # 生成的skills
    └── commands/           # 生成的commands
```

## 与Skill Creator的集成

使用[Skill Creator GitHub App](https://skill-creator.app)时，**两者**都会生成：
- 传统的SKILL.md文件（为了向后兼容）
- instinct集合（用于v2学习系统）

来自仓库分析的instinct具有`source: "repo-analysis"`，并包含源仓库URL。

## 置信度评分

置信度随时间进化：

| 评分 | 含义 | 动作 |
|-------|---------|----------|
| 0.3 | 暂定 | 被建议但不强制执行 |
| 0.5 | 中等 | 相关时应用 |
| 0.7 | 强 | 应用被自动批准 |
| 0.9 | 几乎确定 | 核心行为 |

**置信度上升**时：
- 模式被重复观察
- 用户不修正建议的行为
- 来自其他来源的类似instinct一致

**置信度下降**时：
- 用户明确修正行为
- 模式长期未被观察
- 出现矛盾的证据

## 为什么使用hooks而不是skills进行观察？

> "v1依赖skills进行观察。Skills是概率性的，基于Claude的判断，约有50-80%的触发几率。"

Hooks以**100%的概率**确定性触发。这意味着：
- 所有工具调用都被观察
- 模式不会被遗漏
- 学习具有全面性

## 向后兼容性

v2与v1完全兼容：
- 现有的`~/.claude/skills/learned/`skills继续工作
- Stop hook继续执行（但也会馈送到v2）
- 渐进迁移路径：两者并行运行

## 隐私

- 观察**本地**保留在机器上
- 仅**instinct**（模式）可导出
- 实际代码或对话内容不会被共享
- 可以控制导出的内容

## 相关

- [Skill Creator](https://skill-creator.app) - 从仓库历史生成instinct
- Homunculus - v2架构的灵感（原子观察、置信度评分、instinct进化管道）
- [The Longform Guide](https://x.com/affaanmustafa/status/2014040193557471352) - 持续学习部分

---

*基于instinct的学习：通过一次观察，教会Claude你的模式。*
