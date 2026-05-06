---
description: 列出所有已配置的 hookify 规则
---

查找并以格式化表格形式展示所有 hookify 规则。

## 步骤

1. 查找所有 `.claude/hookify.*.local.md` 文件
2. 读取每个文件的 frontmatter：
   - `name`
   - `enabled`
   - `event`
   - `action`
   - `pattern`
3. 将它们展示为表格：

| Rule | Enabled | Event | Pattern | File |
|------|---------|-------|---------|------|

4. 显示规则数量并提醒用户之后可以使用 `/hookify-configure` 修改状态。
