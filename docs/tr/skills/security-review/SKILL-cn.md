---
name: security-review
description: 在添加身份验证、处理用户输入、使用密钥、创建 API endpoints 或实现支付/敏感功能时使用此技能。提供全面的安全检查清单和模式。
origin: ECC
---

# 安全审查技能

本技能确保所有代码遵循安全最佳实践，并识别潜在的安全漏洞。

## 何时激活

- 实现身份验证或授权时
- 处理用户输入或文件上传时
- 创建新的 API endpoints 时
- 使用密钥或凭证时
- 实现支付功能时
- 存储或传输敏感数据时
- 集成第三方 API 时

## 安全检查清单

### 1. 密钥管理

#### 失败：切勿这样做
```typescript
const apiKey = "sk-proj-xxxxx"  // Hardcoded secret
const dbPassword = "password123" // 源代码中
```

#### 通过：始终这样做
```typescript
const apiKey = process.env.OPENAI_API_KEY
const dbUrl = process.env.DATABASE_URL

// 验证密钥存在
if (!apiKey) {
  throw new Error('OPENAI_API_KEY not configured')
}
```

#### 验证步骤
- [ ] 没有硬编码的 API key、token 或密码
- [ ] 所有密钥都在 environment variables 中
- [ ] `.env.local` 在 .gitignore 中
- [ ] Git history 中没有密钥
- [ ] Production 密钥在托管平台上 (Vercel, Railway)

### 2. 输入验证

#### 始终验证用户输入
```typescript
import { z } from 'zod'

// 定义验证模式
const CreateUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1).max(100),
  age: z.number().int().min(0).max(150)
})

// 处理前验证
export async function createUser(input: unknown) {
  try {
    const validated = CreateUserSchema.parse(input)
    return await db.users.create(validated)
  } catch (error) {
    if (error instanceof z.ZodError) {
      return { success: false, errors: error.errors }
    }
    throw error
  }
}
```

#### 文件上传验证
```typescript
function validateFileUpload(file: File) {
  // 大小检查 (5MB max)
  const maxSize = 5 * 1024 * 1024
  if (file.size > maxSize) {
    throw new Error('文件太大 (最大 5MB)')
  }

  // 类型检查
  const allowedTypes = ['image/jpeg', 'image/png', 'image/gif']
  if (!allowedTypes.includes(file.type)) {
    throw new Error('无效的文件类型')
  }

  // 扩展名检查
  const allowedExtensions = ['.jpg', '.jpeg', '.png', '.gif']
  const extension = file.name.toLowerCase().match(/\.[^.]+$/)?.[0]
  if (!extension || !allowedExtensions.includes(extension)) {
    throw new Error('无效的文件扩展名')
  }

  return true
}
```

#### 验证步骤
- [ ] 所有用户输入都用 schema 验证
- [ ] 文件上传有限制 (大小、类型、扩展名)
- [ ] 用户输入不直接用于查询
- [ ] 使用白名单验证 (不是黑名单)
- [ ] 错误消息不泄露敏感信息

### 3. SQL Injection 预防

#### 失败：切勿使用 SQL 拼接
```typescript
// 危险 - SQL Injection 漏洞
const query = `SELECT * FROM users WHERE email = '${userEmail}'`
await db.query(query)
```

#### 通过：始终使用参数化查询
```typescript
// 安全 - 参数化查询
const { data } = await supabase
  .from('users')
  .select('*')
  .eq('email', userEmail)

// 或使用 raw SQL
await db.query(
  'SELECT * FROM users WHERE email = $1',
  [userEmail]
)
```

#### 验证步骤
- [ ] 所有数据库查询都是参数化的
- [ ] SQL 中没有字符串拼接
- [ ] 正确使用 ORM/query builder
- [ ] Supabase 查询正确 sanitize

### 4. 身份验证和授权

#### JWT Token 处理
```typescript
// 失败：错误：localStorage (易受 XSS 攻击)
localStorage.setItem('token', token)

// 通过：正确：httpOnly cookies
res.setHeader('Set-Cookie',
  `token=${token}; HttpOnly; Secure; SameSite=Strict; Max-Age=3600`)
```

