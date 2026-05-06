# 安全策略

## 支持的版本

| 版本 | 支持状态 |
| ------- | ------------------ |
| 1.9.x   | :white_check_mark: |
| 1.8.x   | :white_check_mark: |
| < 1.8   | :x:                |

## 报告漏洞

如果你在 ECC 中发现安全漏洞，请负责任地进行报告。

**不要为安全漏洞公开创建 GitHub Issue。**

相反，请发送邮件至 **<security@ecc.tools>**，并包含以下内容：

- 漏洞描述
- 复现步骤
- 受影响的版本
- 任何潜在的影响评估

你可以预期：

- **48小时内**收到确认回复
- **7天内**收到状态更新
- 严重问题**30天内**得到修复或缓解方案

如果漏洞被接受，我们将：

- 在发布说明中注明你的贡献（除非你希望匿名）
- 及时修复问题
- 与你协调披露时间

如果漏洞被拒绝，我们将解释原因，并指导你是否应该向其他地方报告。

## 适用范围

本政策涵盖：

- ECC 插件及本仓库中的所有脚本
- 在你的机器上执行的 Hook 脚本
- 安装/卸载/修复生命周期脚本
- ECC 附带的 MCP 配置
- AgentShield 安全扫描器 ([github.com/affaan-m/agentshield](https://github.com/affaan-m/agentshield))

## 安全资源

- **AgentShield**: 扫描你的 Agent 配置是否存在漏洞 — `npx ecc-agentshield scan`
- **安全指南**: [Agentic 安全速通指南](./the-security-guide.md)
- **OWASP MCP Top 10**: [owasp.org/www-project-mcp-top-10](https://owasp.org/www-project-mcp-top-10/)
- **OWASP Agentic 应用 Top 10**: [genai.owasp.org](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/)
