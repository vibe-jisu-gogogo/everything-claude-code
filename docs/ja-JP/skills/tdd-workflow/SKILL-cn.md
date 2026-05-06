---
name: tdd-workflow
description: 创建新功能、修复bug、重构代码时使用此技能。强制测试驱动开发，包括unit test、integration test、E2E test，覆盖率达到80%以上。
---

# 测试驱动开发工作流

此技能确保所有代码开发遵循TDD原则，并具有全面的test coverage。

## 何时使用

- 创建新功能或特性
- 修复bug或问题
- 重构现有代码
- 添加API endpoint
- 创建新组件

## 核心原则

### 1. 测试优先于代码
总是先编写测试，然后实现让测试通过的代码。

### 2. 覆盖率要求
- 最低80%的coverage（unit + integration + E2E）
- 覆盖所有edge case
- 测试error scenario
- 验证boundary condition

### 3. 测试类型

#### Unit Test
- 单个函数和utility
- 组件logic
- pure function
- helper和utility

#### Integration Test
- API endpoint
- 数据库operation
- 服务间interaction
- 外部API call

#### E2E Test (Playwright)
- 关键user flow
- 完整workflow
- 浏览器automation
- UI interaction

## TDD工作流步骤

### 步骤1：编写user story
```
作为[角色]，我想要[行为]，以便[获得的利益]

例：
作为user，我想要semantic地搜索market，
这样即使没有准确的keyword，我也能找到相关的market。
```

### 步骤2：生成test case
为每个user story创建全面的test case：

```typescript
describe('Semantic Search', () => {
  it('returns relevant markets for query', async () => {
    // 测试实现
  })

  it('handles empty query gracefully', async () => {
    // edge case的测试
  })

  it('falls back to substring search when Redis unavailable', async () => {
    // fallback行为的测试
  })

  it('sorts results by similarity score', async () => {
    // sort logic的测试
  })
})
```

### 步骤3：运行测试（应该失败）
```bash
npm test
# 测试应该失败 - 还没有实现
```

### 步骤4：实现代码
编写让测试通过的最小代码：

```typescript
// 测试指导的实现
export async function searchMarkets(query: string) {
  // 实现在这里
}
```

### 步骤5：重新运行测试
```bash
npm test
# 测试现在应该成功
```

### 步骤6：refactor
保持测试通过的同时提高代码质量：
- 移除duplication
- 改进naming
- 优化performance
- 提高readability

### 步骤7：检查coverage
```bash
npm run test:coverage
# 确认达到80%以上的coverage
```

## 测试pattern

### Unit Test Pattern (Jest/Vitest)
```typescript
import { render, screen, fireEvent } from '@testing-library/react'
import { Button } from './Button'

describe('Button Component', () => {
  it('renders with correct text', () => {
    render(<Button>Click me</Button>)
    expect(screen.getByText('Click me')).toBeInTheDocument()
  })

  it('calls onClick when clicked', () => {
    const handleClick = jest.fn()
    render(<Button onClick={handleClick}>Click</Button>)

    fireEvent.click(screen.getByRole('button'))

    expect(handleClick).toHaveBeenCalledTimes(1)
  })

  it('is disabled when disabled prop is true', () => {
    render(<Button disabled>Click</Button>)
    expect(screen.getByRole('button')).toBeDisabled()
  })
})
```

### API Integration Test Pattern
```typescript
import { NextRequest } from 'next/server'
import { GET } from './route'

describe('GET /api/markets', () => {
  it('returns markets successfully', async () => {
    const request = new NextRequest('http://localhost/api/markets')
    const response = await GET(request)
    const data = await response.json()

    expect(response.status).toBe(200)
    expect(data.success).toBe(true)
    expect(Array.isArray(data.data)).toBe(true)
  })

  it('validates query parameters', async () => {
    const request = new NextRequest('http://localhost/api/markets?limit=invalid')
    const response = await GET(request)

    expect(response.status).toBe(400)
  })

  it('handles database errors gracefully', async () => {
    // 数据库故障的mock
    const request = new NextRequest('http://localhost/api/markets')
    // error handling的测试
  })
})
```

