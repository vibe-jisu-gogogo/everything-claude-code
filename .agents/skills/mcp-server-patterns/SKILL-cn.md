---
name: mcp-server-patterns
description: 使用Node/TypeScript SDK构建MCP服务器——包含工具、资源、提示词、Zod校验、stdio与Streamable HTTP方案对比。如需最新API，请参考Context7或官方MCP文档。
---

# MCP 服务器开发模式

Model Context Protocol (MCP) 允许AI助手调用你的服务器上的工具、读取资源、使用提示词。构建或维护MCP服务器时可使用此技能。SDK API处于迭代中，请查阅Context7（查询文档关键词“MCP”）或官方MCP文档获取当前的方法名和签名。

## 适用场景

当你需要实现新的MCP服务器、添加工具或资源、选择stdio或HTTP传输方式、升级SDK、调试MCP注册和传输问题时使用。

## 工作原理

### 核心概念

- **Tools**：模型可以调用的操作（例如搜索、执行命令）。根据SDK版本不同，使用`registerTool()`或`tool()`进行注册。
- **Resources**：模型可以获取的只读数据（例如文件内容、API响应）。使用`registerResource()`或`resource()`进行注册。处理函数通常会接收`uri`参数。
- **Prompts**：可复用的参数化提示词模板，客户端可以直接展示使用（例如在Claude Desktop中）。使用`registerPrompt()`或等效方法进行注册。
- **Transport**：本地客户端使用stdio传输（例如Claude Desktop）；远程场景（Cursor、云端）优先使用Streamable HTTP。传统HTTP/SSE仅用于向后兼容。

Node/TypeScript SDK可能会提供`tool()`/`resource()`或者`registerTool()`/`registerResource()`方法，官方SDK一直处于迭代中。请始终对照最新的[MCP文档](https://modelcontextprotocol.io)或Context7确认用法。

### 使用stdio连接

对于本地客户端，创建stdio传输实例并传入服务器的connect方法。具体API因SDK版本而异（例如构造函数或工厂函数）。请查阅官方MCP文档或在Context7中查询“MCP stdio server”获取当前最佳实践。

保持服务器逻辑（工具+资源）与传输层解耦，这样你可以在入口文件中灵活切换stdio或HTTP传输方式。

### 远程传输（Streamable HTTP）

对于Cursor、云端或其他远程客户端，使用**Streamable HTTP**（当前规范下每个MCP对应单个HTTP端点）。仅当需要向后兼容时才支持传统HTTP/SSE。

## 示例

### 安装与服务器初始化

```bash
npm install @modelcontextprotocol/sdk zod
```

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { z } from "zod";

const server = new McpServer({ name: "my-server", version: "1.0.0" });
```

根据你使用的SDK版本提供的API注册工具和资源：部分版本使用`server.tool(name, description, schema, handler)`（位置参数），其他版本使用`server.tool({ name, description, inputSchema }, handler)`或者`registerTool()`。资源注册同理——如果API提供了`uri`参数，请在处理函数中包含该参数。请查阅官方MCP文档或Context7确认当前`@modelcontextprotocol/sdk`的方法签名，避免复制粘贴错误。

使用**Zod**（或SDK推荐的其他 schema 格式）进行输入校验。

## 最佳实践

- **Schema优先**：为每个工具定义输入schema；为参数和返回结构编写文档。
- **错误处理**：返回模型可识别的结构化错误或消息；避免返回原始栈追踪信息。
- **幂等性**：尽可能优先设计幂等工具，确保重试操作安全。
- **速率与成本**：对于调用外部API的工具，考虑速率限制和成本；在工具描述中注明相关信息。
- **版本管理**：在package.json中固定SDK版本；升级时查阅更新日志。

## 官方SDK与文档

- **JavaScript/TypeScript**：`@modelcontextprotocol/sdk`（npm包）。在Context7中使用库名“MCP”查询当前注册和传输模式。
- **Go**：GitHub上的官方Go SDK（`modelcontextprotocol/go-sdk`）。
- **C#**：适用于.NET的官方C# SDK。
