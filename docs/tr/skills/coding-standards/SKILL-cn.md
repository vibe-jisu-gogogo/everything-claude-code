---
name: coding-standards
description: TypeScript, JavaScript, React 和 Node.js 开发的通用编码标准、最佳实践和模式。
origin: ECC
---

# 编码标准和最佳实践

适用于所有项目的通用编码标准。

## 何时激活

- 启动新项目或模块时
- 审查代码质量和可维护性时
- 重构现有代码以符合规则时
- 强制命名、格式化或结构一致性时
- 设置 Linting、格式化或类型检查规则时
- 培训新贡献者遵守编码规则时

## 代码质量原则

### 1. 可读性优先
- 代码被阅读的次数远多于被编写的次数
- 清晰的变量和函数命名
- 优先选择自文档化代码而非注释
- 一致的格式化

### 2. KISS (Keep It Simple, Stupid - 保持简单)
- 能工作的最简单解决方案
- 避免过度工程化
- 不要过早优化
- 易懂的代码 > 巧妙的代码

### 3. DRY (Don't Repeat Yourself - 不要重复自己)
- 将通用逻辑提取到函数中
- 创建可重用的组件
- 在模块之间共享辅助工具
- 避免复制粘贴式编程

### 4. YAGNI (You Aren't Gonna Need It - 你不会需要它)
- 不需要时不要构建功能
- 避免推测性泛化
- 仅在需要时增加复杂性
- 从简单开始，需要时重构

## TypeScript/JavaScript 标准

### 变量命名

```typescript
// PASS: GOOD: 描述性命名
const marketSearchQuery = 'election'
const isUserAuthenticated = true
const totalRevenue = 1000

// FAIL: BAD: 模糊命名
const q = 'election'
const flag = true
const x = 1000
```

### 函数命名

```typescript
// PASS: GOOD: 动词-名词模式
async function fetchMarketData(marketId: string) { }
function calculateSimilarity(a: number[], b: number[]) { }
function isValidEmail(email: string): boolean { }

// FAIL: BAD: 模糊或仅名词
async function market(id: string) { }
function similarity(a, b) { }
function email(e) { }
```

### 不可变性模式 (CRITICAL)

```typescript
// PASS: ALWAYS 使用 spread 操作符
const updatedUser = {
  ...user,
  name: 'New Name'
}

const updatedArray = [...items, newItem]

// FAIL: NEVER 直接修改
user.name = 'New Name'  // BAD
items.push(newItem)     // BAD
```

### 错误处理

```typescript
// PASS: GOOD: 全面的错误处理
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

// FAIL: BAD: 无错误处理
async function fetchData(url) {
  const response = await fetch(url)
  return response.json()
}
```

### Async/Await 最佳实践

```typescript
// PASS: GOOD: 尽可能并行执行
const [users, markets, stats] = await Promise.all([
  fetchUsers(),
  fetchMarkets(),
  fetchStats()
])

// FAIL: BAD: 不必要的顺序执行
const users = await fetchUsers()
const markets = await fetchMarkets()
const stats = await fetchStats()
```

### 类型安全

```typescript
// PASS: GOOD: 正确的类型
interface Market {
  id: string
  name: string
  status: 'active' | 'resolved' | 'closed'
  created_at: Date
}

function getMarket(id: string): Promise<Market> {
  // Implementation
}

// FAIL: BAD: 使用 'any'
function getMarket(id: any): Promise<any> {
  // Implementation
}
```

## React 最佳实践

### 组件结构

```typescript
// PASS: GOOD: 带类型的函数式组件
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

// FAIL: BAD: 无类型，结构模糊
export function Button(props) {
  return <button onClick={props.onClick}>{props.children}</button>
}
```

### 自定义 Hooks

```typescript
// PASS: GOOD: 可重用的自定义 hook
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

// Usage
const debouncedQuery = useDebounce(searchQuery, 500)
```

### State 管理

```typescript
// PASS: GOOD: 正确的 state 更新
const [count, setCount] = useState(0)

// 基于前一个 state 的函数式更新
setCount(prev => prev + 1)

// FAIL: BAD: 直接 state 引用
setCount(count + 1)  // 在异步场景中可能是旧值
```

### 条件渲染

```typescript
// PASS: GOOD: 明确的条件渲染
{isLoading && <Spinner />}
{error && <ErrorMessage error={error} />}
{data && <DataDisplay data={data} />}

// FAIL: BAD: 三元运算符地狱
{isLoading ? <Spinner /> : error ? <ErrorMessage error={error} /> : data ? <DataDisplay data={data} /> : null}
```

## API 设计标准

### REST API 规则

```
GET    /api/markets              # 列出所有 markets
GET    /api/markets/:id          # 获取特定 market
POST   /api/markets              # 创建新 market
PUT    /api/markets/:id          # 更新 market (完整)
PATCH  /api/markets/:id          # 更新 market (部分)
DELETE /api/markets/:id          # 删除 market

# 过滤使用 query 参数
GET /api/markets?status=active&limit=10&offset=0
```