### E2E Test Pattern (Playwright)
```typescript
import { test, expect } from '@playwright/test'

test('user can search and filter markets', async ({ page }) => {
  // 导航到market页面
  await page.goto('/')
  await page.click('a[href="/markets"]')

  // 确认page已加载
  await expect(page.locator('h1')).toContainText('Markets')

  // 搜索market
  await page.fill('input[placeholder="Search markets"]', 'election')

  // 等待debounce和result
  await page.waitForTimeout(600)

  // 确认search result显示
  const results = page.locator('[data-testid="market-card"]')
  await expect(results).toHaveCount(5, { timeout: 5000 })

  // 确认result包含search term
  const firstResult = results.first()
  await expect(firstResult).toContainText('election', { ignoreCase: true })

  // 按status过滤
  await page.click('button:has-text("Active")')

  // 确认filtered result
  await expect(results).toHaveCount(3)
})

test('user can create a new market', async ({ page }) => {
  // 首先login
  await page.goto('/creator-dashboard')

  // 填写market create form
  await page.fill('input[name="name"]', 'Test Market')
  await page.fill('textarea[name="description"]', 'Test description')
  await page.fill('input[name="endDate"]', '2025-12-31')

  // submit form
  await page.click('button[type="submit"]')

  // 确认success message
  await expect(page.locator('text=Market created successfully')).toBeVisible()

  // 确认redirect到market page
  await expect(page).toHaveURL(/\/markets\/test-market/)
})
```

## 测试文件结构

```
src/
├── components/
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.test.tsx          # unit test
│   │   └── Button.stories.tsx       # Storybook
│   └── MarketCard/
│       ├── MarketCard.tsx
│       └── MarketCard.test.tsx
├── app/
│   └── api/
│       └── markets/
│           ├── route.ts
│           └── route.test.ts         # integration test
└── e2e/
    ├── markets.spec.ts               # E2E test
    ├── trading.spec.ts
    └── auth.spec.ts
```

## 外部服务的mock

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
    new Array(1536).fill(0.1) // 1536维embedding的mock
  ))
}))
```

## 测试coverage验证

### 运行coverage report
```bash
npm run test:coverage
```

### coverage threshold
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

## 避免常见的测试错误

### FAIL: 错误：测试实现detail
```typescript
// 不要测试内部state
expect(component.state.count).toBe(5)
```

### PASS: 正确：测试user可见的behavior
```typescript
// 测试user看到的内容
expect(screen.getByText('Count: 5')).toBeInTheDocument()
```

### FAIL: 错误：脆弱的selector
```typescript
// 容易被破坏
await page.click('.css-class-xyz')
```

### PASS: 正确：semantic selector
```typescript
// 对变化有弹性
await page.click('button:has-text("Submit")')
await page.click('[data-testid="submit-button"]')
```

### FAIL: 错误：测试没有isolation
```typescript
// 测试相互依赖
test('creates user', () => { /* ... */ })
test('updates same user', () => { /* 前一个测试的依赖 */ })
```

### PASS: 正确：独立的测试
```typescript
// 每个test设置自己的数据
test('creates user', () => {
  const user = createTestUser()
  // 测试logic
})

test('updates user', () => {
  const user = createTestUser()
  // update logic
})
```

## 持续测试

### 开发中的watch mode
```bash
npm test -- --watch
# 文件change时自动运行测试
```

### pre-commit hook
```bash
# 所有commit前运行
npm test && npm run lint
```

### CI/CD集成
```yaml
# GitHub Actions
- name: Run Tests
  run: npm test -- --coverage
- name: Upload Coverage
  uses: codecov/codecov-action@v3
```

## 最佳实践

1. **测试优先** - 始终使用TDD
2. **每个test一个assert** - 专注于单一behavior
3. **描述性的test name** - 说明测试什么
4. **Arrange-Act-Assert** - 清晰的test结构
5. **mock外部dependency** - 隔离unit test
6. **测试edge case** - null、undefined、空值、大值
7. **测试error path** - 不仅是happy path
8. **保持测试快速** - 每个unit test < 50ms
9. **测试后cleanup** - 没有side effect
10. **review coverage report** - 识别gap

## 成功指标

- 80%以上的code coverage
- 所有测试通过（green）
- 没有skip或disabled的测试
- 快速的test execution（unit test < 30秒）
- E2E test覆盖critical的user flow
- 测试在production前发现bug

---

**记住**：测试不是可选的。测试是让你能够自信地refactor、快速开发和可靠production的安全网。
