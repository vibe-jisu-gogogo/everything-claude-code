---
name: security-review
description: 在实现身份验证、用户输入处理、密钥管理、API 端点创建、支付/敏感功能时使用此技能。提供全面的安全检查清单和模式。
origin: ECC
---

# 安全审查技能

该技能确保所有代码遵循安全最佳实践并识别潜在漏洞。

## 激活时机

- 实现身份验证或授权时
- 处理用户输入或文件上传时
- 创建新的 API 端点时
- 处理密钥或凭证时
- 实现支付功能时
- 存储或传输敏感数据时
- 集成第三方 API 时

## 安全检查清单

### 1. 密钥管理

#### 绝对禁止

```typescript
const apiKey = "sk-proj-xxxxx"  // Hardcoded secret
const dbPassword = "password123" // In source code
```

#### 必须执行

```typescript
const apiKey = process.env.OPENAI_API_KEY
const dbUrl = process.env.DATABASE_URL

// Verify secrets exist
if (!apiKey) {
  throw new Error('OPENAI_API_KEY not configured')
}
```

#### 检查步骤
- [ ] 无硬编码的 API 密钥、令牌、密码
- [ ] 所有密钥存储在环境变量中
- [ ] `.env.local` 包含在 .gitignore 中
- [ ] git 历史中无密钥
- [ ] 生产密钥存储在托管平台（Vercel、Railway）

### 2. 输入验证

#### 始终验证用户输入

```typescript
import { z } from 'zod'

// Define validation schema
const CreateUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1).max(100),
  age: z.number().int().min(0).max(150)
})

// Validate before processing
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
  // Size check (5MB max)
  const maxSize = 5 * 1024 * 1024
  if (file.size > maxSize) {
    throw new Error('File too large (max 5MB)')
  }

  // Type check
  const allowedTypes = ['image/jpeg', 'image/png', 'image/gif']
  if (!allowedTypes.includes(file.type)) {
    throw new Error('Invalid file type')
  }

  // Extension check
  const allowedExtensions = ['.jpg', '.jpeg', '.png', '.gif']
  const extension = file.name.toLowerCase().match(/\.[^.]+$/)?.[0]
  if (!extension || !allowedExtensions.includes(extension)) {
    throw new Error('Invalid file extension')
  }

  return true
}
```

#### 检查步骤
- [ ] 所有用户输入通过 schema 验证
- [ ] 文件上传受限制（大小、类型、扩展名）
- [ ] 用户输入不直接用于查询
- [ ] 使用白名单验证（而非黑名单）
- [ ] 错误消息不暴露敏感信息

### 3. 防止 SQL Injection

#### 绝对不要拼接 SQL

```typescript
// DANGEROUS - SQL Injection vulnerability
const query = `SELECT * FROM users WHERE email = '${userEmail}'`
await db.query(query)
```

#### 必须使用参数化查询

```typescript
// Safe - parameterized query
const { data } = await supabase
  .from('users')
  .select('*')
  .eq('email', userEmail)

// Or with raw SQL
await db.query(
  'SELECT * FROM users WHERE email = $1',
  [userEmail]
)
```

#### 检查步骤
- [ ] 所有数据库查询使用参数化查询
- [ ] SQL 中无字符串拼接
- [ ] ORM/查询构建器正确使用
- [ ] Supabase 查询适当 sanitized

### 4. 身份验证和授权

#### JWT 令牌处理

```typescript
// FAIL: WRONG: localStorage (vulnerable to XSS)
localStorage.setItem('token', token)

// PASS: CORRECT: httpOnly cookies
res.setHeader('Set-Cookie',
  `token=${token}; HttpOnly; Secure; SameSite=Strict; Max-Age=3600`)
```

#### 授权检查

```typescript
export async function deleteUser(userId: string, requesterId: string) {
  // ALWAYS verify authorization first
  const requester = await db.users.findUnique({
    where: { id: requesterId }
  })

  if (requester.role !== 'admin') {
    return NextResponse.json(
      { error: 'Unauthorized' },
      { status: 403 }
    )
  }

  // Proceed with deletion
  await db.users.delete({ where: { id: userId } })
}
```

