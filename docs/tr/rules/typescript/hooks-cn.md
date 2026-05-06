---
paths:
  - "**/*.ts"
  - "**/*.tsx"
  - "**/*.js"
  - "**/*.jsx"
---
# TypeScript/JavaScript Hooks

> 此文件使用 TypeScript/JavaScript 特定内容扩展了 [common/hooks.md](../common/hooks.md) 文件。

## PostToolUse Hooks

在 `~/.claude/settings.json` 中配置：

- **Prettier**：Edit 后自动格式化 JS/TS 文件
- **TypeScript check**：编辑 `.ts`/`.tsx` 文件后运行 `tsc`
- **console.log 提醒**：编辑文件时提醒有关 `console.log` 的内容

## Stop Hooks

- **console.log audit**：会话结束前检查所有修改文件中的 `console.log`
