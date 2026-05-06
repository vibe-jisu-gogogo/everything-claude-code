---
description: 使用 Playwright 生成并执行端到端测试。创建测试流程、运行测试、捕获截图/视频/追踪，并上传产物。
---

# E2E 命令

此命令调用 **e2e-runner** agent，使用 Playwright 生成、维护和执行端到端测试。

## 此命令的功能

1. **测试流程生成** - 创建用户流程的 Playwright 测试
2. **E2E 测试执行** - 在多个浏览器间运行测试
3. **产物捕获** - 失败时的截图、视频、追踪
4. **结果上传** - HTML 报告和 JUnit XML
5. **不稳定测试识别** - 分离不稳定的测试

## 何时使用

在以下情况下使用 `/e2e`：

* 测试重要用户流程（登录、交易、支付）
* 验证多步骤流程能端到端正常工作
* 测试 UI 交互和导航
* 验证前端与后端间的集成
* 为生产环境部署做准备

## 工作方式

e2e-runner agent：

1. **分析用户流程**以确定测试场景
2. **使用页面对象模型模式生成 Playwright 测试**
3. **在多个浏览器间（Chrome、Firefox、Safari）执行测试**
4. **捕获失败**（包括截图、视频、追踪）
5. **生成包含结果和产物的报告**
6. **识别不稳定测试**并建议修复

## 使用示例

````
User: /e2e 测试市场搜索和查看流程

Agent (e2e-runner):
# E2E测试生成: 市场搜索和查看流程

## 确定的测试场景

**用户流程:** 市场搜索 → 结果显示 → 点击市场 → 详情显示

**测试覆盖:**
1. 导航到市场页面
2. 执行语义搜索
3. 验证搜索结果
4. 点击第一个结果
5. 验证市场详情页面加载
6. 验证图表渲染

## 生成的测试代码

```typescript
// tests/e2e/markets/search-and-view.spec.ts
import { test, expect } from '@playwright/test'
import { MarketsPage } from '../../pages/MarketsPage'
import { MarketDetailsPage } from '../../pages/MarketDetailsPage'

test.describe('Market Search and View Flow', () => {
  test('user can search markets and view details', async ({ page }) => {
    // 1. Navigate to markets page
    const marketsPage = new MarketsPage(page)
    await marketsPage.goto()

    // Verify page loaded
    await expect(page).toHaveTitle(/Markets/)
    await expect(page.locator('h1')).toContainText('Markets')

    // 2. Perform semantic search
    await marketsPage.searchMarkets('election')

    // Wait for API response
    await page.waitForResponse(resp =>
      resp.url().includes('/api/markets/search') && resp.status() === 200
    )

    // 3. Verify search results
    const marketCards = marketsPage.marketCards
    await expect(marketCards.first()).toBeVisible()
    const resultCount = await marketCards.count()
    expect(resultCount).toBeGreaterThan(0)

    // Take screenshot of search results
    await page.screenshot({ path: 'artifacts/search-results.png' })

    // 4. Click on first result
    const firstMarketTitle = await marketCards.first().textContent()
    await marketCards.first().click()

    // 5. Verify market details page loads
    await expect(page).toHaveURL(/\/markets\/[a-z0-9-]+/)

    const detailsPage = new MarketDetailsPage(page)
    await expect(detailsPage.marketName).toBeVisible()
    await expect(detailsPage.marketDescription).toBeVisible()

    // 6. Verify chart renders
    await expect(detailsPage.priceChart).toBeVisible()

    // Verify market name matches
    const detailsTitle = await detailsPage.marketName.textContent()
    expect(detailsTitle?.toLowerCase()).toContain(
      firstMarketTitle?.toLowerCase().substring(0, 20) || ''
    )

    // Take screenshot of market details
    await page.screenshot({ path: 'artifacts/market-details.png' })
  })

  test('search with no results shows empty state', async ({ page }) => {
    const marketsPage = new MarketsPage(page)
    await marketsPage.goto()

    // Search for non-existent market
    await marketsPage.searchMarkets('xyznonexistentmarket123456')

    // Verify empty state
    await expect(page.locator('[data-testid="no-results"]')).toBeVisible()
    await expect(page.locator('[data-testid="no-results"]')).toContainText(
      /no.*results|no.*markets/i
    )

    const marketCount = await marketsPage.marketCards.count()
    expect(marketCount).toBe(0)
  })

  test('can clear search and see all markets again', async ({ page }) => {
    const marketsPage = new MarketsPage(page)
    await marketsPage.goto()

    // Initial market count
    const initialCount = await marketsPage.marketCards.count()

    // Perform search
    await marketsPage.searchMarkets('trump')
    await page.waitForLoadState('networkidle')

    // Verify filtered results
    const filteredCount = await marketsPage.marketCards.count()
    expect(filteredCount).toBeLessThan(initialCount)

    // Clear search
    await marketsPage.searchInput.clear()
    await page.waitForLoadState('networkidle')

    // Verify all markets shown again
    const finalCount = await marketsPage.marketCards.count()
    expect(finalCount).toBe(initialCount)
  })
})
````

## 测试执行

```bash
# 执行生成的测试
npx playwright test tests/e2e/markets/search-and-view.spec.ts

