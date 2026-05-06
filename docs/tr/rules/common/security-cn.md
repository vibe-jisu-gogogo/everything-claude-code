# 安全规则

## 强制安全检查

提交 ANY 之前：
- [ ] 无硬编码 secret (API keys, passwords, tokens)
- [ ] 所有用户输入已 validated
- [ ] SQL injection 防护 (参数化查询)
- [ ] XSS 防护 (sanitized HTML)
- [ ] CSRF 保护已启用
- [ ] Authentication/authorization 已验证
- [ ] 所有 endpoints 已启用 rate limiting
- [ ] 错误消息不泄露敏感数据

## Secret 管理

- 永远不要在源代码中 hardcode secrets
- 始终使用 environment variables 或 secret manager
- 在启动时 validate 所需的 secrets 存在
- Rotate 可能已泄露的 secrets

## 安全响应协议

发现安全问题时：
1. 立即停止
2. 使用 **security-reviewer** agent
3. 在继续之前修复 CRITICAL 问题
4. Rotate 已泄露的 secrets
5. 审查整个代码库以查找类似问题
