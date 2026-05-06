# 安全指南

## 强制安全检查

在任何提交之前：
- [ ] 没有硬编码的 secrets（API keys、密码、tokens）
- [ ] 所有用户输入已验证
- [ ] SQL 注入防护（参数化查询）
- [ ] XSS 防护（HTML 已清理）
- [ ] CSRF 保护已启用
- [ ] 身份认证/授权已验证
- [ ] 所有端点已启用 rate limiting
- [ ] 错误消息不泄露敏感数据

## Secrets 管理

- 绝不在源代码中硬编码 secrets
- 始终使用环境变量或 secrets 管理器
- 启动时验证所需的 secrets 是否存在
- 轮换任何可能已暴露的 secrets

## 安全响应协议

如果发现安全问题：
1. 立即停止
2. 使用 **security-reviewer** agent
3. 在继续之前修复关键问题
4. 轮换所有已暴露的 secrets
5. 检查整个代码库中是否存在类似问题
