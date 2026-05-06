---
paths:
  - "**/*.ts"
  - "**/*.tsx"
  - "**/*.js"
  - "**/*.jsx"
---
# TypeScript/JavaScript 编码风格

> 此文件扩展了 [common/coding-style.md](../common/coding-style.md)，添加了 TypeScript/JavaScript 特定内容。

## 类型和接口

使用类型使公共 API、共享模型和组件 props 变得清晰、可读且可重用。

### 公共 API

- 为导出的函数、共享的工具函数和公共类方法添加参数和返回类型
- 允许 TypeScript 推断局部变量的类型
- 将重复的内联对象形状提取为命名类型或接口

```typescript
// 错误：导出函数没有明确类型
export function formatUser(user) {
  return `${user.firstName} ${user.lastName}`
}

// 正确：公共 API 有明确类型
interface User {
  firstName: string
  lastName: string
}

export function formatUser(user: User): string {
  return `${user.firstName} ${user.lastName}`
}
```

### 接口与类型别名

- 对于可扩展或可实现的对象形状使用 `interface`
- 对于联合类型、交叉类型、元组、映射类型和工具类型使用 `type`
- 除非互操作性需要 `enum`，否则优先使用字符串字面量联合类型

```typescript
interface User {
  id: string
  email: string
}

type UserRole = 'admin' | 'member'
type UserWithRole = User & {
  role: UserRole
}
```

### 避免使用 `any`

- 在应用代码中避免使用 `any`
- 对于外部或不受信任的输入使用 `unknown`，然后安全地进行类型收窄
- 当值的类型取决于调用者时使用泛型

```typescript
// 错误：any 移除了类型安全性
function getErrorMessage(error: any) {
  return error.message
}

// 正确：unknown 强制进行安全的类型收窄
function getErrorMessage(error: unknown): string {
  if (error instanceof Error) {
    return error.message
  }

  return 'Unexpected error'
}
```

### React Props

- 使用命名的 `interface` 或 `type` 定义组件 props
- 明确类型化回调 props
- 除非有特定原因，否则不要使用 `React.FC`

```typescript
interface User {
  id: string
  email: string
}

interface UserCardProps {
  user: User
  onSelect: (id: string) => void
}

function UserCard({ user, onSelect }: UserCardProps) {
  return <button onClick={() => onSelect(user.id)}>{user.email}</button>
}
```

### JavaScript 文件

- 在 `.js` 和 `.jsx` 文件中，当类型可以提高清晰度且 TypeScript 迁移不切实际时使用 JSDoc
- 保持 JSDoc 与运行时行为一致

```javascript
/**
 * @param {{ firstName: string, lastName: string }} user
 * @returns {string}
 */
export function formatUser(user) {
  return `${user.firstName} ${user.lastName}`
}
```

## 不可变性

使用展开运算符进行不可变更新：

```typescript
interface User {
  id: string
  name: string
}

// 错误：直接修改
function updateUser(user: User, name: string): User {
  user.name = name // 修改了！
  return user
}

// 正确：不可变更新
function updateUser(user: Readonly<User>, name: string): User {
  return {
    ...user,
    name
  }
}
```

## 错误处理

使用 try-catch 和 async/await，并安全地收窄 unknown 错误：

```typescript
interface User {
  id: string
  email: string
}

declare function riskyOperation(userId: string): Promise<User>

function getErrorMessage(error: unknown): string {
  if (error instanceof Error) {
    return error.message
  }

  return 'Unexpected error'
}

const logger = {
  error: (message: string, error: unknown) => {
    // 替换为你的生产日志记录器（例如 pino 或 winston）。
  }
}

async function loadUser(userId: string): Promise<User> {
  try {
    const result = await riskyOperation(userId)
    return result
  } catch (error: unknown) {
    logger.error('Operation failed', error)
    throw new Error(getErrorMessage(error))
  }
}
```

## 输入验证

使用 Zod 进行基于 Schema 的验证，并从 Schema 中推断类型：

```typescript
import { z } from 'zod'

const userSchema = z.object({
  email: z.string().email(),
  age: z.number().int().min(0).max(150)
})

type UserInput = z.infer<typeof userSchema>

const validated: UserInput = userSchema.parse(input)
```

## Console.log

- 生产代码中没有 `console.log` 语句
- 改用适当的日志库
- 查看 hooks 进行自动检测