### 响应格式

```typescript
// PASS: GOOD: 一致的响应结构
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

// PASS: GOOD: Schema 验证
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
    // 继续使用验证后的数据
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
│   ├── api/               # API routes
│   ├── markets/           # Market 页面
│   └── (auth)/           # Auth 页面 (route groups)
├── components/            # React 组件
│   ├── ui/               # 通用 UI 组件
│   ├── forms/            # Form 组件
│   └── layouts/          # Layout 组件
├── hooks/                # 自定义 React hooks
├── lib/                  # 辅助工具和配置
│   ├── api/             # API 客户端
│   ├── utils/           # 辅助函数
│   └── constants/       # 常量
├── types/                # TypeScript 类型
└── styles/              # 全局样式
```

### 文件命名

```
components/Button.tsx          # 组件使用 PascalCase
hooks/useAuth.ts              # hooks 使用 'use' 前缀的 camelCase
lib/formatDate.ts             # 辅助工具使用 camelCase
types/market.types.ts         # 类型使用 .types 后缀的 camelCase
```

## 注释和文档

### 何时添加注释

```typescript
// PASS: GOOD: 解释为什么，而非是什么
// 使用 exponential backoff 避免在中断期间过度加载 API
const delay = Math.min(1000 * Math.pow(2, retryCount), 30000)

// 为了大型数组的性能故意在此处使用 mutation
items.push(newItem)

// FAIL: BAD: 陈述显而易见的事情
// 将计数器加1
count++

// 将名称设置为用户的名称
name = user.name
```

### 公共 API 的 JSDoc

```typescript
/**
 * 使用语义相似度搜索 markets。
 *
 * @param query - 自然语言搜索查询
 * @param limit - 最大结果数 (默认: 10)
 * @returns 按相似度分数排序的 market 数组
 * @throws {Error} 如果 OpenAI API 失败或 Redis 不可用
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
  // Implementation
}
```

## 性能最佳实践

### Memoization

```typescript
import { useMemo, useCallback } from 'react'

// PASS: GOOD: Memoize 昂贵的计算
const sortedMarkets = useMemo(() => {
  return markets.sort((a, b) => b.volume - a.volume)
}, [markets])

// PASS: GOOD: Memoize callbacks
const handleSearch = useCallback((query: string) => {
  setSearchQuery(query)
}, [])
```

### Lazy Loading

```typescript
import { lazy, Suspense } from 'react'

// PASS: GOOD: Lazy 加载重型组件
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
// PASS: GOOD: 只选择需要的列
const { data } = await supabase
  .from('markets')
  .select('id, name, status')
  .limit(10)

// FAIL: BAD: 选择所有内容
const { data } = await supabase
  .from('markets')
  .select('*')
```

## 测试标准

### 测试结构 (AAA 模式)

```typescript
test('正确计算相似度', () => {
  // Arrange (准备)
  const vector1 = [1, 0, 0]
  const vector2 = [0, 1, 0]

  // Act (执行)
  const similarity = calculateCosineSimilarity(vector1, vector2)

  // Assert (断言)
  expect(similarity).toBe(0)
})
```

### 测试命名

```typescript
// PASS: GOOD: 描述性测试名称
test('未找到匹配查询的 markets 时返回空数组', () => { })
test('缺少 OpenAI API 密钥时抛出错误', () => { })
test('Redis 不可用时回退到子字符串搜索', () => { })

// FAIL: BAD: 模糊的测试名称
test('工作正常', () => { })
test('搜索测试', () => { })
```

## 代码异味检测

注意这些反模式：

### 1. 长函数

```typescript
// FAIL: BAD: 超过50行的函数
function processMarketData() {
  // 100 行代码
}

// PASS: GOOD: 拆分成小函数
function processMarketData() {
  const validated = validateData()
  const transformed = transformData(validated)
  return saveData(transformed)
}
```

### 2. 深层嵌套

```typescript
// FAIL: BAD: 5+ 层嵌套
if (user) {
  if (user.isAdmin) {
    if (market) {
      if (market.isActive) {
        if (hasPermission) {
          // 做某事
        }
      }
    }
  }
}

// PASS: GOOD: 提前返回
if (!user) return
if (!user.isAdmin) return
if (!market) return
if (!market.isActive) return
if (!hasPermission) return

// 做某事
```

### 3. 魔法数字

```typescript
// FAIL: BAD: 未解释的数字
if (retryCount > 3) { }
setTimeout(callback, 500)

// PASS: GOOD: 命名常量
const MAX_RETRIES = 3
const DEBOUNCE_DELAY_MS = 500

if (retryCount > MAX_RETRIES) { }
setTimeout(callback, DEBOUNCE_DELAY_MS)
```

**记住**: 代码质量不是谈判筹码。清晰、可维护的代码实现快速开发和安全的重构。
