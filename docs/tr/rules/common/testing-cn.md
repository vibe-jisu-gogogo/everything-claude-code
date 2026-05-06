# 测试要求

## 最低测试覆盖率：80%

测试类型（全部必需）：
1. **Unit Tests** - 单个函数、utility 函数、component
2. **Integration Tests** - API endpoint、数据库操作
3. **E2E Tests** - 关键用户流程（根据框架语言选择）

## 测试驱动开发

强制工作流程：
1. 先写测试（RED）
2. 运行测试 - 应该失败
3. 写最小实现（GREEN）
4. 运行测试 - 应该成功
5. 重构（IMPROVE）
6. 验证覆盖率（80%+）

## 测试错误故障排除

1. 使用 **tdd-guide** agent
2. 检查测试隔离
3. 验证 mock 正确性
4. 修复实现而非测试（除非测试本身有误）

## Agent 支持

- **tdd-guide** - 主动用于新功能，强制执行测试优先
