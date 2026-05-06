---
name: continuous-learning
description: 从 Claude Code 会话中自动提取可复用模式，并作为已学习的技能保存以供将来使用。
---

# 持续学习技能

在 Claude Code 会话结束时自动评估，提取可复用模式并保存为已学习的技能。

## 工作原理

此技能在每次会话结束时作为 **Stop 钩子** 执行：

1. **会话评估**：确认会话中有足够的消息（默认：10 条以上）
2. **模式检测**：识别会话中可提取的模式
3. **技能提取**：将有用的模式保存到 `~/.claude/skills/learned/`

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

| 模式 | 描述 |
|---------|-------------|
| `error_resolution` | 特定错误的解决方法 |
| `user_corrections` | 来自用户修正的模式 |
| `workarounds` | 框架/库特性的解决方案 |
| `debugging_techniques` | 有效的调试方法 |
| `project_specific` | 项目特有的约定 |

## 钩子配置

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

## 使用 Stop 钩子的理由

- **轻量**：仅在会话结束时执行一次
- **非阻塞**：不会为所有消息增加延迟
- **完整上下文**：可以访问整个会话的 transcript

## 相关资源

- [The Longform Guide](https://x.com/affaanmustafa/status/2014040193557471352) - 关于持续学习的章节
- `/learn` 命令 - 会话中手动提取模式

---

## 比较笔记（调研：2025年1月）

### vs Homunculus

Homunculus v2 采用更精细的方法：

| 功能 | 此方法 | Homunculus v2 |
|---------|--------------|---------------|
| 观察 | Stop 钩子（会话结束时） | PreToolUse/PostToolUse 钩子（100% 可靠性） |
| 分析 | 主上下文 | 后台代理（Haiku） |
| 粒度 | 完整技能 | 原子化的「本能」 |
| 置信度 | 无 | 0.3-0.9 权重 |
| 进化 | 直接生成技能 | 本能 → 聚类 → 技能/命令/代理 |
| 共享 | 无 | 本能的导出/导入 |

**来自 homunculus 的重要洞察：**
> "v1 依赖技能进行观察。技能是概率性的，触发率约 50-80%。v2 使用钩子（100% 可靠性）进行观察，并使用本能作为学习行为的原子单位。"

### v2 的潜在改进

1. **基于本能的学习** - 更小、原子化的行为，附带置信度评分
2. **后台观察者** - 并行分析的 Haiku 代理
3. **置信度衰减** - 本能在矛盾时置信度降低
4. **领域标记** - 代码风格、测试、git、调试等
5. **进化路径** - 将相关本能聚类为技能/命令

详情：参考 `docs/continuous-learning-v2-spec.md`。
