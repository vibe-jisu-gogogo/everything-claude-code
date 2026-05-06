# 安全政策

## 支持版本

| 版本     | 支持状态           |
| -------- | ------------------ |
| 1.9.x    | :white_check_mark: |
| 1.8.x    | :white_check_mark: |
| < 1.8    | :x:                |

## 报告漏洞

如果您在 ECC 中发现安全漏洞，请负责任地报告。

**请勿为安全漏洞创建公开的 GitHub issue。**

请发送邮件至 **<security@ecc.tools>**，邮件中需包含：

- 漏洞描述
- 复现步骤
- 受影响的版本
- 任何潜在的影响评估

您可以期待：

- **确认通知**：48 小时内
- **状态更新**：7 天内
- **修复或缓解措施**：对于 critical 问题，30 天内

如果漏洞被采纳，我们将：

- 在 release notes 中注明您的贡献（除非您希望匿名）
- 及时修复问题
- 与您协调披露时间

如果漏洞被拒绝，我们将解释原因，并提供是否应向其他地方报告的指导。

## 范围

本政策涵盖：

- ECC plugin 及此仓库中的所有 scripts
- 在您机器上执行的 hook scripts
- Install/uninstall/repair lifecycle scripts
- 随 ECC 分发的 MCP configurations
- AgentShield security scanner ([github.com/affaan-m/agentshield](https://github.com/affaan-m/agentshield))

## 安全资源

- **AgentShield**：扫描您的 agent config 以查找漏洞 — `npx ecc-agentshield scan`
- **安全指南**：[The Shorthand Guide to Everything Agentic Security](./the-security-guide.md)
- **OWASP MCP Top 10**：[owasp.org/www-project-mcp-top-10](https://owasp.org/www-project-mcp-top-10/)
- **OWASP Agentic Applications Top 10**：[genai.owasp.org](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/)
