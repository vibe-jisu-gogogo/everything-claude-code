# 安全指南

## 必需安全检查

每次提交前：
- [ ] 是否存在硬编码的 secrets（API keys、passwords、tokens）
- [ ] 所有用户输入是否已验证
- [ ] 是否已防止 SQL injection（使用参数化查询）
- [ ] 是否已防止 XSS（HTML sanitizing）
- [ ] CSRF protection 是否已启用
- [ ] authentication/authorization 是否已验证
- [ ] 所有 endpoints 是否都有限速
- [ ] 错误消息是否不会暴露敏感数据

## Secrets 管理

- 绝不在源代码中硬编码 secrets
- 始终使用环境变量或 secrets manager
- 启动时验证所需的 secrets 是否存在
- 更换可能已暴露的 secrets

## 安全响应协议

发现安全问题时：
1. 立即停止
2. 使用 **security-reviewer** agent
3. 继续前先修复关键问题
4. 更换已暴露的 secrets
5. 检查整个代码库是否存在类似问题
