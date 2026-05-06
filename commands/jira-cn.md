---
description: 获取 Jira 工单、分析需求、更新状态或添加评论。使用 jira-integration 技能以及 MCP 或 REST API。
---

# Jira 命令

直接在工作流中与 Jira 工单交互 — 获取工单、分析需求、添加评论以及切换状态。

## 用法

```
/jira get <TICKET-KEY>          # 获取并分析工单
/jira comment <TICKET-KEY>      # 添加进度评论
/jira transition <TICKET-KEY>   # 变更工单状态
/jira search <JQL>              # 使用 JQL 搜索问题
```

## 命令功能

1. **获取与分析** — 拉取 Jira 工单并提取需求、验收标准、测试场景和依赖项
2. **评论** — 向工单添加结构化的进度更新
3. **状态流转** — 将工单在工作流状态间移动（待办 → 进行中 → 已完成）
4. **搜索** — 使用 JQL 查询查找问题

## 工作原理

### `/jira get <TICKET-KEY>`

1. 从 Jira 拉取工单（通过 MCP `jira_get_issue` 或 REST API）
2. 提取所有字段：摘要、描述、验收标准、优先级、标签、关联问题
3. 可选择拉取评论以获取更多上下文
4. 生成结构化分析：

```
Ticket: PROJ-1234
Summary: [标题]
Status: [状态]
Priority: [优先级]
Type: [Story/Bug/Task]

Requirements:
1. [提取的需求]
2. [提取的需求]

Acceptance Criteria:
- [ ] [工单中的验收标准]

Test Scenarios:
- 正常路径: [描述]
- 错误场景: [描述]
- 边界场景: [描述]

Dependencies:
- [关联问题、API、服务]

Recommended Next Steps:
- /plan 创建实现方案
- `tdd-workflow` 技能用于测试优先的实现
```

### `/jira comment <TICKET-KEY>`

1. 总结当前会话进度（已构建、已测试、已提交的内容）
2. 格式化为结构化评论
3. 发布到 Jira 工单

### `/jira transition <TICKET-KEY>`

1. 拉取工单可用的状态流转选项
2. 向用户展示选项
3. 执行选中的状态流转

### `/jira search <JQL>`

1. 向 Jira 执行 JQL 查询
2. 返回匹配问题的汇总表格

## 前置条件

本命令需要 Jira 凭证。选择以下任一方式：

**选项 A — MCP 服务器（推荐）：**
将 `jira` 添加到你的 `mcpServers` 配置中（模板参见 `mcp-configs/mcp-servers.json`）。

**选项 B — 环境变量：**
```bash
export JIRA_URL="https://yourorg.atlassian.net"
export JIRA_EMAIL="your.email@example.com"
export JIRA_API_TOKEN="your-api-token"
```

如果缺少凭证，停止操作并引导用户进行配置。

## 与其他命令的集成

分析工单后：
- 使用 `/plan` 根据需求创建实现方案
- 使用 `tdd-workflow` 技能进行测试驱动开发
- 实现完成后使用 `/code-review`
- 使用 `/jira comment` 将进度同步回工单
- 工作完成后使用 `/jira transition` 切换工单状态

## 相关资源

- **技能：** `skills/jira-integration/`
- **MCP 配置：** `mcp-configs/mcp-servers.json` → `jira`