#### 授权检查
```typescript
export async function deleteUser(userId: string, requesterId: string) {
  // 始终首先验证授权
  const requester = await db.users.findUnique({
    where: { id: requesterId }
  })

  if (requester.role !== 'admin') {
    return NextResponse.json(
      { error: 'Unauthorized' },
      { status: 403 }
    )
  }

  // 继续删除操作
  await db.users.delete({ where: { id: userId } })
}
```

#### Row Level Security (Supabase)
```sql
-- 在所有表上启用 RLS
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

-- 用户只能查看自己的数据
CREATE POLICY "Users view own data"
  ON users FOR SELECT
  USING (auth.uid() = id);

-- 用户只能更新自己的数据
CREATE POLICY "Users update own data"
  ON users FOR UPDATE
  USING (auth.uid() = id);
```

#### 验证步骤
- [ ] Token 在 httpOnly cookies 中 (不在 localStorage)
- [ ] 敏感操作前有授权检查
- [ ] Supabase 中 Row Level Security 已启用
- [ ] 已应用基于角色的访问控制
- [ ] Session 管理安全

### 5. XSS 预防

#### 清理 HTML
```typescript
import DOMPurify from 'isomorphic-dompurify'

// 始终清理用户提供的 HTML
function renderUserContent(html: string) {
  const clean = DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'p'],
    ALLOWED_ATTR: []
  })
  return <div dangerouslySetInnerHTML={{ __html: clean }} />
}
```

#### Content Security Policy
```typescript
// next.config.js
const securityHeaders = [
  {
    key: 'Content-Security-Policy',
    value: `
      default-src 'self';
      script-src 'self' 'unsafe-eval' 'unsafe-inline';
      style-src 'self' 'unsafe-inline';
      img-src 'self' data: https:;
      font-src 'self';
      connect-src 'self' https://api.example.com;
    `.replace(/\s{2,}/g, ' ').trim()
  }
]
```

#### 验证步骤
- [ ] 用户提供的 HTML 已清理
- [ ] 已配置 CSP headers
- [ ] 没有未验证的动态内容渲染
- [ ] 使用 React 内置的 XSS 保护

### 6. CSRF 保护

#### CSRF Token
```typescript
import { csrf } from '@/lib/csrf'

export async function POST(request: Request) {
  const token = request.headers.get('X-CSRF-Token')

  if (!csrf.verify(token)) {
    return NextResponse.json(
      { error: 'Invalid CSRF token' },
      { status: 403 }
    )
  }

  // 处理请求
}
```

#### SameSite Cookie
```typescript
res.setHeader('Set-Cookie',
  `session=${sessionId}; HttpOnly; Secure; SameSite=Strict`)
```

#### 验证步骤
- [ ] 状态更改操作中有 CSRF token
- [ ] 所有 cookie 使用 SameSite=Strict
- [ ] 已应用 double-submit cookie 模式

### 7. Rate Limiting

#### API Rate Limiting
```typescript
import rateLimit from 'express-rate-limit'

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 分钟
  max: 100, // 每个窗口 100 个请求
  message: '请求过多'
})

// 应用到路由
app.use('/api/', limiter)
```

#### 昂贵操作
```typescript
// 搜索使用更严格的 rate limiting
const searchLimiter = rateLimit({
  windowMs: 60 * 1000, // 1 分钟
  max: 10, // 每分钟 10 个请求
  message: '搜索请求过多'
})

app.use('/api/search', searchLimiter)
```

#### 验证步骤
- [ ] 所有 API endpoints 都有 rate limiting
- [ ] 昂贵操作有更严格的限制
- [ ] 基于 IP 的 rate limiting
- [ ] 基于用户的 rate limiting (已认证)

### 8. 敏感数据泄露

#### 日志记录
```typescript
// 失败：错误：记录敏感数据
console.log('User login:', { email, password })
console.log('Payment:', { cardNumber, cvv })

// 通过：正确：隐藏敏感数据
console.log('User login:', { email, userId })
console.log('Payment:', { last4: card.last4, userId })
```

#### 错误消息
```typescript
// 失败：错误：暴露内部细节
catch (error) {
  return NextResponse.json(
    { error: error.message, stack: error.stack },
    { status: 500 }
  )
}

// 通过：正确：通用错误消息
catch (error) {
  console.error('Internal error:', error)
  return NextResponse.json(
    { error: '发生错误，请重试。' },
    { status: 500 }
  )
}
```

