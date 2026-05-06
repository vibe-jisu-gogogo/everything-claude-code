---
description: 强制执行测试驱动开发工作流。首先搭建接口，然后生成测试，最后实现使测试通过的最少代码。保证 80% 以上的覆盖率。
---

# TDD 命令

此命令调用 **tdd-guide** agent 来强制执行测试驱动开发方法论。

## 此命令的功能

1. **接口搭建** - 首先定义类型/接口
2. **先创建测试** - 编写失败的测试 (RED)
3. **最少代码实现** - 只编写足够通过测试的代码 (GREEN)
4. **重构** - 在保持测试通过的同时改进代码 (REFACTOR)
5. **覆盖率检查** - 确保 80% 以上的测试覆盖率

## 使用时机

应使用 `/tdd` 的情况：
- 实现新功能
- 添加新函数/组件
- 修复 bug (首先编写重现 bug 的测试)
- 重构现有代码
- 实现核心业务逻辑

## 工作原理

tdd-guide agent 执行的操作：

1. 定义输入输出的**接口**
2. 编写**失败的测试** (因为代码还不存在)
3. **运行测试**并确认因正确原因失败
4. 编写**最少实现**以使测试通过
5. **运行测试**并确认通过
6. 在保持测试通过的同时**重构**代码
7. **检查覆盖率**，如果低于 80% 则添加测试

## TDD 循环

```
RED → GREEN → REFACTOR → REPEAT

RED:      编写失败的测试
GREEN:    编写使测试通过的最少代码
REFACTOR: 改进代码，保持测试继续通过
REPEAT:   下一个功能/场景
```

## 使用示例

````
User: /tdd 需要一个计算市场流动性分数的函数

Agent (tdd-guide):
# TDD 会话：市场流动性分数计算器

## 第1步：接口定义 (SCAFFOLD)

```typescript
// lib/liquidity.ts
export interface MarketData {
  totalVolume: number
  bidAskSpread: number
  activeTraders: number
  lastTradeTime: Date
}

export function calculateLiquidityScore(market: MarketData): number {
  // TODO: 实现
  throw new Error('Not implemented')
}
```

## 第2步：编写失败的测试 (RED)

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
      lastTradeTime: new Date(Date.now() - 86400000) // 1天前
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

## 第3步：运行测试 - 确认失败

```bash
npm test lib/liquidity.test.ts

FAIL lib/liquidity.test.ts
  ✕ should return high score for liquid market (2 ms)
    Error: Not implemented

1 test failed, 0 passed
```

PASS: 测试按预期失败。准备实现。

## 第4步：最少代码实现 (GREEN)

```typescript
// lib/liquidity.ts
export function calculateLiquidityScore(market: MarketData): number {
  // 处理交易量为0的边界情况
  if (market.totalVolume === 0) {
    return 0
  }

  // 计算各组成部分的分数 (0-100 范围)
  const volumeScore = Math.min(market.totalVolume / 1000, 100)
  const spreadScore = Math.max(100 - (market.bidAskSpread * 1000), 0)
  const traderScore = Math.min(market.activeTraders / 10, 100)

  // 近期活动奖励
  const hoursSinceLastTrade = (Date.now() - market.lastTradeTime.getTime()) / (1000 * 60 * 60)
  const recencyScore = Math.max(100 - (hoursSinceLastTrade * 10), 0)

  // 加权平均
  const score = (
    volumeScore * 0.4 +
    spreadScore * 0.3 +
    traderScore * 0.2 +
    recencyScore * 0.1
  )

  return Math.min(Math.max(score, 0), 100) // 限制在 0-100 范围内
}
```

## 第5步：运行测试 - 确认通过

```bash
npm test lib/liquidity.test.ts

PASS lib/liquidity.test.ts
  ✓ should return high score for liquid market (3 ms)
  ✓ should return low score for illiquid market (2 ms)
  ✓ should handle edge case: zero volume (1 ms)

3 tests passed
```

PASS: 所有测试通过！

## 第6步：重构 (IMPROVE)

```typescript
// lib/liquidity.ts - 为提高可读性重构常量
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

## 第7步：确认测试仍然通过

```bash
npm test lib/liquidity.test.ts

PASS lib/liquidity.test.ts
  ✓ should return high score for liquid market (3 ms)
  ✓ should return low score for illiquid market (2 ms)
  ✓ should handle edge case: zero volume (1 ms)

3 tests passed
```

PASS: 重构完成，测试仍然通过！

## 第8步：检查覆盖率

```bash
npm test -- --coverage lib/liquidity.test.ts

File           | % Stmts | % Branch | % Funcs | % Lines
---------------|---------|----------|---------|--------
liquidity.ts   |   100   |   100    |   100   |   100

Coverage: 100% PASS: (目标: 80%)
```

PASS: TDD 会话完成！
````

## TDD 最佳实践

**应该做的：**
- 在实现之前先编写测试
- 在实现之前运行测试确认会失败
- 编写使测试通过的最少代码
- 只有在测试通过后才进行重构
- 添加边界情况和错误场景
- 目标 80% 以上覆盖率 (核心代码 100%)

**不应该做的：**
- 在测试之前编写实现
- 跳过每次更改后的测试运行
- 一次编写太多代码
- 忽略失败的测试
- 测试实现细节 (测试行为)
- 模拟所有内容 (优先集成测试)

## 要包含的测试类型

**单元测试** (函数级别)：
- 正常路径场景
- 边界情况 (空值、null、最大值)
- 错误条件
- 边界值

**集成测试** (组件级别)：
- API 端点
- 数据库操作
- 外部服务调用
- 包含 hooks 的 React 组件

**E2E 测试** (使用 `/e2e` 命令)：
- 核心用户流程
- 多步骤过程
- 全栈集成

## 覆盖率要求

- **最少 80%** - 适用于所有代码
- **必须 100%** - 适用于以下内容：
  - 金融计算
  - 认证逻辑
  - 安全关键代码
  - 核心业务逻辑

## 重要事项

**必需**：测试必须在实现之前编写。TDD 循环如下：

1. **RED** - 编写失败的测试
2. **GREEN** - 实现以使测试通过
3. **REFACTOR** - 改进代码

永远不要跳过 RED 阶段。永远不要在测试之前编写代码。

## 与其他命令的关联

- 首先使用 `/plan` 来理解要创建什么
- 使用 `/tdd` 与测试一起实现
- 出现构建错误时使用 `/build-fix` 进行修复
- 使用 `/code-review` 审查实现
- 使用 `/test-coverage` 验证覆盖率

## 相关 Agent

此命令调用 `tdd-guide` agent：
`~/.claude/agents/tdd-guide.md`

还可以参考 `tdd-workflow` 技能：
`~/.claude/skills/tdd-workflow/`
