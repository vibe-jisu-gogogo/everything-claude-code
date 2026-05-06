---
name: strategic-compact
description: 建议在任务阶段通过逻辑间隔进行手动压缩，而不是任意的自动压缩，以保持上下文。
---

# Strategic Compact 技能

建议在工作流程的战略点进行手动 `/compact`，而不是依赖任意的自动压缩。

## 为什么选择战略压缩？

自动压缩在任意点触发：
- 通常在任务中途，丢失重要上下文
- 不识别任务的逻辑边界
- 可能中断复杂的多步骤操作

在逻辑边界进行战略压缩：
- **探索后，执行前** - 压缩研究上下文，保留实施计划
- **里程碑完成后** - 为下一阶段提供新的开始
- **主要上下文转换前** - 在不同任务前清除探索上下文

## 工作原理

`suggest-compact.sh` 脚本在 PreToolUse（Edit/Write）时执行：

1. **跟踪工具调用** - 计数会话内的工具调用
2. **阈值检测** - 在可配置的阈值时建议（默认：50次）
3. **定期提醒** - 阈值后每25次提醒一次

## 钩子配置

添加到 `~/.claude/settings.json`：

```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "tool == \"Edit\" || tool == \"Write\"",
      "hooks": [{
        "type": "command",
        "command": "~/.claude/skills/strategic-compact/suggest-compact.sh"
      }]
    }]
  }
}
```

## 配置

环境变量：
- `COMPACT_THRESHOLD` - 首次建议前的工具调用次数（默认：50）

## 最佳实践

1. **计划后压缩** - 计划确定后，压缩并重新开始
2. **调试后压缩** - 继续前清除错误解决上下文
3. **实施期间不压缩** - 为相关更改保留上下文
4. **阅读建议** - 钩子会告诉你*何时*压缩，但*是否*压缩由你决定

## 相关

- [The Longform Guide](https://x.com/affaanmustafa/status/2014040193557471352) - 令牌优化部分
- 内存持久化钩子 - 用于在压缩后仍然存在的状态
