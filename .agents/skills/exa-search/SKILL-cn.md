---
name: exa-search
description: 通过 Exa MCP 实现针对网页、代码和企业调研的神经搜索。适用于用户需要网页搜索、代码示例、企业情报、人员查找，或使用 Exa 神经搜索引擎开展 AI 驱动的深度调研的场景。
---

# Exa 搜索

通过 Exa MCP 服务器实现针对网页内容、代码、企业和人员的神经搜索。

## 激活时机

- 用户需要获取最新的网页信息或新闻
- 搜索代码示例、API 文档或技术参考资料
- 调研企业、竞争对手或市场参与者
- 查找专业档案或特定领域的人员
- 为任意开发任务开展背景调研
- 用户说出 "search for"、"look up"、"find" 或 "what's the latest on" 等关键词

## MCP 配置要求

必须配置 Exa MCP 服务器。添加以下配置到 `~/.claude.json`：

```json
"exa-web-search": {
  "command": "npx",
  "args": ["-y", "exa-mcp-server"],
  "env": { "EXA_API_KEY": "YOUR_EXA_API_KEY_HERE" }
}
```

前往 [exa.ai](https://exa.ai) 获取 API 密钥。

## 核心工具

### web_search_exa
用于搜索最新信息、新闻或事实的通用网页搜索工具。

```
web_search_exa(query: "latest AI developments 2026", numResults: 5)
```

**参数：**

| Param | Type | Default | Notes |
|-------|------|---------|-------|
| `query` | string | required | 搜索查询词 |
| `numResults` | number | 8 | 返回结果数量 |

### web_search_advanced_exa
支持域名和日期约束的过滤搜索。

```
web_search_advanced_exa(
  query: "React Server Components best practices",
  numResults: 5,
  includeDomains: ["github.com", "react.dev"],
  startPublishedDate: "2025-01-01"
)
```

**参数：**

| Param | Type | Default | Notes |
|-------|------|---------|-------|
| `query` | string | required | 搜索查询词 |
| `numResults` | number | 8 | 返回结果数量 |
| `includeDomains` | string[] | none | 限制仅搜索指定域名 |
| `excludeDomains` | string[] | none | 排除指定域名 |
| `startPublishedDate` | string | none | ISO 格式日期过滤（起始时间） |
| `endPublishedDate` | string | none | ISO 格式日期过滤（结束时间） |

### get_code_context_exa
从 GitHub、Stack Overflow 和文档站点查找代码示例和文档。

```
get_code_context_exa(query: "Python asyncio patterns", tokensNum: 3000)
```

**参数：**

| Param | Type | Default | Notes |
|-------|------|---------|-------|
| `query` | string | required | 代码或 API 搜索查询词 |
| `tokensNum` | number | 5000 | 内容 tokens 数量（1000-50000） |

### company_research_exa
为商业情报和新闻需求调研企业信息。

```
company_research_exa(companyName: "Anthropic", numResults: 5)
```

**参数：**

| Param | Type | Default | Notes |
|-------|------|---------|-------|
| `companyName` | string | required | 企业名称 |
| `numResults` | number | 5 | 返回结果数量 |

### people_search_exa
查找专业档案和个人简介。

```
people_search_exa(query: "AI safety researchers at Anthropic", numResults: 5)
```

### crawling_exa
从 URL 提取完整页面内容。

```
crawling_exa(url: "https://example.com/article", tokensNum: 5000)
```

**参数：**

| Param | Type | Default | Notes |
|-------|------|---------|-------|
| `url` | string | required | 待提取内容的 URL |
| `tokensNum` | number | 5000 | 内容 tokens 数量 |

### deep_researcher_start / deep_researcher_check
启动异步运行的 AI research agent。

```
# 启动调研
deep_researcher_start(query: "comprehensive analysis of AI code editors in 2026")

# 检查状态（完成后返回结果）
deep_researcher_check(researchId: "<id from start>")
```

## 使用模式

### 快速查找
```
web_search_exa(query: "Node.js 22 new features", numResults: 3)
```

### 代码调研
```
get_code_context_exa(query: "Rust error handling patterns Result type", tokensNum: 3000)
```

### 企业尽职调查
```
company_research_exa(companyName: "Vercel", numResults: 5)
web_search_advanced_exa(query: "Vercel funding valuation 2026", numResults: 3)
```

### 技术深度调研
```
# 启动异步调研
deep_researcher_start(query: "WebAssembly component model status and adoption")
# ... 处理其他工作 ...
deep_researcher_check(researchId: "<id>")
```

## 提示要点

- 通用查询使用 `web_search_exa`，需要过滤结果时使用 `web_search_advanced_exa`
- 查找精准代码片段时使用较小的 `tokensNum`（1000-2000），需要完整上下文时使用更大的值（5000+）
- 结合 `company_research_exa` 和 `web_search_advanced_exa` 开展全面的企业分析
- 使用 `crawling_exa` 获取搜索结果中特定 URL 的完整内容
- `deep_researcher_start` 最适合需要 AI 综合整理的全面性主题

## 相关技能

- `deep-research` — 结合 firecrawl + exa 的完整 research workflow
- `market-research` — 带决策框架的商业导向调研
