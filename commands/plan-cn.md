---
description: 重述需求，评估风险，并创建分步实施计划。在编写任何代码前等待用户确认。
---

# Plan 命令

该命令用于在编写任何代码前创建全面的实施计划。

默认以内联方式运行。默认不调用Task工具或任何子代理。这确保了`/plan`可以在不包含代理文件的插件安装版本中使用。

## 命令功能

1. **重述需求** - 明确需要构建的内容
2. **识别风险** - 指出潜在问题和阻碍
3. **创建分步计划** - 将实施过程拆分为多个阶段
4. **等待确认** - 继续之前必须获得用户批准

## 适用场景

使用`/plan`当：
- 开始开发新功能
- 进行重大架构变更
- 处理复杂重构任务
- 会影响多个文件/组件
- 需求不清晰或存在歧义

## 工作流程

助手将：

1. **分析请求**并以清晰的语言重述需求
2. **拆分为多个阶段**，包含具体可执行的步骤
3. **识别组件之间的依赖关系**
4. **评估风险**和潜在阻碍
5. **评估复杂度**（高/中/低）
6. **展示计划**并等待你的明确确认

## 使用示例

```
User: /plan I need to add real-time notifications when markets resolve

Assistant:
# 实施计划：市场结算实时通知

## 需求重述
- 当用户关注的市场结算时，向用户发送通知
- 支持多种通知渠道（in-app、email、webhook）
- 确保通知可靠送达
- 包含市场结果和用户的持仓收益

## 实施阶段

### 阶段1：数据库 Schema
- 添加notifications表，包含字段：id、user_id、market_id、type、status、created_at
- 添加user_notification_preferences表用于存储渠道偏好
- 为user_id和market_id创建索引以提升性能

### 阶段2：通知服务
- 在lib/notifications.ts中创建通知服务
- 使用BullMQ/Redis实现通知队列
- 为投递失败的通知添加重试逻辑
- 创建通知模板

### 阶段3：集成点
- 接入市场结算逻辑（当状态变为"resolved"时）
- 查询所有在该市场有持仓的用户
- 为每个用户将通知加入队列

### 阶段4：前端组件
- 在头部创建NotificationBell组件
- 添加NotificationList弹窗
- 通过Supabase subscriptions实现实时更新
- 添加通知偏好设置页面

## 依赖
- Redis（用于队列）
- Email服务（SendGrid/Resend）
- Supabase real-time subscriptions

## 风险
- 高：Email送达率（需要SPF/DKIM配置）
- 中：单市场1000+用户时的性能表现
- 中：如果市场频繁结算可能导致通知垃圾信息
- 低：实时订阅的开销

## 预估复杂度：中
- 后端：4-6小时
- 前端：3-4小时
- 测试：2-3小时
- 总计：9-13小时

**等待确认**：是否继续执行该计划？（是/否/修改）
```

## 重要说明

**关键提示**：在你使用"yes"、"proceed"或类似肯定回复明确确认计划之前，该命令不会编写任何代码。

如果你需要修改，请回复：
- "modify: [你的修改内容]"
- "different approach: [替代方案]"
- "skip phase 2 and do phase 3 first"

## 与其他命令的集成

计划完成后：
- 使用`tdd-workflow`技能以测试驱动开发方式实施
- 如果出现构建错误使用`/build-fix`
- 使用`/code-review`评审已完成的实现

> **需要更深度的规划？** 使用`/prp-plan`进行可产出文档的规划，包含PRD集成、代码库分析和模式提取。使用`/prp-implement`通过严格的验证循环执行这些计划。

## 可选 Planner 代理

ECC同时为包含代理文件的手动安装版本提供了`planner`代理。仅当本地运行时已暴露该子代理且用户明确要求你委托规划任务时使用。

如果`planner`子代理不可用，继续以内联方式规划，不要抛出"Agent type 'planner' not found"错误。

对于手动安装版本，源文件位于：
`agents/planner.md`
