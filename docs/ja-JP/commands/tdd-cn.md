---
description: 强制实施测试驱动开发工作流。首先搭建接口脚手架，生成测试，然后实现使测试通过的最小代码。保证80%以上的覆盖率。
---

# TDD命令

此命令调用**tdd-guide** agent，强制实施测试驱动开发方法。

## 此命令的功能

1. **搭建接口脚手架** - 首先定义类型/接口
2. **先生成测试** - 编写会失败的测试(RED)
3. **实现最小代码** - 编写仅能使测试通过的代码(GREEN)
4. **重构** - 保持测试通过的同时改进代码(REFACTOR)
5. **验证覆盖率** - 保证80%以上的测试覆盖率

## 使用时机

在以下情况下使用`/tdd`:
- 实现新功能时
- 添加新函数/组件时
- 修复bug时(首先编写重现bug的测试)
- 重构现有代码时
- 构建重要业务逻辑时

## 工作方式

tdd-guide agent执行以下操作:

1. **定义接口** - 确定输入/输出类型
2. **编写会失败的测试** (因为代码还不存在)
3. **运行测试**，确认因正确的原因失败
4. **编写最小实现** 使测试通过
5. **运行测试**，确认通过
6. **重构** - 保持测试绿色的同时改进代码
7. **确认覆盖率** - 如果低于80%则添加测试

## TDD周期

```
RED → GREEN → REFACTOR → REPEAT

RED:      编写会失败的测试
GREEN:    编写使测试通过的最小代码
REFACTOR: 改进代码，保持测试通过状态
REPEAT:   进入下一个功能/场景
```

## 使用示例

```
User: /tdd 需要一个计算市场流动性分数的函数

Agent (tdd-guide):
# TDD会话: 市场流动性分数计算器

## 步骤1: 定义接口(SCAFFOLD)

```typescript
// lib/liquidity.ts
export interface MarketData {
  totalVolume: number
  bidAskSpread: number
  activeTraders: number
  lastTradeTime: Date
}

export function calculateLiquidityScore(market: MarketData): number {
  // TODO: Implementation
  throw new Error('Not implemented')
}
```

## 步骤2: 编写会失败的测试(RED)

```typescript
// lib/liquidity.test.ts
import { calculateLiquidityScore } from './liquidity'

describe('calculateLiquidityScore', () => {
  it('should return high score for liquid market', () => {
    const market = {
      totalVolume: 100000,
      bidAskSpread: 0.01,
      activeTraders: 500,
      lastTradeTime: new Date()
    }

    const score = calculateLiquidityScore(market)

    expect(score).toBeGreaterThan(80)
    expect(score).toBeLessThanOrEqual(100)
  })

  it('should return low score for illiquid market', () => {
    const market = {
      totalVolume: 100,
      bidAskSpread: 0.5,
      activeTraders: 2,
      lastTradeTime: new Date(Date.now() - 86400000) // 1 day ago
    }

    const score = calculateLiquidityScore(market)

    expect(score).toBeLessThan(30)
    expect(score).toBeGreaterThanOrEqual(0)
  })

  it('should handle edge case: zero volume', () => {
    const market = {
      totalVolume: 0,
      bidAskSpread: 0,
      activeTraders: 0,
      lastTradeTime: new Date()
    }

    const score = calculateLiquidityScore(market)

    expect(score).toBe(0)
  })
})
```

## 步骤3: 运行测试 - 确认失败

```bash
npm test lib/liquidity.test.ts

FAIL lib/liquidity.test.ts
  ✕ should return high score for liquid market (2 ms)
    Error: Not implemented

1 test failed, 0 passed
```

PASS: 测试按预期失败。准备好进行实现。

## 步骤4: 实现最小代码(GREEN)

```typescript
// lib/liquidity.ts
export function calculateLiquidityScore(market: MarketData): number {
  // Handle zero volume edge case
  if (market.totalVolume === 0) {
    return 0
  }

  // Calculate component scores (0-100 scale)
  const volumeScore = Math.min(market.totalVolume / 1000, 100)
  const spreadScore = Math.max(100 - (market.bidAskSpread * 1000), 0)
  const traderScore = Math.min(market.activeTraders / 10, 100)

  // Recent activity bonus
  const hoursSinceLastTrade = (Date.now() - market.lastTradeTime.getTime()) / (1000 * 60 * 60)
  const recencyScore = Math.max(100 - (hoursSinceLastTrade * 10), 0)

  // Weighted average
  const score = (
    volumeScore * 0.4 +
    spreadScore * 0.3 +
    traderScore * 0.2 +
    recencyScore * 0.1
  )

  return Math.min(Math.max(score, 0), 100) // Clamp to 0-100
}
```

## 步骤5: 运行测试 - 确认通过

```bash
npm test lib/liquidity.test.ts

