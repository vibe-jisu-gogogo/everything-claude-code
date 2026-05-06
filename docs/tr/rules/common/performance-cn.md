# 性能优化

## 模型选择策略

**Haiku 4.5** (Sonnet 容量的 90%，3 倍成本节省):
- 频繁调用的轻量 agents
- 结对编程和代码生成
- 多代理系统中的 worker agents

**Sonnet 4.6** (最佳编码模型):
- 核心开发工作
- 多代理工作流编排
- 复杂编码任务

**Opus 4.5** (最深层推理):
- 复杂架构决策
- 最大推理需求
- 研究和分析任务

## Context Window 管理

避免使用 context window 最后 20% 的任务:
- 大规模重构
- 跨多个文件的功能实现
- 复杂交互的调试

对 context 敏感性较低的任务:
- 单文件编辑
- 创建独立的 utility
- 文档更新
- 简单错误修复

## Extended Thinking + Plan Mode

Extended thinking 默认启用，为内部推理预留高达 31,999 个 tokens。

Extended thinking 控制:
- **切换**: Option+T (macOS) / Alt+T (Windows/Linux)
- **配置**: 在 `~/.claude/settings.json` 中设置 `alwaysThinkingEnabled`
- **预算上限**: `export MAX_THINKING_TOKENS=10000`
- **详细模式**: 按 Ctrl+O 查看 Thinking 输出

对于需要深度推理的复杂任务:
1. 确保 Extended thinking 已启用（默认开启）
2. 启用 **Plan Mode** 以获得结构化方法
3. 使用多个关键回合进行全面分析
4. 使用分角色的 sub-agents 获得多种视角

## Build 故障排除

如果 Build 失败:
1. 使用 **build-error-resolver** agent
2. 分析错误消息
3. 逐步修复
4. 每次修复后验证
