---
name: coding-standards
description: 跨项目的基础编码约定，涵盖命名、可读性、不可变性和代码质量审查。特定框架的模式请使用详细的前端或后端技能。
---

# 编码标准与最佳实践

适用于所有项目的基础编码约定。

本技能是通用基础规范，而非详细的框架操作指南。

- 对于 React、状态管理、表单、渲染和 UI 架构，请使用 `frontend-patterns`。
- 对于仓储/服务层、端点设计、验证和服务器特定问题，请使用 `backend-patterns` 或 `api-design`。
- 当你需要最精简的可复用规则层而非完整的技能指南时，请使用 `rules/common/coding-style.md`。

## 激活时机

- 启动新项目或模块时
- 审查代码质量和可维护性时
- 重构现有代码以遵循约定时
- 强制执行命名、格式或结构一致性时
- 设置 linting、格式化或类型检查规则时
- 向新贡献者介绍编码约定时

## 适用边界

激活本技能用于：
- 描述性命名
- 默认使用不可变性
- 可读性、KISS、DRY 和 YAGNI 原则执行
- 错误处理预期和代码坏味道审查

不要将本技能作为以下场景的主要参考：
- React 组合、hooks 或渲染模式
- 后端架构、API 设计或数据库分层
- 当已有更细分的 ECC 技能时，不要使用本技能作为特定领域框架指南

## 代码质量原则

### 1. 可读性优先
- 代码被阅读的次数远多于被编写的次数
- 清晰的变量和函数命名
- 优先编写自解释代码而非注释
- 一致的格式

### 2. KISS (Keep It Simple, Stupid)
- 能运行的最简单解决方案
- 避免过度工程
- 不要过早优化
- 易于理解 > 巧妙的代码

### 3. DRY (Don't Repeat Yourself)
- 将通用逻辑提取到函数中
- 创建可复用组件
- 跨模块共享工具函数
- 避免复制粘贴编程

### 4. YAGNI (You Aren't Gonna Need It)
- 不要在需求出现前就开发功能
- 避免投机性的通用性设计
- 仅在必要时增加复杂度
- 从简单开始，需要时再重构

## TypeScript/JavaScript 标准

### 变量命名

```typescript
// 正确: 优秀：描述性命名
const marketSearchQuery = 'election'
const isUserAuthenticated = true
const totalRevenue = 1000

// 错误: 糟糕：含义不明确的命名
const q = 'election'
const flag = true
const x = 1000
```

### 函数命名

```typescript
// 正确: 优秀：动词-名词模式
async function fetchMarketData(marketId: string) { }
function calculateSimilarity(a: number[], b: number[]) { }
function isValidEmail(email: string): boolean { }

// 错误: 糟糕：含义不明确或仅使用名词
async function market(id: string) { }
function similarity(a, b) { }
function email(e) { }
```

### 不可变性模式（关键）

```typescript
// 正确: 始终使用展开运算符
const updatedUser = {
  ...user,
  name: 'New Name'
}

const updatedArray = [...items, newItem]

// 错误: 永远不要直接修改
user.name = 'New Name'  // 糟糕
items.push(newItem)     // 糟糕
```

### 错误处理

```typescript
// 正确: 优秀：完善的错误处理
async function fetchData(url: string) {
  try {
    const response = await fetch(url)

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`)
    }

    return await response.json()
  } catch (error) {
    console.error('Fetch failed:', error)
    throw new Error('Failed to fetch data')
  }
}

// 错误: 糟糕：无错误处理
async function fetchData(url) {
  const response = await fetch(url)
  return response.json()
}
```

### Async/Await 最佳实践

```typescript
// 正确: 优秀：尽可能并行执行
const [users, markets, stats] = await Promise.all([
  fetchUsers(),
  fetchMarkets(),
  fetchStats()
])

// 错误: 糟糕：不必要的串行执行
const users = await fetchUsers()
const markets = await fetchMarkets()
const stats = await fetchStats()
```

### 类型安全

```typescript
// 正确: 优秀：恰当的类型定义
interface Market {
  id: string
  name: string
  status: 'active' | 'resolved' | 'closed'
  created_at: Date
}

function getMarket(id: string): Promise<Market> {
  // 实现代码
}

