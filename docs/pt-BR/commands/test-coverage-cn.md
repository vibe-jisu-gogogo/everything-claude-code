# 测试覆盖率

分析测试覆盖率，识别差距并生成缺失的测试以达到80%以上的覆盖率。

## 步骤 1：检测测试框架

| Indicator | Coverage Command |
|-----------|-----------------|
| `jest.config.*` or `package.json` jest | `npx jest --coverage --coverageReporters=json-summary` |
| `vitest.config.*` | `npx vitest run --coverage` |
| `pytest.ini` / `pyproject.toml` pytest | `pytest --cov=src --cov-report=json` |
| `Cargo.toml` | `cargo llvm-cov --json` |
| `pom.xml` with JaCoCo | `mvn test jacoco:report` |
| `go.mod` | `go test -coverprofile=coverage.out ./...` |

## 步骤 2：分析覆盖率报告

1. 运行覆盖率命令
2. 解析输出（JSON摘要或终端输出）
3. 列出**覆盖率低于80%**的文件，从最差到最佳排序
4. 对于每个低于目标的文件，识别：
   - 未测试的函数或方法
   - 缺失的分支覆盖率（if/else、switch、错误路径）
   - 增大分母的死代码

## 步骤 3：生成缺失的测试

对于每个低于目标的文件，按以下优先级生成测试：

1. **Happy path** — 使用有效输入的主要功能
2. **错误处理** — 无效输入、缺失数据、网络故障
3. **边界情况** — 空数组、null/undefined、边界值（0、-1、MAX_INT）
4. **分支覆盖率** — 每个if/else、switch case、三元运算符

### 测试生成规则

- 将测试放在源代码旁边：`foo.ts` → `foo.test.ts`（或项目约定）
- 使用项目现有的测试模式（导入风格、断言库、mocking方法）
- Mock外部依赖（数据库、APIs、文件系统）
- 每个测试应独立 — 测试之间无共享可变状态
- 以描述性方式命名测试：`test_create_user_with_duplicate_email_returns_409`

## 步骤 4：验证

1. 运行完整的测试套件 — 所有测试应通过
2. 再次运行覆盖率 — 确认改进
3. 如果仍低于80%，对剩余差距重复步骤3

## 步骤 5：报告

显示前后比较：

```
Coverage Report
──────────────────────────────
File                   Before  After
src/services/auth.ts   45%     88%
src/utils/validation.ts 32%    82%
──────────────────────────────
Overall:               67%     84%  PASS:
```

## 重点关注领域

- 具有复杂分支的函数（高圈复杂度）
- 错误处理程序和catch块
- 在整个代码库中使用的工具函数
- API端点处理程序（请求→响应流）
- 边界情况：null、undefined、空字符串、空数组、零、负数
