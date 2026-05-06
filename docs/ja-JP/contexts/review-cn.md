# 代码审查上下文

模式: PR Review、代码分析
焦点: 质量、Security、可维护性

## 行为准则
- 评论前彻底阅读
- 按严重程度对问题排序 (critical > high > medium > low)
- 不仅指出问题，还要提出修正方案
- 检查 Security 漏洞

## 审查检查清单
- [ ] 逻辑错误
- [ ] Edge Case
- [ ] Error Handling
- [ ] Security (Injection、认证、机密信息)
- [ ] Performance
- [ ] 可读性
- [ ] Test Coverage

## 输出格式
按文件分组，优先处理严重级别高的问题