Running 3 tests using 3 workers

  ✓  [chromium] › search-and-view.spec.ts:5:3 › user can search markets and view details (4.2s)
  ✓  [chromium] › search-and-view.spec.ts:52:3 › search with no results shows empty state (1.8s)
  ✓  [chromium] › search-and-view.spec.ts:67:3 › can clear search and see all markets again (2.9s)

  3 passed (9.1s)

Artifacts generated:
- artifacts/search-results.png
- artifacts/market-details.png
- playwright-report/index.html
```

## 测试报告

```
╔══════════════════════════════════════════════════════════════╗
║                    E2E测试结果                                ║
╠══════════════════════════════════════════════════════════════╣
║ 状态: PASS: 全部测试通过                                      ║
║ 总计:       3测试                                             ║
║ 通过:       3 (100%)                                         ║
║ 失败:       0                                                ║
║ 不稳定:     0                                                ║
║ 所需时间:   9.1s                                             ║
╚══════════════════════════════════════════════════════════════╝

产物:
 截图: 2文件
 视频: 0文件 (仅失败时)
 追踪: 0文件 (仅失败时)
 HTML报告: playwright-report/index.html

报告显示: npx playwright show-report
```

PASS: E2E 测试套件已准备好 CI/CD 集成！

````

## 测试产物

测试执行时，将捕获以下产物：

**全测试共通:**
- 包含时间线和结果的HTML报告
- CI集成用的JUnit XML

**仅失败时:**
- 失败状态的截图
- 测试的视频录制
- 调试用追踪文件 (分步回放)
- 网络日志
- 控制台日志

## 产物确认

```bash
# 在浏览器中显示HTML报告
npx playwright show-report

# 显示特定的追踪文件
npx playwright show-trace artifacts/trace-abc123.zip

# 截图保存在artifacts/目录
open artifacts/search-results.png
````

## 不稳定测试检测

测试间歇性失败时：

```
WARNING:  FLAKY TEST DETECTED: tests/e2e/markets/trade.spec.ts

测试10次中7次通过 (合格率70%)

常见失败:
"Timeout waiting for element '[data-testid="confirm-btn"]'"

推荐修正:
1. 添加显式等待: await page.waitForSelector('[data-testid="confirm-btn"]')
2. 增加超时: { timeout: 10000 }
3. 确认组件的竞争状态
4. 确认元素是否被动画隐藏

隔离推荐: 修正前标记为test.fixme()
```

## 浏览器设置

默认情况下，测试在多个浏览器中执行：

* PASS: Chromium（桌面 Chrome）
* PASS: Firefox（桌面）
* PASS: WebKit（桌面 Safari）
* PASS: Mobile Chrome（可选）

在 `playwright.config.ts` 中设置以调整浏览器。

## CI/CD 集成

添加到 CI 流水线：

```yaml
# .github/workflows/e2e.yml
- name: Install Playwright
  run: npx playwright install --with-deps

- name: Run E2E tests
  run: npx playwright test

- name: Upload artifacts
  if: always()
  uses: actions/upload-artifact@v3
  with:
    name: playwright-report
    path: playwright-report/
```

## PMX 特有的重要流程

对于 PMX，优先以下 E2E 测试：

**重大（必须始终成功）：**

1. 用户可连接钱包
2. 用户可浏览市场
3. 用户可搜索市场（语义搜索）
4. 用户可显示市场详情
5. 用户可下达交易订单（使用测试资金）
6. 市场正确结算
7. 用户可提取资金

**重要：**

1. 市场创建流程
2. 用户资料更新
3. 实时价格更新
4. 图表渲染
5. 市场过滤和排序
6. 移动响应式布局

## 最佳实践

**应该做：**

* PASS: 为提高可维护性使用页面对象模型
* PASS: 作为选择器使用 data-testid 属性
* PASS: 不是任意超时，而是等待 API 响应
* PASS: 重要用户流程的端到端测试
* PASS: 合并到 main 前执行测试
* PASS: 测试失败时审查产物

**不应该做：**

* FAIL: 使用不稳定的选择器（CSS 类可能变化）
* FAIL: 测试实现的细节
* FAIL: 对生产环境执行测试
* FAIL: 忽视不稳定的测试
* FAIL: 失败时跳过产物审查
* FAIL: 在 E2E 测试中测试所有边缘情况（使用单元测试）

## 重要注意事项

**对 PMX 来说重大：**

* 涉及实际资金的 E2E 测试**仅应在测试网/预发布环境执行**
* 不要对生产环境执行交易测试
* 对金融测试设置 `test.skip(process.env.NODE_ENV === 'production')`
* 仅使用持有少量测试资金的测试钱包

## 与其他命令的集成

* 使用 `/plan` 确定要测试的重要流程
* 使用 `/tdd` 进行单元测试（更快、更细粒度）
* 使用 `/e2e` 进行集成和用户流程测试
* 使用 `/code-review` 验证测试质量

## 相关 agent

此命令调用 `~/.claude/agents/e2e-runner.md` 的 `e2e-runner` agent。

## 快速命令

```bash
# 执行全部E2E测试
npx playwright test

# 执行特定的测试文件
npx playwright test tests/e2e/markets/search.spec.ts

# 有头模式执行 (显示浏览器)
npx playwright test --headed

# 调试测试
npx playwright test --debug

# 生成测试代码
npx playwright codegen http://localhost:3000

# 显示报告
npx playwright show-report
```
