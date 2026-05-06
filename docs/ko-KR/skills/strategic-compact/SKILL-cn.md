---
name: strategic-compact
description: 在逻辑间隔中建议手动上下文压缩，而不是任意自动压缩，通过工作步骤保留上下文。
origin: ECC
---

# 战略压缩技能

不依赖任意自动压缩，在工作流的战略点建议手动 `/compact`。

## 激活时机

- 执行接近上下文限制的长会话时（200K+ tokens）
- 执行多阶段任务时（调查 -> 计划 -> 实现 -> 测试）
- 在同一会话中切换不相关任务时
- 完成主要里程碑并开始新任务时
- 响应变慢或一致性下降时（上下文压力）

## 为何需要战略压缩

自动压缩在任意点执行：
- 经常在任务中间执行，丢失重要上下文
- 不识别逻辑任务边界
- 可能中断复杂的多阶段任务

在逻辑边界的战略压缩：
- **探索后，执行前** -- 压缩调查上下文，保留实现计划
- **里程碑完成后** -- 为下一阶段提供新的开始
- **主要上下文切换前** -- 在开始其他任务前清理探索上下文

## 工作方式

`suggest-compact.js` 脚本在 PreToolUse（Edit/Write）中执行，执行以下操作：

1. **工具调用跟踪** -- 统计会话内的工具调用次数
2. **阈值检测** -- 在可配置阈值时建议（默认值：50次）
3. **周期性通知** -- 阈值后每25次通知一次

## Hook 设置

添加到 `~/.claude/settings.json`：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "node \"${CLAUDE_PLUGIN_ROOT}/scripts/hooks/run-with-flags.js\" \"pre:edit-write:suggest-compact\" \"scripts/hooks/suggest-compact.js\" \"standard,strict\""
          }
        ],
        "description": "Suggest manual compaction at logical intervals"
      }
    ]
  }
}
```

## 配置

环境变量：
- `COMPACT_THRESHOLD` -- 第一次建议前的工具调用次数（默认值：50）

## 压缩决策指南

使用此表决定何时压缩：

| 阶段切换 | 压缩？ | 理由 |
|-----------------|----------|-----|
| 调查 -> 计划 | 是 | 调查上下文体积大，计划是提炼的产物 |
| 计划 -> 实现 | 是 | 计划在 TodoWrite 或文件中，确保代码的上下文空间 |
| 实现 -> 测试 | 视情况而定 | 如果测试引用最近代码则保留；焦点切换时压缩 |
| 调试 -> 下一功能 | 是 | 调试跟踪会污染不相关任务的上下文 |
| 实现中途 | 否 | 丢失变量名、文件路径、部分状态的成本很大 |
| 失败的尝试后 | 是 | 在尝试新方法前清理死胡同的推理 |

## 压缩中保留的内容

理解保留什么可以让您自信地进行压缩：

| 保留 | 丢失 |
|----------|------|
| CLAUDE.md 指令 | 中间推理及分析 |
| TodoWrite 任务列表 | 之前读取的文件内容 |
| 内存文件（`~/.claude/memory/`） | 多阶段对话上下文 |
| Git 状态（提交、分支） | 工具调用记录及次数 |
| 磁盘上的文件 | 口头提及的细微用户偏好 |

## 最佳实践

1. **计划后压缩** -- 计划在 TodoWrite 确定后，为新开始进行压缩
2. **调试后压缩** -- 在继续前清理错误解决上下文
3. **实现中途不压缩** -- 保留相关变更的上下文
4. **阅读建议** -- Hook 告诉您 *何时*，您决定 *是否* 执行
5. **压缩前记录** -- 压缩前将重要上下文保存到文件或内存
6. **使用带摘要的 `/compact`** -- 添加自定义消息：`/compact Focus on implementing auth middleware next`

## 相关项目

- [The Longform Guide](https://x.com/affaanmustafa/status/2014040193557471352) -- tokens 优化部分
- 内存持久化 Hook -- 用于在压缩中存活的状态
- `continuous-learning` 技能 -- 会话结束前提取模式
