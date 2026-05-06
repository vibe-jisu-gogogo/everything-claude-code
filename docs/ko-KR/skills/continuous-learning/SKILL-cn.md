---
name: continuous-learning
description: 自动从 Claude Code 会话中提取可重用模式并保存为学习技能以供将来使用。
origin: ECC
---

# 持续学习技能

在 Claude Code 会话结束时自动评估并提取可保存为学习技能的可重用模式。

## 激活时机

- 在 Claude Code 会话中设置自动模式提取时
- 配置用于会话评估的 Stop Hook 时
- 在 `~/.claude/skills/learned/` 中审查或整理已学习的技能时
- 调整提取阈值或模式类别时
- 比较 v1（此方式）与 v2（基于本能）方法时

## 工作方式

此技能在每次会话结束时作为 **Stop Hook** 运行：

1. **会话评估**：检查会话是否包含足够消息（默认：10条以上）
2. **模式检测**：识别会话中可提取的模式
3. **技能提取**：将有用模式保存到 `~/.claude/skills/learned/`

## 配置

编辑 `config.json` 进行自定义：

```json
{
  "min_session_length": 10,
  "extraction_threshold": "medium",
  "auto_approve": false,
  "learned_skills_path": "~/.claude/skills/learned/",
  "patterns_to_detect": [
    "error_resolution",
    "user_corrections",
    "workarounds",
    "debugging_techniques",
    "project_specific"
  ],
  "ignore_patterns": [
    "simple_typos",
    "one_time_fixes",
    "external_api_issues"
  ]
}
```

## 模式类型

| 模式 | 说明 |
|---------|-------------|
| `error_resolution` | 特定错误是如何解决的 |
| `user_corrections` | 来自用户修正的模式 |
| `workarounds` | 框架/库特性的解决方案 |
| `debugging_techniques` | 有效的调试方法 |
| `project_specific` | 项目特有的约定 |

## Hook 设置

添加到 `~/.claude/settings.json`：

```json
{
  "hooks": {
    "Stop": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "~/.claude/skills/continuous-learning/evaluate-session.sh"
      }]
    }]
  }
}
```

## 示例

### 自动模式提取配置示例

```json
{
  "min_session_length": 10,
  "extraction_threshold": "medium",
  "auto_approve": false,
  "learned_skills_path": "~/.claude/skills/learned/"
}
```

### Stop Hook 连接示例

```json
{
  "hooks": {
    "Stop": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "~/.claude/skills/continuous-learning/evaluate-session.sh"
      }]
    }]
  }
}
```

## 为什么使用 Stop Hook

- **轻量级**：仅在会话结束时运行一次
- **非阻塞**：不会为每条消息增加延迟
- **完整上下文**：可访问完整的会话记录

## 相关项目

- [The Longform Guide](https://x.com/affaanmustafa/status/2014040193557471352) - 持续学习章节
- `/learn` 命令 - 会话期间手动模式提取

---

## 比较说明（研究：2025年1月）

### vs Homunculus

Homunculus v2 采用更精细的方法：

| 功能 | 我们的方法 | Homunculus v2 |
|---------|--------------|---------------|
| 观察 | Stop Hook（会话结束时） | PreToolUse/PostToolUse Hook（100% 可靠） |
| 分析 | 主上下文 | 后台代理（Haiku） |
| 粒度 | 完整技能 | 原子化"本能" |
| 可信度 | 无 | 0.3-0.9 权重 |
| 演进 | 直接成为技能 | 本能 → 聚类 → 技能/命令/代理 |
| 共享 | 无 | 本能导出/导入 |

**Homunculus 的核心洞察：**
> "v1 依赖技能进行观察。技能是概率性的，约有 50-80% 的概率被执行。v2 使用 Hook（100% 可靠）进行观察，并将本能作为学习行为的原子单位。"

### 潜在的 v2 改进

1. **基于本能的学习** - 带有可信度分数的更小、原子化的行为
2. **后台观察者** - 并行分析的 Haiku 代理
3. **可信度衰减** - 被反驳时降低本能的可信度
4. **域标签** - code-style、testing、git、debugging 等
5. **演进路径** - 将相关本能聚类为技能/命令

详细规范请参阅 [`continuous-learning-v2-spec.md`](../../../continuous-learning-v2-spec.md)。
