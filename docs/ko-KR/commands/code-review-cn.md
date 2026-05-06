# 代码审查

对未提交的更改执行全面的安全和质量审查：

1. 查看变更文件列表: git diff --name-only HEAD

2. 对每个变更文件检查以下内容：

**安全问题 (CRITICAL):**
- 硬编码的认证信息、API密钥、令牌
- SQL注入漏洞
- XSS漏洞
- 缺失的输入验证
- 不安全的依赖项
- 路径遍历(Path Traversal)风险

**代码质量 (HIGH):**
- 超过50行的函数
- 超过800行的文件
- 超过4层的嵌套深度
- 缺失的错误处理
- 调试日志语句（例如：开发用日志/print等）
- TODO/FIXME注释
- 活跃语言的公开API文档缺失（例如：JSDoc/Go doc/Docstring等）

**最佳实践 (MEDIUM):**
- 变异(Mutation)模式（请使用不可变模式）
- 代码/注释中使用Emoji
- 新代码缺少测试
- 可访问性(a11y)问题

3. 生成包含以下内容的报告：
   - 严重程度: CRITICAL, HIGH, MEDIUM, LOW
   - 文件位置和行号
   - 问题描述
   - 修复建议

4. 如果发现CRITICAL或HIGH级别问题，阻止commit

永远不要批准存在安全漏洞的代码！
