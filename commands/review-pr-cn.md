---
description: 使用专用代理进行全面的PR评审
---

对pull request进行全面的多视角评审。

## 使用方法

`/review-pr [PR-number-or-URL] [--focus=comments|tests|errors|types|code|simplify]`

如果未指定PR，则评审当前分支的PR。如果未指定焦点，则运行完整的评审栈。

## 步骤

1. 识别PR：
   - 使用 `gh pr view` 获取PR详情、变更文件和diff
2. 查找项目指南：
   - 寻找 `CLAUDE.md`、lint配置、TypeScript配置、仓库规范
3. 运行专用评审代理：
   - `code-reviewer`
   - `comment-analyzer`
   - `pr-test-analyzer`
   - `silent-failure-hunter`
   - `type-design-analyzer`
   - `code-simplifier`
4. 汇总结果：
   - 去重重叠的发现
   - 按严重性排序
5. 按严重性分组报告发现

## 置信度规则

仅报告置信度 >= 80的问题：

- Critical：bug、安全问题、数据丢失
- Important：缺少测试、质量问题、风格违规
- Advisory：仅在明确要求时提供建议
