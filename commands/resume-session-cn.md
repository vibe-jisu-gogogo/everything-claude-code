---
description: 从 ~/.claude/session-data/ 加载最近的会话文件，从上一个会话结束的地方恢复完整上下文继续工作。
---

# 恢复会话命令

加载上次保存的会话状态，在开始任何工作前全面了解当前进度。
此命令是 `/save-session` 的对应命令。

## 何时使用

- 开启新会话，继续前一天的工作
- 由于上下文限制开启了全新会话后
- 当从其他来源收到会话文件时（只需提供文件路径）
- 任何时候你有一个会话文件，希望 Claude 在继续前完全吸收其内容

## 用法

```
/resume-session                                                      # 加载 ~/.claude/session-data/ 中最近的文件
/resume-session 2024-01-15                                           # 加载该日期最近的会话
/resume-session ~/.claude/session-data/2024-01-15-abc123de-session.tmp  # 加载当前短 ID 格式的会话文件
/resume-session ~/.claude/sessions/2024-01-15-session.tmp               # 加载特定旧格式文件
```

## 流程

### 步骤 1：查找会话文件

如果未提供参数：

1. 检查 `~/.claude/session-data/`
2. 选择最近修改的 `*-session.tmp` 文件
3. 如果文件夹不存在或没有匹配文件，告知用户：
   ```
   在 ~/.claude/session-data/ 中未找到会话文件
   请在会话结束时运行 /save-session 创建会话文件。
   ```
   然后停止执行。

如果提供了参数：

- 如果看起来是日期格式（`YYYY-MM-DD`），先搜索 `~/.claude/session-data/`，再搜索旧格式的 `~/.claude/sessions/`，查找匹配 `YYYY-MM-DD-session.tmp`（旧格式）或 `YYYY-MM-DD-<shortid>-session.tmp`（当前格式）的文件，加载该日期最近修改的版本
- 如果看起来是文件路径，直接读取该文件
- 如果未找到，清晰报告并停止执行

### 步骤 2：读取完整会话文件

读取整个文件。暂不进行摘要。

### 步骤 3：确认理解

按照以下精确格式返回结构化简报：

```
SESSION LOADED: [文件的实际解析路径]
════════════════════════════════════════════════

PROJECT: [文件中的项目名称/主题]

WHAT WE'RE BUILDING:
[用你自己的话写 2-3 句摘要]

CURRENT STATE:
PASS: 已完成: [已确认的项目数量]
 进行中: [列出进行中的文件]
 未开始: [列出已计划但未触及的项目]

WHAT NOT TO RETRY:
[列出每个失败的方法及其原因——这一点至关重要]

OPEN QUESTIONS / BLOCKERS:
[列出任何阻碍或未回答的问题]

NEXT STEP:
[如果文件中定义了明确的下一步，列出]
[如果未定义："未定义下一步——建议在开始前一起查看'尚未尝试的内容'部分"]

════════════════════════════════════════════════
准备好继续了。你想做什么？
```

### 步骤 4：等待用户指令

**不要自动开始工作。不要修改任何文件。** 等待用户告知下一步要做什么。

如果会话文件中明确定义了下一步，且用户说"continue"、"yes"或类似内容——按照定义的精确步骤执行。

如果未定义下一步——询问用户从哪里开始，可以选择从"尚未尝试的内容"部分建议一个方法。

---

## 边缘情况

**同一日期有多个会话**（`2024-01-15-session.tmp`、`2024-01-15-abc123de-session.tmp`）：
加载该日期最近修改的匹配文件，无论它使用的是旧的无 ID 格式还是当前的短 ID 格式。

**会话文件引用的文件不再存在：**
在简报中注明——"WARNING: `path/to/file.ts` 在会话中被引用但在磁盘上未找到。"

**会话文件是 7 天前的：**
注明时间间隔——"WARNING: 这个会话是 N 天前的（阈值：7 天）。内容可能已经变更。"——然后正常继续。

**用户直接提供文件路径（例如，从队友转发）：**
读取文件并遵循相同的简报流程——无论来源如何，格式都相同。

**会话文件为空或格式错误：**
报告："找到会话文件但似乎为空或无法读取。你可能需要使用 /save-session 创建新的会话文件。"

---

## 输出示例

```
SESSION LOADED: /Users/you/.claude/session-data/2024-01-15-abc123de-session.tmp
════════════════════════════════════════════════

PROJECT: my-app — JWT Authentication

WHAT WE'RE BUILDING:
用户认证系统，JWT 令牌存储在 httpOnly cookies 中。
注册和登录端点已部分完成。通过中间件的路由保护尚未开始。

CURRENT STATE:
PASS: 已完成: 3 项（注册端点、JWT 生成、密码哈希）
 进行中: app/api/auth/login/route.ts（令牌有效，尚未设置 cookie）
 未开始: middleware.ts、app/login/page.tsx

WHAT NOT TO RETRY:
FAIL: Next-Auth — 与自定义 Prisma adapter 冲突，每次请求都抛出 adapter 错误
FAIL: localStorage 存储 JWT — 导致 SSR hydration 不匹配，与 Next.js 不兼容

OPEN QUESTIONS / BLOCKERS:
- cookies().set() 在 Route Handler 中是否有效？还是仅在 Server Actions 中有效？

NEXT STEP:
在 app/api/auth/login/route.ts 中——使用以下代码将 JWT 设置为 httpOnly cookie：
cookies().set('token', jwt, { httpOnly: true, secure: true, sameSite: 'strict' })
然后用 Postman 测试响应中是否有 Set-Cookie 头。

════════════════════════════════════════════════
准备好继续了。你想做什么？
```

---

## 注意事项

- 加载会话文件时切勿修改它——它是只读的历史记录
- 简报格式是固定的——即使部分内容为空也不要跳过
- "What Not To Retry" 必须始终显示，即使内容仅为"None"——这部分非常重要，不能遗漏
- 恢复会话后，用户可能希望在新会话结束时再次运行 `/save-session` 以创建新的日期文件