#### Row Level Security (Supabase)

```sql
-- Enable RLS on all tables
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

-- Users can only view their own data
CREATE POLICY "Users view own data"
  ON users FOR SELECT
  USING (auth.uid() = id);

-- Users can only update their own data
CREATE POLICY "Users update own data"
  ON users FOR UPDATE
  USING (auth.uid() = id);
```

#### 检查步骤
- [ ] 令牌存储在 httpOnly cookie 中（而非 localStorage）
- [ ] 敏感操作前检查授权
- [ ] Supabase 中 Row Level Security 已启用
- [ ] 基于角色的访问控制已实现
- [ ] 会话管理安全

### 5. XSS 防护

#### HTML Sanitizing

```typescript
import DOMPurify from 'isomorphic-dompurify'

// ALWAYS sanitize user-provided HTML
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
      script-src 'self' 'nonce-{nonce}';
      style-src 'self' 'nonce-{nonce}';
      img-src 'self' data: https:;
      font-src 'self';
      connect-src 'self' https://api.example.com;
    `.replace(/\s{2,}/g, ' ').trim()
  }
]
```

`{nonce}` 必须为每个请求重新生成，并在 header 和 inline `<script>`/`<style>` 标签中注入相同的值。

#### 检查步骤
- [ ] 用户提供的 HTML 已 sanitized
- [ ] CSP header 已配置
- [ ] 无未验证的动态内容渲染
- [ ] React 内置 XSS 保护已使用

### 6. CSRF 防护

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

  // Process request
}
```

#### SameSite Cookie

```typescript
res.setHeader('Set-Cookie',
  `session=${sessionId}; HttpOnly; Secure; SameSite=Strict`)
```

#### 检查步骤
- [ ] 状态变更操作应用 CSRF token
- [ ] 所有 cookie 设置 SameSite=Strict
- [ ] Double-submit cookie 模式已实现

### 7. 速率限制

#### API 速率限制

```typescript
import rateLimit from 'express-rate-limit'

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // 100 requests per window
  message: 'Too many requests'
})

// Apply to routes
app.use('/api/', limiter)
```

#### 高成本操作

```typescript
// Aggressive rate limiting for searches
const searchLimiter = rateLimit({
  windowMs: 60 * 1000, // 1 minute
  max: 10, // 10 requests per minute
  message: 'Too many search requests'
})

app.use('/api/search', searchLimiter)
```

#### 检查步骤
- [ ] 所有 API 端点应用速率限制
- [ ] 高成本操作有更严格的限制
- [ ] 基于 IP 的速率限制
- [ ] 基于用户的速率限制（已认证用户）

### 8. 敏感数据暴露

#### 日志记录

```typescript
// FAIL: WRONG: Logging sensitive data
console.log('User login:', { email, password })
console.log('Payment:', { cardNumber, cvv })

// PASS: CORRECT: Redact sensitive data
console.log('User login:', { email, userId })
console.log('Payment:', { last4: card.last4, userId })
```

#### 错误消息

```typescript
// FAIL: WRONG: Exposing internal details
catch (error) {
  return NextResponse.json(
    { error: error.message, stack: error.stack },
    { status: 500 }
  )
}

// PASS: CORRECT: Generic error messages
catch (error) {
  console.error('Internal error:', error)
  return NextResponse.json(
    { error: 'An error occurred. Please try again.' },
    { status: 500 }
  )
}
```

#### 检查步骤
- [ ] 日志中无密码、令牌、密钥
- [ ] 向用户显示的错误消息是通用的
- [ ] 详细错误仅记录在服务器日志中
- [ ] 堆栈跟踪不向用户暴露

### 9. 区块链安全 (Solana)

#### 钱包验证