// 错误: 糟糕：使用 'any'
function getMarket(id: any): Promise<any> {
  // 实现代码
}
```

## React 最佳实践

### 组件结构

```typescript
// 正确: 优秀：带类型的函数式组件
interface ButtonProps {
  children: React.ReactNode
  onClick: () => void
  disabled?: boolean
  variant?: 'primary' | 'secondary'
}

export function Button({
  children,
  onClick,
  disabled = false,
  variant = 'primary'
}: ButtonProps) {
  return (
    <button
      onClick={onClick}
      disabled={disabled}
      className={`btn btn-${variant}`}
    >
      {children}
    </button>
  )
}

// 错误: 糟糕：无类型，结构不清晰
export function Button(props) {
  return <button onClick={props.onClick}>{props.children}</button>
}
```

### 自定义 Hooks

```typescript
// 正确: 优秀：可复用的自定义 hook
export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value)

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value)
    }, delay)

    return () => clearTimeout(handler)
  }, [value, delay])

  return debouncedValue
}

// 使用示例
const debouncedQuery = useDebounce(searchQuery, 500)
```

### 状态管理

```typescript
// 正确: 优秀：正确的状态更新
const [count, setCount] = useState(0)

// 基于之前的状态更新时使用函数式更新
setCount(prev => prev + 1)

// 错误: 糟糕：直接引用状态
setCount(count + 1)  // 异步场景下可能出现陈旧值
```

### 条件渲染

```typescript
// 正确: 优秀：清晰的条件渲染
{isLoading && <Spinner />}
{error && <ErrorMessage error={error} />}
{data && <DataDisplay data={data} />}

// 错误: 糟糕：三元运算符地狱
{isLoading ? <Spinner /> : error ? <ErrorMessage error={error} /> : data ? <DataDisplay data={data} /> : null}
```

## API 设计标准

### REST API 约定

```
GET    /api/markets              # 列出所有市场
GET    /api/markets/:id          # 获取指定市场
POST   /api/markets              # 创建新市场
PUT    /api/markets/:id          # 全量更新市场
PATCH  /api/markets/:id          # 部分更新市场
DELETE /api/markets/:id          # 删除市场

# 用于过滤的查询参数
GET /api/markets?status=active&limit=10&offset=0
```

### 响应格式

```typescript
// 正确: 优秀：一致的响应结构
interface ApiResponse<T> {
  success: boolean
  data?: T
  error?: string
  meta?: {
    total: number
    page: number
    limit: number
  }
}

// 成功响应
return NextResponse.json({
  success: true,
  data: markets,
  meta: { total: 100, page: 1, limit: 10 }
})

// 错误响应
return NextResponse.json({
  success: false,
  error: 'Invalid request'
}, { status: 400 })
```

### 输入验证

```typescript
import { z } from 'zod'

// 正确: 优秀：Schema 验证
const CreateMarketSchema = z.object({
  name: z.string().min(1).max(200),
  description: z.string().min(1).max(2000),
  endDate: z.string().datetime(),
  categories: z.array(z.string()).min(1)
})

export async function POST(request: Request) {
  const body = await request.json()

  try {
    const validated = CreateMarketSchema.parse(body)
    // 使用校验后的数据继续处理
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json({
        success: false,
        error: 'Validation failed',
        details: error.errors
      }, { status: 400 })
    }
  }
}
```

## 文件组织

### 项目结构

```
src/
├── app/                    # Next.js App Router
│   ├── api/               # API 路由
│   ├── markets/           # 市场页面
│   └── (auth)/           # 认证页面（路由组）
├── components/            # React 组件
│   ├── ui/               # 通用 UI 组件
│   ├── forms/            # 表单组件
│   └── layouts/          # 布局组件
├── hooks/                # 自定义 React hooks
├── lib/                  # 工具函数和配置
│   ├── api/             # API 客户端
│   ├── utils/           # 辅助函数
│   └── constants/       # 常量
├── types/                # TypeScript 类型定义
└── styles/              # 全局样式
```

### 文件命名

```
components/Button.tsx          # 组件使用 PascalCase
hooks/useAuth.ts              # hooks 使用 camelCase 并加 'use' 前缀
lib/formatDate.ts             # 工具函数使用 camelCase
types/market.types.ts         # 类型定义使用 camelCase 并加 .types 后缀
```

## 注释与文档

### 何时添加注释

```typescript
// 正确: 优秀：解释原因，而非内容
// 使用指数退避避免故障期间 API 负载过高
const delay = Math.min(1000 * Math.pow(2, retryCount), 30000)

