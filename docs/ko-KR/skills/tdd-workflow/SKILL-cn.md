---
name: tdd-workflow
description: 在编写新功能、修复bug或重构代码时使用此技能。执行包含单元、集成、E2E测试的80%以上覆盖率的测试驱动开发。
origin: ECC
---

# 测试驱动开发工作流

此技能确保所有代码开发遵循TDD原则，并具备全面的测试覆盖率。

## 激活时机

- 编写新功能或特性时
- 修复bug或问题时
- 重构现有代码时
- 添加API端点时
- 创建新组件时

## 核心原则

### 1. 测试先于代码
始终先编写测试，然后实现使测试通过的代码。

### 2. 覆盖率要求
- 最低80%覆盖率（单元 + 集成 + E2E）
- 覆盖所有边界情况
- 错误场景测试
- 边界条件验证

### 3. 测试类型

#### 单元测试
- 个别函数和工具
- 组件逻辑
- 纯函数
- 辅助函数和工具

#### 集成测试
- API端点
- 数据库操作
- 服务交互
- 外部API调用

#### E2E测试 (Playwright)
- 核心用户流程
- 完整工作流
- 浏览器自动化
- UI交互

## TDD工作流步骤

### 步骤1: 编写用户旅程
```
As a [role], I want to [action], so that [benefit]

Example:
As a user, I want to search for markets semantically,
so that I can find relevant markets even without exact keywords.
```

### 步骤2: 创建测试用例
为每个用户旅程编写全面的测试用例：

```typescript
describe('Semantic Search', () => {
  it('returns relevant markets for query', async () => {
    // Test implementation
  })

  it('handles empty query gracefully', async () => {
    // Test edge case
  })

  it('falls back to substring search when Redis unavailable', async () => {
    // Test fallback behavior
  })

  it('sorts results by similarity score', async () => {
    // Test sorting logic
  })
})
```

### 步骤3: 运行测试（应该失败）
```bash
npm test
# Tests should fail - we haven't implemented yet
```

### 步骤4: 代码实现
编写使测试通过的最小代码：

```typescript
// Implementation guided by tests
export async function searchMarkets(query: string) {
  // Implementation here
}
```

### 步骤5: 重新运行测试
```bash
npm test
# Tests should now pass
```

### 步骤6: 重构
在保持测试通过的同时提高代码质量：
- 消除重复
- 改进命名
- 性能优化
- 提高可读性

### 步骤7: 检查覆盖率
```bash
npm run test:coverage
# Verify 80%+ coverage achieved
```

## 测试模式

### 单元测试模式 (Jest/Vitest)
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

### API集成测试模式
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
    // Mock database failure
    const request = new NextRequest('http://localhost/api/markets')
    // Test error handling
  })
})
```

### E2E测试模式 (Playwright)
```typescript
import { test, expect } from '@playwright/test'

test('user can search and filter markets', async ({ page }) => {
  // Navigate to markets page
  await page.goto('/')
  await page.click('a[href="/markets"]')

  // Verify page loaded
  await expect(page.locator('h1')).toContainText('Markets')

  // Search for markets
  await page.fill('input[placeholder="Search markets"]', 'election')

  // Wait for stable search results instead of sleeping
  const results = page.locator('[data-testid="market-card"]')
  await expect(results.first()).toBeVisible({ timeout: 5000 })
  await expect(results).toHaveCount(5, { timeout: 5000 })

  // Verify results contain search term
  const firstResult = results.first()
  await expect(firstResult).toContainText('election', { ignoreCase: true })

  // Filter by status
  await page.click('button:has-text("Active")')

  // Verify filtered results
  await expect(results).toHaveCount(3)
})

test('user can create a new market', async ({ page }) => {
  // Login first
  await page.goto('/creator-dashboard')

  // Fill market creation form
  await page.fill('input[name="name"]', 'Test Market')
  await page.fill('textarea[name="description"]', 'Test description')
  await page.fill('input[name="endDate"]', '2025-12-31')

  // Submit form
  await page.click('button[type="submit"]')

  // Verify success message
  await expect(page.locator('text=Market created successfully')).toBeVisible()

  // Verify redirect to market page
  await expect(page).toHaveURL(/\/markets\/test-market/)
})
```

## 测试文件结构

```
src/
├── components/
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.test.tsx          # Unit tests
│   │   └── Button.stories.tsx       # Storybook
│   └── MarketCard/
│       ├── MarketCard.tsx
│       └── MarketCard.test.tsx
├── app/
│   └── api/
│       └── markets/
│           ├── route.ts
│           └── route.test.ts         # Integration tests
└── e2e/
    ├── markets.spec.ts               # E2E tests
    ├── trading.spec.ts
    └── auth.spec.ts
```

## 外部服务Mocking

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
    new Array(1536).fill(0.1) // Mock 1536-dim embedding
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
    "coverageThreshold": {
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

## 常见测试错误

### 错误示例: 测试实现细节
```typescript
// Don't test internal state
expect(component.state.count).toBe(5)
```

### 正确示例: 测试用户可见行为
```typescript
// Test what users see
expect(screen.getByText('Count: 5')).toBeInTheDocument()
```

### 错误示例: 脆弱的选择器
```typescript
// Breaks easily
await page.click('.css-class-xyz')
```

### 正确示例: 语义化选择器
```typescript
// Resilient to changes
await page.click('button:has-text("Submit")')
await page.click('[data-testid="submit-button"]')
```

### 错误示例: 没有测试隔离
```typescript
// Tests depend on each other
test('creates user', () => { /* ... */ })
test('updates same user', () => { /* depends on previous test */ })
```

### 正确示例: 独立测试
```typescript
// Each test sets up its own data
test('creates user', () => {
  const user = createTestUser()
  // Test logic
})

test('updates user', () => {
  const user = createTestUser()
  // Update logic
})
```

## 持续测试

### 开发期间Watch模式
```bash
npm test -- --watch
# Tests run automatically on file changes
```

### Pre-Commit Hook
```bash
# Runs before every commit
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

1. **先写测试** - 始终TDD
2. **每个测试一个Assert** - 聚焦单一行为
3. **描述性测试名称** - 说明测试什么
4. **Arrange-Act-Assert** - 清晰的测试结构
5. **Mock外部依赖** - 隔离单元测试
6. **测试边界情况** - null, undefined, 空值, 大值
7. **测试错误路径** - 不仅仅是正常路径
8. **保持测试快速** - 每个单元测试低于50ms
9. **测试后清理** - 无副作用
10. **审查覆盖率报告** - 识别遗漏部分

## 成功指标

- 达到80%以上的代码覆盖率
- 所有测试通过（绿色）
- 没有跳过或禁用的测试
- 快速测试执行（单元测试低于30秒）
- E2E测试覆盖核心用户流程
- 测试在生产发布前捕获bug

---

**记住**: 测试不是可选的。测试是实现自信重构、快速开发和生产稳定性的安全网。
