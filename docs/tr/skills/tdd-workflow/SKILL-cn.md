---
name: tdd-workflow
description: 编写新功能、修复 bug 或重构代码时使用此 skill。强制进行包含 unit、integration 和 E2E 测试的 80%+ 覆盖率的测试驱动开发。
origin: ECC
---

# 测试驱动开发工作流

此 skill 确保所有代码开发都遵循 TDD 原则并具有全面的测试覆盖。

## 何时激活

- 编写新功能或特性时
- 修复 bug 或问题时
- 重构现有代码时
- 添加 API endpoint 时
- 创建新组件时

## 核心原则

### 1. 测试优先于代码
始终先编写测试，然后编写代码使测试通过。

### 2. 覆盖率要求
- 最低 80% 覆盖率（unit + integration + E2E）
- 覆盖所有边缘情况
- 测试错误场景
- 验证边界条件

### 3. 测试类型

#### Unit 测试
- 独立函数和辅助工具
- 组件逻辑
- Pure 函数
- 辅助工具和 utilities

#### Integration 测试
- API endpoint
- 数据库操作
- Service 交互
- 外部 API 调用

#### E2E 测试 (Playwright)
- 关键用户流程
- 完整工作流程
- 浏览器自动化
- UI 交互

## TDD 工作流程步骤

### 步骤 1：编写用户故事
```
作为 [角色]，我想要 [操作]，以便获得 [价值]

示例：
作为用户，我想要对市场进行语义搜索，
这样即使没有精确关键词，我也能找到相关市场。
```

### 步骤 2：创建测试场景
为每个用户故事创建全面的测试场景：

```typescript
describe('语义搜索', () => {
  it('返回与查询相关的市场', async () => {
    // 测试实现
  })

  it('优雅地处理空查询', async () => {
    // 测试边缘情况
  })

  it('当 Redis 不可用时回退到子字符串搜索', async () => {
    // 测试回退行为
  })

  it('按相似度分数对结果排序', async () => {
    // 测试排序逻辑
  })
})
```

### 步骤 3：运行测试（应该失败）
```bash
npm test
# 测试应该失败 - 我们还没有实现
```

### 步骤 4：实现代码
编写使测试通过的最小代码：

```typescript
// 由测试驱动的实现
export async function searchMarkets(query: string) {
  // 实现内容
}
```

### 步骤 5：重新运行测试
```bash
npm test
# 测试现在应该通过
```

### 步骤 6：重构
在保持测试通过的同时提高代码质量：
- 消除重复
- 改进命名
- 优化性能
- 提高可读性

### 步骤 7：验证覆盖率
```bash
npm run test:coverage
# 验证达到 80%+ 覆盖率
```

## 测试模式

### Unit 测试模板 (Jest/Vitest)
```typescript
import { render, screen, fireEvent } from '@testing-library/react'
import { Button } from './Button'

describe('Button 组件', () => {
  it('使用正确的文本渲染', () => {
    render(<Button>点击</Button>)
    expect(screen.getByText('点击')).toBeInTheDocument()
  })

  it('点击时调用 onClick', () => {
    const handleClick = jest.fn()
    render(<Button onClick={handleClick}>点击</Button>)

    fireEvent.click(screen.getByRole('button'))

    expect(handleClick).toHaveBeenCalledTimes(1)
  })

  it('当 disabled prop 为 true 时禁用', () => {
    render(<Button disabled>点击</Button>)
    expect(screen.getByRole('button')).toBeDisabled()
  })
})
```

### API Integration 测试模板
```typescript
import { NextRequest } from 'next/server'
import { GET } from './route'

describe('GET /api/markets', () => {
  it('成功返回市场列表', async () => {
    const request = new NextRequest('http://localhost/api/markets')
    const response = await GET(request)
    const data = await response.json()

    expect(response.status).toBe(200)
    expect(data.success).toBe(true)
    expect(Array.isArray(data.data)).toBe(true)
  })

  it('验证 query 参数', async () => {
    const request = new NextRequest('http://localhost/api/markets?limit=invalid')
    const response = await GET(request)

    expect(response.status).toBe(400)
  })

  it('优雅地处理数据库错误', async () => {
    // Mock 数据库失败
    const request = new NextRequest('http://localhost/api/markets')
    // 测试错误处理
  })
})
```