PASS lib/liquidity.test.ts
  ✓ should return high score for liquid market (3 ms)
  ✓ should return low score for illiquid market (2 ms)
  ✓ should handle edge case: zero volume (1 ms)

3 tests passed
```

PASS: 所有测试已通过!

## 步骤6: 重构(IMPROVE)

```typescript
// lib/liquidity.ts - 通过改进常量和可读性进行重构
const WEIGHTS = {
  VOLUME: 0.4,
  SPREAD: 0.3,
  TRADERS: 0.2,
  RECENCY: 0.1,
} as const

const SCALE_FACTORS = {
  VOLUME: 1000,
  SPREAD: 1000,
  TRADERS: 10,
  RECENCY_PENALTY: 10,
} as const

function clamp(value: number, min: number, max: number): number {
  return Math.min(Math.max(value, min), max)
}

export function calculateLiquidityScore(market: MarketData): number {
  if (market.totalVolume === 0) return 0

  const volumeScore = Math.min(market.totalVolume / SCALE_FACTORS.VOLUME, 100)
  const spreadScore = clamp(100 - (market.bidAskSpread * SCALE_FACTORS.SPREAD), 0, 100)
  const traderScore = Math.min(market.activeTraders / SCALE_FACTORS.TRADERS, 100)

  const hoursSinceLastTrade = (Date.now() - market.lastTradeTime.getTime()) / (1000 * 60 * 60)
  const recencyScore = clamp(100 - (hoursSinceLastTrade * SCALE_FACTORS.RECENCY_PENALTY), 0, 100)

  const weightedScore =
    volumeScore * WEIGHTS.VOLUME +
    spreadScore * WEIGHTS.SPREAD +
    traderScore * WEIGHTS.TRADERS +
    recencyScore * WEIGHTS.RECENCY

  return clamp(weightedScore, 0, 100)
}
```

## 步骤7: 确认测试仍然通过

```bash
npm test lib/liquidity.test.ts

PASS lib/liquidity.test.ts
  ✓ should return high score for liquid market (3 ms)
  ✓ should return low score for illiquid market (2 ms)
  ✓ should handle edge case: zero volume (1 ms)

3 tests passed
```

PASS: 重构完成，测试仍然通过!

## 步骤8: 确认覆盖率

```bash
npm test -- --coverage lib/liquidity.test.ts

File           | % Stmts | % Branch | % Funcs | % Lines
---------------|---------|----------|---------|--------
liquidity.ts   |   100   |   100    |   100   |   100

Coverage: 100% PASS: (Target: 80%)
```

PASS: TDD会话完成!
```

## TDD最佳实践

**应该做:**
- PASS: 实现前先编写测试
- PASS: 运行测试，确认在实现前失败
- PASS: 编写使测试通过的最小代码
- PASS: 仅在测试变绿后才进行重构
- PASS: 添加边界情况和错误场景
- PASS: 目标80%以上覆盖率(重要代码100%)

**不应该做:**
- FAIL: 在测试前编写实现
- FAIL: 跳过每次变更后的测试运行
- FAIL: 一次编写太多代码
- FAIL: 忽略会失败的测试
- FAIL: 测试实现细节(测试行为)
- FAIL: 模拟所有内容(优先集成测试)

## 应包含的测试类型

**单元测试**(函数级别):
- 快乐路径场景
- 边界情况(空、null、最大值)
- 错误条件
- 边界值

**集成测试**(组件级别):
- API端点
- 数据库操作
- 外部服务调用
- 使用hooks的React组件

**E2E测试**(使用`/e2e`命令):
- 重要用户流程
- 多步骤流程
- 全栈集成

## 覆盖率要求

- **所有代码80%以上**
- **以下代码必须100%**:
  - 财务计算
  - 认证逻辑
  - 安全关键代码
  - 核心业务逻辑

## 重要事项

**必须**: 测试必须在实现之前编写。TDD周期是:

1. **RED** - 编写会失败的测试
2. **GREEN** - 编写使测试通过的实现
3. **REFACTOR** - 改进代码

不能跳过RED阶段。不能在测试前编写代码。

## 与其他命令的集成

- 首先使用`/plan`了解要构建什么
- 使用`/tdd`进行带测试的实现
- 发生构建错误时使用`/build-and-fix`
- 使用`/code-review`审查实现
- 使用`/test-coverage`验证覆盖率

## 相关Agent

此命令调用位于以下位置的`tdd-guide` agent:
`~/.claude/agents/tdd-guide.md`

还可以参考位于以下位置的`tdd-workflow` skill:
`~/.claude/skills/tdd-workflow/`
