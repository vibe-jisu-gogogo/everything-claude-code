---
paths:
  - "**/*.ts"
  - "**/*.tsx"
  - "**/*.js"
  - "**/*.jsx"
---
# TypeScript/JavaScript 安全

> 此文件扩展了 [common/security.md](../common/security.md)，添加了 TypeScript/JavaScript 特定内容。

## Secret 管理

```typescript
// 绝不：硬编码 secret
const apiKey = "sk-proj-xxxxx"

// 始终：Environment variable
const apiKey = process.env.OPENAI_API_KEY

if (!apiKey) {
  throw new Error('OPENAI_API_KEY not configured')
}
```

## Agent 支持

- 使用 **security-reviewer** skill 进行全面的安全检查
