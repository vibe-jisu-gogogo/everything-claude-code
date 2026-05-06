---
name: coding-standards
description: 适用于 TypeScript、JavaScript、React、Node.js 开发的通用编码标准、最佳实践和设计模式。
---

# 编码标准与最佳实践

适用于所有项目的通用编码标准。

## 代码质量原则

### 1. 可读性优先

* 代码被阅读的次数远多于被编写的次数
* 使用清晰的变量名和函数名
* 优先选择自文档化代码，而非注释
* 保持一致的格式化风格

### 2. KISS (Keep It Simple, Stupid)

* 采用能够正常工作的最简单解决方案
* 避免过度设计
* 避免过早优化
* 易于理解 > 巧妙的代码

### 3. DRY (Don't Repeat Yourself)

* 将共用逻辑提取为函数
* 创建可复用组件
* 在模块间共享工具函数
* 避免复制粘贴式编程

### 4. YAGNI (You Aren't Gonna Need It)

* 不要预先构建暂时不需要的功能
* 避免推测性的泛化设计
* 只在必要时增加复杂度
* 从简单开始，按需重构

## TypeScript/JavaScript 标准

### 变量命名

```typescript
// PASS: GOOD: 描述性名称
const marketSearchQuery = 'election'
const isUserAuthenticated = true
const totalRevenue = 1000

// FAIL: BAD: 不清晰的名称
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

// FAIL: BAD: 不清晰或仅使用名词
async function market(id: string) { }
function similarity(a, b) { }
function email(e) { }
```

### 不可变性模式（重要）

```typescript
// PASS: ALWAYS 使用展开运算符
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

// FAIL: BAD: 无类型，结构不清晰
export function Button(props) {
  return <button onClick={props.onClick}>{props.children}</button>
}
```

### 自定义 Hook

```typescript
// PASS: GOOD: 可复用的自定义 hook
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

### 状态管理

```typescript
// PASS: GOOD: 正确的状态更新
const [count, setCount] = useState(0)

// 基于前一个状态使用函数式更新
setCount(prev => prev + 1)

// FAIL: BAD: 直接引用状态
setCount(count + 1)  // 在异步场景中可能过期
```

### 条件渲染

```typescript
// PASS: GOOD: 清晰的条件渲染
{isLoading && <Spinner />}
{error && <ErrorMessage error={error} />}
{data && <DataDisplay data={data} />}

// FAIL: BAD: 三元运算符嵌套地狱
{isLoading ? <Spinner /> : error ? <ErrorMessage error={error} /> : data ? <DataDisplay data={data} /> : null}
```

## API 设计标准

### REST API 约定

```
GET    /api/markets              # 列出所有市场
GET    /api/markets/:id          # 获取特定市场
POST   /api/markets              # 创建新市场
PUT    /api/markets/:id          # 更新市场（完整）
PATCH  /api/markets/:id          # 更新市场（部分）
DELETE /api/markets/:id          # 删除市场

# 过滤用查询参数
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
    // Proceed with validated data
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

## 文件结构

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
├── hooks/                # 自定义 React Hook
├── lib/                  # 工具类与配置
│   ├── api/             # API 客户端
│   ├── utils/           # 辅助函数
│   └── constants/       # 常量
├── types/                # TypeScript 类型定义
└── styles/              # 全局样式
```

### 文件命名

```
components/Button.tsx          # 组件使用 PascalCase
hooks/useAuth.ts              # Hook 使用带 'use' 前缀的 camelCase
lib/formatDate.ts             # 工具类使用 camelCase
types/market.types.ts         # 类型定义使用带 .types 后缀的 camelCase
```

## 注释与文档

### 何时添加注释

```typescript
// PASS: GOOD: 解释 WHY，而非 WHAT
// 使用指数退避避免在服务中断期间压垮 API
const delay = Math.min(1000 * Math.pow(2, retryCount), 30000)

// 出于大数组性能考量，此处有意使用修改操作
items.push(newItem)

// FAIL: BAD: 陈述显而易见的内容
// 将计数器加 1
count++

// 将名称设置为用户名
name = user.name
```

### 公共 API 的 JSDoc

````typescript
/**
 * 使用语义相似度搜索市场。
 *
 * @param query - 自然语言搜索查询
 * @param limit - 最大结果数（默认：10）
 * @returns 按相似度分数排序的市场数组
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
````

## 性能最佳实践

### 记忆化

```typescript
import { useMemo, useCallback } from 'react'

// PASS: GOOD: 记忆化昂贵计算
const sortedMarkets = useMemo(() => {
  return markets.sort((a, b) => b.volume - a.volume)
}, [markets])

// PASS: GOOD: 记忆化回调
const handleSearch = useCallback((query: string) => {
  setSearchQuery(query)
}, [])
```

### 延迟加载

```typescript
import { lazy, Suspense } from 'react'

// PASS: GOOD: 延迟加载重型组件
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
// PASS: GOOD: 仅选择所需列
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

### 测试结构（AAA 模式）

```typescript
test('calculates similarity correctly', () => {
  // Arrange
  const vector1 = [1, 0, 0]
  const vector2 = [0, 1, 0]

  // Act
  const similarity = calculateCosineSimilarity(vector1, vector2)

  // Assert
  expect(similarity).toBe(0)
})
```

### 测试命名

```typescript
// PASS: GOOD: 描述性测试名称
test('returns empty array when no markets match query', () => { })
test('throws error when OpenAI API key is missing', () => { })
test('falls back to substring search when Redis unavailable', () => { })

// FAIL: BAD: 模糊的测试名称
test('works', () => { })
test('test search', () => { })
```

## 代码异味检测

请注意以下反模式。

### 1. 长函数

```typescript
// FAIL: BAD: 函数超过 50 行
function processMarketData() {
  // 100 lines of code
}

// PASS: GOOD: 拆分为更小的函数
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
          // Do something
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

// Do something
```

### 3. 魔术数字

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

**请记住**：代码质量不可妥协。清晰且可维护的代码使快速开发和充满信心的重构成为可能。