### E2E 测试模板 (Playwright)
```typescript
import { test, expect } from '@playwright/test'

test('用户可以搜索和筛选市场', async ({ page }) => {
  // 导航到市场页面
  await page.goto('/')
  await page.click('a[href="/markets"]')

  // 验证页面加载
  await expect(page.locator('h1')).toContainText('Markets')

  // 搜索市场
  await page.fill('input[placeholder="搜索市场"]', 'election')

  // 等待防抖和结果
  await page.waitForTimeout(600)

  // 验证显示搜索结果
  const results = page.locator('[data-testid="market-card"]')
  await expect(results).toHaveCount(5, { timeout: 5000 })

  // 验证结果包含搜索词
  const firstResult = results.first()
  await expect(firstResult).toContainText('election', { ignoreCase: true })

  // 按状态筛选
  await page.click('button:has-text("活跃")')

  // 验证筛选结果
  await expect(results).toHaveCount(3)
})

test('用户可以创建新市场', async ({ page }) => {
  // 先登录
  await page.goto('/creator-dashboard')

  // 填写创建市场表单
  await page.fill('input[name="name"]', 'Test Market')
  await page.fill('textarea[name="description"]', '测试描述')
  await page.fill('input[name="endDate"]', '2025-12-31')

  // 提交表单
  await page.click('button[type="submit"]')

  // 验证成功消息
  await expect(page.locator('text=市场创建成功')).toBeVisible()

  // 验证重定向到市场页面
  await expect(page).toHaveURL(/\/markets\/test-market/)
})
```

## 测试文件组织

```
src/
├── components/
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.test.tsx          # Unit 测试
│   │   └── Button.stories.tsx       # Storybook
│   └── MarketCard/
│       ├── MarketCard.tsx
│       └── MarketCard.test.tsx
├── app/
│   └── api/
│       └── markets/
│           ├── route.ts
│           └── route.test.ts         # Integration 测试
└── e2e/
    ├── markets.spec.ts               # E2E 测试
    ├── trading.spec.ts
    └── auth.spec.ts
```

## Mock 外部服务

### Supabase Mock
```typescript
jest.mock('@/lib/supabase', () => ({
  supabase: {
    from: jest.fn(() => ({
      select: jest.fn(() => ({
        eq: jest.fn(() => Promise.resolve({
          data: [{ id: 1, name: 'Test Market' }],
          error: null
        }))
      }))
    }))
  }
}))
```

### Redis Mock
```typescript
jest.mock('@/lib/redis', () => ({
  searchMarketsByVector: jest.fn(() => Promise.resolve([
    { slug: 'test-market', similarity_score: 0.95 }
  ])),
  checkRedisHealth: jest.fn(() => Promise.resolve({ connected: true }))
}))
```

### OpenAI Mock
```typescript
jest.mock('@/lib/openai', () => ({
  generateEmbedding: jest.fn(() => Promise.resolve(
    new Array(1536).fill(0.1) // Mock 1536 维 embedding
  ))
}))
```

## 测试覆盖率验证

### 运行覆盖率报告
```bash
npm run test:coverage
```

### 覆盖率阈值
```json
{
  "jest": {
    "coverageThresholds": {
      "global": {
        "branches": 80,
        "functions": 80,
        "lines": 80,
        "statements": 80
      }
    }
  }
}
```

## 常见测试陷阱要避免

### 错误：测试实现细节
```typescript
// 不要测试内部 state
expect(component.state.count).toBe(5)
```

### 正确：测试用户可见行为
```typescript
// 测试用户看到的内容
expect(screen.getByText('计数: 5')).toBeInTheDocument()
```

### 错误：脆弱的 Selector
```typescript
// 容易被破坏
await page.click('.css-class-xyz')
```

### 正确：语义 Selector
```typescript
// 对变更具有弹性
await page.click('button:has-text("提交")')
await page.click('[data-testid="submit-button"]')
```

### 错误：没有测试隔离
```typescript
// 测试相互依赖
test('创建用户', () => { /* ... */ })
test('更新同一用户', () => { /* 依赖前一个测试 */ })
```

### 正确：独立测试
```typescript
// 每个测试准备自己的数据
test('创建用户', () => {
  const user = createTestUser()
  // 测试逻辑
})

test('更新用户', () => {
  const user = createTestUser()
  // 更新逻辑
})
```

## 持续测试

### 开发期间的 Watch 模式
```bash
npm test -- --watch
# 文件变更时测试自动运行
```

### Pre-Commit Hook
```bash
# 每次提交前运行
npm test && npm run lint
```

### CI/CD 集成
```yaml
# GitHub Actions
- name: Run Tests
  run: npm test -- --coverage
- name: Upload Coverage
  uses: codecov/codecov-action@v3
```

## 最佳实践

1. **先写测试** - 始终使用 TDD
2. **每个测试一个 Assert** - 专注于单一行为
3. **描述性的测试名称** - 清楚说明正在测试什么
4. **Arrange-Act-Assert** - 清晰的测试结构
5. **Mock 外部依赖** - 隔离 Unit 测试
6. **测试边缘情况** - Null、undefined、空值、大值
7. **测试错误路径** - 不只是 happy path
8. **保持测试快速** - 每个 Unit 测试 < 50ms
9. **测试后清理** - 无副作用
10. **查看覆盖率报告** - 发现差距

## 成功指标

- 达到 80%+ 代码覆盖率
- 所有测试通过（绿色）
- 无跳过或禁用的测试
- 快速测试执行（Unit 测试 < 30s）
- E2E 测试覆盖关键用户流程
- 测试在 production 之前捕获 bug

---

**记住**：测试不是可选的。它们是允许安全重构、快速开发和 production 可靠性的安全网。