// 出于大数组性能考虑，此处特意使用修改操作
items.push(newItem)

// 错误: 糟糕：陈述显而易见的内容
// 计数器加 1
count++

// 将名称设置为用户名
name = user.name
```

### 公共 API 的 JSDoc

```typescript
/**
 * 使用语义相似度搜索市场。
 *
 * @param query - 自然语言搜索查询
 * @param limit - 最大结果数量（默认：10）
 * @returns 按相似度评分排序的市场数组
 * @throws {Error} 当 OpenAI API 调用失败或 Redis 不可用时抛出
 *
 * @example
 * ```typescript
 * const results = await searchMarkets('election', 5)
 * console.log(results[0].name) // "Trump vs Biden"
 * ```
 */
export async function searchMarkets(
  query: string,
  limit: number = 10
): Promise<Market[]> {
  // 实现代码
}
```

## 性能最佳实践

### Memoization

```typescript
import { useMemo, useCallback } from 'react'

// 正确: 优秀：记忆化昂贵的计算
const sortedMarkets = useMemo(() => {
  return markets.sort((a, b) => b.volume - a.volume)
}, [markets])

// 正确: 优秀：记忆化回调函数
const handleSearch = useCallback((query: string) => {
  setSearchQuery(query)
}, [])
```

### 懒加载

```typescript
import { lazy, Suspense } from 'react'

// 正确: 优秀：懒加载重型组件
const HeavyChart = lazy(() => import('./HeavyChart'))

export function Dashboard() {
  return (
    <Suspense fallback={<Spinner />}>
      <HeavyChart />
    </Suspense>
  )
}
```

### 数据库查询

```typescript
// 正确: 优秀：仅选择需要的列
const { data } = await supabase
  .from('markets')
  .select('id, name, status')
  .limit(10)

// 错误: 糟糕：选择所有列
const { data } = await supabase
  .from('markets')
  .select('*')
```

## 测试标准

### 测试结构（AAA 模式）

```typescript
test('calculates similarity correctly', () => {
  // Arrange 准备
  const vector1 = [1, 0, 0]
  const vector2 = [0, 1, 0]

  // Act 执行
  const similarity = calculateCosineSimilarity(vector1, vector2)

  // Assert 断言
  expect(similarity).toBe(0)
})
```

### 测试命名

```typescript
// 正确: 优秀：描述性的测试名称
test('returns empty array when no markets match query', () => { })
test('throws error when OpenAI API key is missing', () => { })
test('falls back to substring search when Redis unavailable', () => { })

// 错误: 糟糕：模糊的测试名称
test('works', () => { })
test('test search', () => { })
```

## 代码坏味道检测

留意以下反模式：

### 1. 过长函数
```typescript
// 错误: 糟糕：函数超过 50 行
function processMarketData() {
  // 100 行代码
}

// 正确: 优秀：拆分为更小的函数
function processMarketData() {
  const validated = validateData()
  const transformed = transformData(validated)
  return saveData(transformed)
}
```

### 2. 深层嵌套
```typescript
// 错误: 糟糕：嵌套层级超过 5 层
if (user) {
  if (user.isAdmin) {
    if (market) {
      if (market.isActive) {
        if (hasPermission) {
          // 执行操作
        }
      }
    }
  }
}

// 正确: 优秀：提前返回
if (!user) return
if (!user.isAdmin) return
if (!market) return
if (!market.isActive) return
if (!hasPermission) return

// 执行操作
```

### 3. 魔法数字
```typescript
// 错误: 糟糕：未解释的数字
if (retryCount > 3) { }
setTimeout(callback, 500)

// 正确: 优秀：具名常量
const MAX_RETRIES = 3
const DEBOUNCE_DELAY_MS = 500

if (retryCount > MAX_RETRIES) { }
setTimeout(callback, DEBOUNCE_DELAY_MS)
```

**注意**：代码质量是不可妥协的。清晰、可维护的代码是快速开发和放心重构的基础。
