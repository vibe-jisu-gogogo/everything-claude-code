---
name: documentation-lookup
description: 借助 Context7 MCP 使用最新的库和框架文档，而非训练数据。在遇到安装设置问题、API 参考、代码示例，或者用户提及特定框架（如 React、Next.js、Prisma）时激活本技能。
---

# Documentation Lookup (Context7)

当用户询问有关库、框架或 API 的问题时，通过 Context7 MCP（工具 `resolve-library-id` 和 `query-docs`）获取最新文档，而非依赖训练数据。

## 核心概念

- **Context7**：提供实时文档的 MCP 服务；查询库和 API 相关内容时使用它替代训练数据。
- **resolve-library-id**：根据库名和查询内容返回兼容 Context7 的库 ID（例如 `/vercel/next.js`）。
- **query-docs**：根据指定的库 ID 和问题获取文档和代码片段。请始终先调用 resolve-library-id 获取有效的库 ID。

## 适用场景

当用户出现以下行为时激活本技能：

- 询问安装设置或配置相关问题（例如“我该如何配置 Next.js middleware？”）
- 需要依赖特定库的代码（“为…编写一个 Prisma 查询”）
- 需要 API 或参考信息（“Supabase 有哪些 auth 方法？”）
- 提及特定框架或库（React、Vue、Svelte、Express、Tailwind、Prisma、Supabase 等）

每当请求需要依赖库、框架或 API 准确的最新行为时，请使用本技能。本技能适用于所有配置了 Context7 MCP 的环境（例如 Claude Code、Cursor、Codex）。

## 工作流程

### 步骤1：解析 Library ID

调用 **resolve-library-id** MCP 工具，传入以下参数：

- **libraryName**：从用户问题中提取的库或产品名称（例如 `Next.js`、`Prisma`、`Supabase`）。
- **query**：用户的完整问题。这可以提升结果的相关性排序。

在查询文档之前，你必须获取兼容 Context7 的库 ID（格式为 `/org/project` 或 `/org/project/version`）。未通过此步骤获取有效库 ID 时，请勿调用 query-docs。

### 步骤2：选择最佳匹配

从解析结果中，依据以下规则选择一个结果：

- **名称匹配**：优先选择与用户询问内容完全匹配或最接近的结果。
- **基准评分**：评分越高表示文档质量越好（满分为100）。
- **来源可信度**：可用时优先选择高或中等可信度的来源。
- **版本**：如果用户指定了版本（例如“React 19”、“Next.js 15”），优先选择列表中对应特定版本的库 ID（例如 `/org/project/v1.2.0`）。

### 步骤3：获取文档

调用 **query-docs** MCP 工具，传入以下参数：

- **libraryId**：步骤2中选择的 Context7 库 ID（例如 `/vercel/next.js`）。
- **query**：用户的具体问题或任务。表述尽量具体以获取相关的代码片段。

限制：每个问题调用 query-docs（或 resolve-library-id）的次数不要超过3次。如果3次调用后答案仍不明确，请说明不确定性，使用你掌握的最佳信息作答，不要凭空猜测。

### 步骤4：使用文档内容

- 使用获取到的最新信息回答用户问题。
- 有帮助时可包含文档中的相关代码示例。
- 必要时注明库或版本（例如“在 Next.js 15 中…”）。

## 示例

### 示例：Next.js middleware

1. 调用 **resolve-library-id**，传入 `libraryName: "Next.js"`，`query: "How do I set up Next.js middleware?"`。
2. 从结果中，根据名称和基准评分选择最佳匹配（例如 `/vercel/next.js`）。
3. 调用 **query-docs**，传入 `libraryId: "/vercel/next.js"`，`query: "How do I set up Next.js middleware?"`。
4. 使用返回的片段和文本作答；相关情况下可包含文档中最小化的 `middleware.ts` 示例。

### 示例：Prisma 查询

1. 调用 **resolve-library-id**，传入 `libraryName: "Prisma"`，`query: "How do I query with relations?"`。
2. 选择官方 Prisma 库 ID（例如 `/prisma/prisma`）。
3. 使用该 `libraryId` 和查询内容调用 **query-docs**。
4. 返回 Prisma Client 模式（例如 `include` 或 `select`），并附上文档中的简短代码片段。

### 示例：Supabase auth 方法

1. 调用 **resolve-library-id**，传入 `libraryName: "Supabase"`，`query: "What are the auth methods?"`。
2. 选择 Supabase 文档库 ID。
3. 调用 **query-docs**；总结 auth 方法并展示从获取的文档中提取的最小化示例。

## 最佳实践

- **表述具体**：尽可能使用用户的完整问题作为查询内容，以获得更好的相关性。
- **版本感知**：当用户提及版本时，解析步骤中如有可用的对应版本的库 ID 请优先使用。
- **优先官方来源**：存在多个匹配项时，优先选择官方或主包，而非社区分叉版本。
- **无敏感数据**：发送到 Context7 的所有查询中，请删除 API 密钥、密码、token 和其他机密信息。在将用户问题传递给 resolve-library-id 或 query-docs 之前，请检查其中是否可能包含机密信息。
