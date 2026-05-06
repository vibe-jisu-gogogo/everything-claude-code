---
name: continuous-learning
description: 从 Claude Code 会话中自动提取可重用的模式，并保存为学习到的 skills 以供将来使用。
origin: ECC
---

# 持续学习 Skill

在每个 Claude Code 会话结束时自动评估，提取可保存为学习到的 skills 的可重用模式。

## 何时激活

- 当设置从 Claude Code 会话自动提取模式时
- 当为会话评估配置 Stop hook 时
- 当检查或编辑 `~/.claude/skills/learned/` 中的学习到的 skills 时
- 当调整提取阈值或模式类别时
- 当比较 v1（本版）与 v2（基于 instinct）方法时

## 工作原理

本 skill 在每次会话结束时作为 **Stop hook** 运行：

1. **会话评估**: 检查会话是否有足够的消息（默认：10+）
2. **模式检测**: 识别可从会话中提取的模式
3. **Skill 提取**: 将有用的模式保存到 `~/.claude/skills/learned/` 目录

## 配置

通过编辑 `config.json` 进行自定义：

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

| 模式 | 描述 |
|---------|-------------|
| `error_resolution` | 特定错误是如何解决的 |
| `user_corrections` | 从用户修正中提取的模式 |
| `workarounds` | 框架/库怪异行为的解决方案 |
| `debugging_techniques` | 有效的调试方法 |
| `project_specific` | 项目特定的规则 |

## Hook 安装

添加到您的 `~/.claude/settings.json`：

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

## 为什么使用 Stop Hook？

- **轻量级**: 仅在会话结束时运行一次
- **非阻塞**: 不会为每条消息添加延迟
- **完整上下文**: 可以访问完整的会话记录

## 相关链接

- [The Longform Guide](https://x.com/affaanmustafa/status/2014040193557471352) - 持续学习部分
- `/learn` command - 会话中的手动模式提取

---

## 比较笔记（研究：2025年1月）

### vs Homunculus

Homunculus v2 采用更复杂的方法：

| 特性 | 我们的方法 | Homunculus v2 |
|---------|--------------|---------------|
| 观察 | Stop hook（会话结束） | PreToolUse/PostToolUse hooks（100% 可靠） |
| 分析 | 主上下文 | 后台 agent（Haiku） |
| 粒度 | 完整 skills | 原子级 "instincts" |
| 置信度 | 无 | 0.3-0.9 加权 |
| 演进 | 直接到 skill | Instincts → 聚类 → skill/command/agent |
| 分享 | 无 | 导入/导出 instincts |

**来自 Homunculus 的关键见解：**
> "v1 依赖 skills 进行观察。Skills 是概率性的——它们在 50-80% 的时间内触发。v2 使用 hooks 进行观察（100% 可靠），并使用 instincts 作为学习行为的原子单位。"

### 潜在的 v2 改进

1. **基于 Instinct 的学习** - 具有置信度评分的更小、原子行为
2. **后台观察者** - 并行分析的 Haiku agent
3. **置信度衰减** - Instincts 在遇到矛盾时失去置信度
4. **领域标记** - code-style、testing、git、debugging 等
5. **演进路径** - 将相关 instincts 聚类为 skills/commands

参见：完整规范请查看 `docs/continuous-learning-v2-spec.md`。
