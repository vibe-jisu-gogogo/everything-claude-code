---
description: 分析 coverage，识别差距，生成缺失的测试以达到目标阈值。
---

# 测试覆盖率

分析 test coverage，识别差距，生成缺失的测试以达到 80%+ 的覆盖率。

## 步骤 1：检测测试框架

| 指示器 | Coverage 命令 |
|-----------|-----------------|
| `jest.config.*` 或 `package.json` 中的 jest | `npx jest --coverage --coverageReporters=json-summary` |
| `vitest.config.*` | `npx vitest run --coverage` |
| `pytest.ini` / `pyproject.toml` 中的 pytest | `pytest --cov=src --cov-report=json` |
| `Cargo.toml` | `cargo llvm-cov --json` |
| `pom.xml` 带 JaCoCo | `mvn test jacoco:report` |
| `go.mod` | `go test -coverprofile=coverage.out ./...` |

## 步骤 2：分析覆盖率报告

1. 运行 coverage 命令
2. 解析输出（JSON 摘要或终端输出）
3. 列出 **覆盖率低于 80%** 的文件，按最差优先排序
4. 对每个覆盖率不足的文件，识别：
   - 未测试的函数或方法
   - 缺失的 branch coverage（if/else、switch、错误路径）
   - 扩大分母的死代码

## 步骤 3：生成缺失的测试

对每个覆盖率不足的文件，按此优先级生成测试：

1. **Happy path** — 有效输入的核心功能
2. **Error handling** — 无效输入、缺失数据、网络故障
3. **Edge cases** — 空数组、null/undefined、边界值（0、-1、MAX_INT）
4. **Branch coverage** — 每个 if/else、switch case、三元表达式

### 测试生成规则

- 将测试放在源码旁边：`foo.ts` → `foo.test.ts`（或项目约定）
- 使用项目现有的测试模式（导入风格、断言库、mocking 方法）
- Mock 外部依赖（database、APIs、文件系统）
- 每个测试应该独立 — 测试之间没有共享可变状态
- 为测试命名要有描述性：`test_create_user_with_duplicate_email_returns_409`

## 步骤 4：验证

1. 运行完整测试套件 — 所有测试必须通过
2. 重新运行 coverage — 验证改进
3. 如果仍低于 80%，对剩余差距重复步骤 3

## 步骤 5：报告

显示前后对比：

```
覆盖率报告
──────────────────────────────
文件                   之前  之后
src/services/auth.ts   45%     88%
src/utils/validation.ts 32%    82%
──────────────────────────────
总体:                   67%     84%  通过 ✓
```

## 重点关注领域

- 具有复杂分支的函数（高圈复杂度）
- 错误处理程序和 catch 块
- 代码库中跨模块使用的工具函数
- API endpoint 处理程序（request → response 流）
- 边缘情况：null、undefined、空字符串、空数组、零、负数