```typescript
import nacl from 'tweetnacl'
import bs58 from 'bs58'
import { PublicKey } from '@solana/web3.js'

async function verifyWalletOwnership(
  publicKey: string,
  signature: string,
  message: string
) {
  try {
    const publicKeyBytes = new PublicKey(publicKey).toBytes()
    const signatureBytes = bs58.decode(signature)
    const messageBytes = new TextEncoder().encode(message)

    return nacl.sign.detached.verify(
      messageBytes,
      signatureBytes,
      publicKeyBytes
    )
  } catch (error) {
    return false
  }
}
```

注意：Solana 公钥和签名通常使用 base58 编码，而非 base64。

#### 交易验证

```typescript
async function verifyTransaction(transaction: Transaction) {
  // Verify recipient
  if (transaction.to !== expectedRecipient) {
    throw new Error('Invalid recipient')
  }

  // Verify amount
  if (transaction.amount > maxAmount) {
    throw new Error('Amount exceeds limit')
  }

  // Verify user has sufficient balance
  const balance = await getBalance(transaction.from)
  if (balance < transaction.amount) {
    throw new Error('Insufficient balance')
  }

  return true
}
```

#### 检查步骤
- [ ] 钱包签名已验证
- [ ] 交易详情已验证
- [ ] 交易前检查余额
- [ ] 无盲交易签名

### 10. 依赖项安全

#### 定期更新

```bash
# Check for vulnerabilities
npm audit

# Fix automatically fixable issues
npm audit fix

# Update dependencies
npm update

# Check for outdated packages
npm outdated
```

#### 锁定文件

```bash
# ALWAYS commit lock files
git add package-lock.json

# Use in CI/CD for reproducible builds
npm ci  # Instead of npm install
```

#### 检查步骤
- [ ] 依赖项为最新状态
- [ ] 无已知漏洞（npm audit 干净）
- [ ] 锁定文件已提交
- [ ] GitHub 上 Dependabot 已启用
- [ ] 定期安全更新

## 安全测试

### 自动化安全测试

```typescript
// Test authentication
test('requires authentication', async () => {
  const response = await fetch('/api/protected')
  expect(response.status).toBe(401)
})

// Test authorization
test('requires admin role', async () => {
  const response = await fetch('/api/admin', {
    headers: { Authorization: `Bearer ${userToken}` }
  })
  expect(response.status).toBe(403)
})

// Test input validation
test('rejects invalid input', async () => {
  const response = await fetch('/api/users', {
    method: 'POST',
    body: JSON.stringify({ email: 'not-an-email' })
  })
  expect(response.status).toBe(400)
})

// Test rate limiting
test('enforces rate limits', async () => {
  const requests = Array(101).fill(null).map(() =>
    fetch('/api/endpoint')
  )

  const responses = await Promise.all(requests)
  const tooManyRequests = responses.filter(r => r.status === 429)

  expect(tooManyRequests.length).toBeGreaterThan(0)
})
```

## 部署前安全检查清单

所有生产部署前：

- [ ] **密钥**：无硬编码密钥，全部存储在环境变量中
- [ ] **输入验证**：所有用户输入已验证
- [ ] **SQL Injection**：所有查询已参数化
- [ ] **XSS**：用户内容已 sanitized
- [ ] **CSRF**：保护已启用
- [ ] **身份验证**：适当的令牌处理
- [ ] **授权**：角色检查已应用
- [ ] **速率限制**：所有端点已启用
- [ ] **HTTPS**：生产环境已强制
- [ ] **安全 Header**：CSP、X-Frame-Options 已配置
- [ ] **错误处理**：错误中无敏感数据
- [ ] **日志记录**：日志中无敏感数据
- [ ] **依赖项**：为最新状态，无漏洞
- [ ] **Row Level Security**：Supabase 中已启用
- [ ] **CORS**：已适当配置
- [ ] **文件上传**：已验证（大小、类型）
- [ ] **钱包签名**：已验证（如为区块链）

## 参考资料

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Next.js Security](https://nextjs.org/docs/security)
- [Supabase Security](https://supabase.com/docs/guides/auth)
- [Web Security Academy](https://portswigger.net/web-security)

---

**记住**：安全不是可选的。一个漏洞就可能危及整个平台。如有疑问，谨慎行事。