#### 验证步骤
- [ ] 日志中没有密码、token 或密钥
- [ ] 用户看到通用错误消息
- [ ] 详细错误仅在服务器日志中
- [ ] 不向用户显示 stack trace

### 9. 区块链安全 (Solana)

#### Wallet 验证
```typescript
import { verify } from '@solana/web3.js'

async function verifyWalletOwnership(
  publicKey: string,
  signature: string,
  message: string
) {
  try {
    const isValid = verify(
      Buffer.from(message),
      Buffer.from(signature, 'base64'),
      Buffer.from(publicKey, 'base64')
    )
    return isValid
  } catch (error) {
    return false
  }
}
```

#### Transaction 验证
```typescript
async function verifyTransaction(transaction: Transaction) {
  // 验证接收方
  if (transaction.to !== expectedRecipient) {
    throw new Error('无效的接收方')
  }

  // 验证金额
  if (transaction.amount > maxAmount) {
    throw new Error('超过金额限制')
  }

  // 验证用户有足够余额
  const balance = await getBalance(transaction.from)
  if (balance < transaction.amount) {
    throw new Error('余额不足')
  }

  return true
}
```

#### 验证步骤
- [ ] Wallet 签名已验证
- [ ] Transaction 详细信息已验证
- [ ] Transaction 前有余额检查
- [ ] 没有盲签名 transaction

### 10. 依赖安全性

#### 定期更新
```bash
# 检查安全漏洞
npm audit

# 修复可自动修复的问题
npm audit fix

# 更新依赖
npm update

# 检查过时的包
npm outdated
```

#### Lock 文件
```bash
# 始终 commit lock 文件
git add package-lock.json

# 在 CI/CD 中用于可重现的 build
npm ci  # 代替 npm install
```

#### 验证步骤
- [ ] 依赖已更新
- [ ] 没有已知的安全漏洞 (npm audit clean)
- [ ] Lock 文件已 commit
- [ ] GitHub 上 Dependabot 已激活
- [ ] 定期安全更新

## 安全测试

### 自动安全测试
```typescript
// 身份验证测试
test('需要身份验证', async () => {
  const response = await fetch('/api/protected')
  expect(response.status).toBe(401)
})

// 授权测试
test('需要管理员角色', async () => {
  const response = await fetch('/api/admin', {
    headers: { Authorization: `Bearer ${userToken}` }
  })
  expect(response.status).toBe(403)
})

// 输入验证测试
test('拒绝无效输入', async () => {
  const response = await fetch('/api/users', {
    method: 'POST',
    body: JSON.stringify({ email: 'not-an-email' })
  })
  expect(response.status).toBe(400)
})

// Rate limiting 测试
test('强制执行 rate limits', async () => {
  const requests = Array(101).fill(null).map(() =>
    fetch('/api/endpoint')
  )

  const responses = await Promise.all(requests)
  const tooManyRequests = responses.filter(r => r.status === 429)

  expect(tooManyRequests.length).toBeGreaterThan(0)
})
```

## Deployment 前安全检查清单

在任何 production deployment 前：

- [ ] **密钥**：没有硬编码密钥，全部在 env vars 中
- [ ] **输入验证**：所有用户输入已验证
- [ ] **SQL Injection**：所有查询参数化
- [ ] **XSS**：用户内容已清理
- [ ] **CSRF**：保护已激活
- [ ] **身份验证**：正确的 token 处理
- [ ] **授权**：角色检查已到位
- [ ] **Rate Limiting**：所有 endpoints 已激活
- [ ] **HTTPS**：Production 中强制
- [ ] **安全 Headers**：CSP、X-Frame-Options 已配置
- [ ] **错误处理**：错误中没有敏感数据
- [ ] **日志记录**：不记录敏感数据
- [ ] **依赖**：已更新，无安全漏洞
- [ ] **Row Level Security**：Supabase 中已激活
- [ ] **CORS**：正确配置
- [ ] **文件上传**：已验证 (大小、类型)
- [ ] **Wallet 签名**：已验证 (如果有 blockchain)

## 资源

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Next.js Security](https://nextjs.org/docs/security)
- [Supabase Security](https://supabase.com/docs/guides/auth)
- [Web Security Academy](https://portswigger.net/web-security)

---

**记住**：安全不是可选的。一个安全漏洞可能危及整个平台。有疑问时，谨慎行事。
