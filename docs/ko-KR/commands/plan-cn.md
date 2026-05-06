---
description: 重新确认需求，评估风险，编写分阶段实施计划。在修改代码前等待用户确认。
---

# Plan 命令

此命令调用 **planner** 代理，在编写代码前创建全面的实施计划。

## 此命令的功能

1. **重新确认需求** - 明确需要创建的内容
2. **识别风险** - 发现潜在问题和阻塞因素
3. **编写分阶段计划** - 将实施分解为多个步骤
4. **等待确认** - 继续之前必须获得用户批准

## 使用时机

应使用 `/plan` 的情况：
- 开始新功能时
- 进行重要架构变更时
- 进行复杂重构工作时
- 影响多个文件/组件时
- 需求不明确或模糊时

## 工作方式

planner 代理执行的任务：

1. **分析** 请求并以明确术语重新确认需求
2. **分解** 为具体、可执行的步骤
3. 识别组件之间的 **依赖关系**
4. **评估风险** 并识别潜在阻塞因素
5. **估算复杂度** (High/Medium/Low)
6. **提出** 计划并 **等待** 明确确认

## 使用示例

```
User: /plan 市场解决时需要添加实时通知

Agent (planner):
# 实施计划：实时市场解决通知

## 需求重新确认
- 用户关注的市场解决时发送通知
- 支持多通知渠道（应用内、电子邮件、webhook）
- 确保通知稳定传递
- 包含市场结果和用户持仓结果

## 实施阶段

### 第1阶段：数据库架构
- 添加包含 id, user_id, market_id, type, status, created_at 列的 notifications 表
- 添加用于渠道设置的 user_notification_preferences 表
- 为 user_id 和 market_id 创建索引以提高性能

### 第2阶段：通知服务
- 在 lib/notifications.ts 中创建通知服务
- 使用 BullMQ/Redis 实现通知队列
- 添加发送失败时的重试逻辑
- 创建通知模板

### 第3阶段：集成点
- 连接到市场解决逻辑（当状态更改为 "resolved" 时）
- 查询在该市场有持仓的所有用户
- 将每个用户的通知添加到队列

### 第4阶段：前端组件
- 在页眉创建 NotificationBell 组件
- 添加 NotificationList 模态框
- 通过 Supabase subscriptions 实现实时更新
- 添加通知设置页面

## 依赖项
- Redis（用于队列）
- 电子邮件服务（SendGrid/Resend）
- Supabase 实时 subscriptions

## 风险
- HIGH：电子邮件送达率（需要 SPF/DKIM）
- MEDIUM：每个市场超过 1000 名用户时的性能
- MEDIUM：市场频繁解决时的通知垃圾邮件
- LOW：实时 subscription 开销

## 预计复杂度：MEDIUM
- 后端：4-6小时
- 前端：3-4小时
- 测试：2-3小时
- 总计：9-13小时

**等待确认**：是否按此计划继续？(yes/no/modify)
```

## 重要注意事项

**核心**：planner 代理在收到 "yes" 或 "proceed" 等肯定响应以明确确认计划之前，**绝对不会**编写代码。

如果想要更改，请如下响应：
- "modify: [更改内容]"
- "different approach: [替代方案]"
- "skip phase 2 and do phase 3 first"

## 与其他命令的关联

制定计划后：
- 使用 `/tdd` 以测试驱动开发方式实施
- 发生构建错误时使用 `/build-fix`
- 使用 `/code-review` 审查完成的实施

## 相关代理

此命令调用以下位置的 `planner` 代理：
`~/.claude/agents/planner.md`
