# 通用模式

## Skeleton Project

实现新功能时：
1. 搜索经过验证的 skeleton project
2. 使用 parallel agent 评估选项：
   - 安全评估
   - 可扩展性分析
   - 相关性评分
   - 实现计划
3. 基于最合适的方案进行 clone
4. 在经过验证的结构内迭代改进

## 设计模式

### Repository Pattern

将数据访问封装在一致的接口之后：
- 定义标准操作：findAll, findById, create, update, delete
- 具体实现处理存储细节（database, API, 文件等）
- 业务逻辑依赖抽象接口而非存储机制
- 可轻松更换数据源并通过 mocking 简化测试

### API Response 格式

对所有 API response 使用一致的 envelope：
- 包含 success/status 指示器
- 包含 data payload（出错时为 null）
- 包含 error message 字段（成功时为 null）
- pagination response 包含 metadata（total, page, limit）
