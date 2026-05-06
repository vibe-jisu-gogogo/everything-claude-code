# 通用 Pattern

## Skeleton 项目

实现新功能时：
1. 寻找经过测试的 skeleton 项目
2. 使用并行 agent 评估选项：
   - 安全评估
   - 可扩展性分析
   - 相关性评分
   - 实施规划
3. 基于最佳匹配进行克隆
4. 在经过验证的结构内迭代

## 设计 Pattern

### Repository Pattern

将数据访问封装在一致的接口后：
- 定义标准操作：findAll, findById, create, update, delete
- 具体实现处理存储细节（database, API, file 等）
- 业务逻辑依赖 abstract interface 而非存储机制
- 便于更换数据源并简化使用 mock 的测试

### API Response 格式

对所有 API 响应使用一致的封装：
- 添加 success/status 指示器
- 添加 data payload（出错时可为 null）
- 添加错误消息字段（成功时可为 null）
- 为分页响应添加 metadata（total, page, limit）
