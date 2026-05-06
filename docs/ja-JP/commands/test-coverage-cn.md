# 测试覆盖率

分析测试覆盖率并生成缺失的测试。

1. 运行带 coverage 的测试: npm test --coverage 或 pnpm test --coverage

2. 分析覆盖率报告 (coverage/coverage-summary.json)

3. 识别覆盖率低于 80% 阈值的文件

4. 针对每个覆盖率不足的文件:
   - 分析未测试的代码路径
   - 生成函数的单元测试
   - 生成 API 的集成测试
   - 生成关键流程的 E2E 测试

5. 验证新测试通过

6. 显示覆盖率指标的前后对比

7. 确保整个项目覆盖率达到 80% 以上

重点项目:
- happy path scenario
- error handling
- edge case (null, undefined, empty)
- boundary condition
