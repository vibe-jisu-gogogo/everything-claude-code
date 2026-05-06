---
name: security-review
description: 在添加认证、处理用户输入、管理密钥、创建API端点、实现支付/敏感功能时使用此技能。提供全面的安全检查清单和模式。
origin: ECC
---

# 安全审查技能

本技能确保所有代码遵循安全最佳实践并识别潜在漏洞。

## 何时使用

- 实现认证或授权时
- 处理用户输入或文件上传时
- 创建新的API端点时
- 处理密钥或凭证时
- 实现支付功能时
- 存储或传输敏感数据时
- 集成第三方API时

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
- [ ] 没有硬编码的API密钥、令牌、密码
- [ ] 所有密钥存储在环境变量中
- [ ] `.env.local` 包含在 .gitignore 中
- [ ] git历史记录中没有密钥
- [ ] 生产环境密钥存储在托管平台(Vercel, Railway)

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
- [ ] 所有用户输入都通过schema验证
- [ ] 文件上传受限（大小、类型、扩展名）
- [ ] 用户输入不直接用于查询
- [ ] 使用白名单验证（而非黑名单）
- [ ] 错误消息不暴露敏感信息

### 3. 防止SQL注入

#### 绝对不要拼接SQL
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
- [ ] SQL中没有字符串拼接
- [ ] ORM/查询构建器正确使用
- [ ] Supabase查询经过适当清理

### 4. 认证和授权

#### JWT令牌处理
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
- [ ] 令牌存储在httpOnly cookie中（而非localStorage）
- [ ] 敏感操作前进行授权检查
- [ ] Supabase中启用Row Level Security
- [ ] 实现基于角色的访问控制
- [ ] 会话管理安全

### 5. XSS防护

#### HTML清理
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

`{nonce}`必须在每个请求时重新生成，并在header和内联`<script>`/`<style>`标签中注入相同的值。

#### 检查步骤
- [ ] 用户提供的HTML经过清理
- [ ] 配置CSP header
- [ ] 没有未经验证的动态内容渲染
- [ ] 使用React内置的XSS保护

### 6. CSRF防护

#### CSRF令牌
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
- [ ] 状态变更操作应用CSRF令牌
- [ ] 所有Cookie设置SameSite=Strict
- [ ] 实现Double-submit cookie模式

### 7. 速率限制

#### API速率限制
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
- [ ] 所有API端点应用速率限制
- [ ] 高成本操作有更严格的限制
- [ ] 基于IP的速率限制
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
- [ ] 日志中没有密码、令牌、密钥
- [ ] 向用户显示的错误消息是通用的
- [ ] 详细错误仅记录在服务器日志中
- [ ] 堆栈跟踪不暴露给用户

### 9. 区块链安全（Solana）

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

注意：Solana公钥和签名通常使用base58编码，而非base64。

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
- [ ] 没有盲签名交易

### 10. 依赖安全

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
- [ ] 依赖是最新的
- [ ] 没有已知漏洞（npm audit clean）
- [ ] 锁定文件已提交
- [ ] GitHub中启用Dependabot
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

所有生产环境部署前：

- [ ] **密钥**：没有硬编码的密钥，全部存储在环境变量中
- [ ] **输入验证**：所有用户输入已验证
- [ ] **SQL注入**：所有查询参数化
- [ ] **XSS**：用户内容已清理
- [ ] **CSRF**：防护已启用
- [ ] **认证**：适当的令牌处理
- [ ] **授权**：应用角色检查
- [ ] **速率限制**：所有端点已启用
- [ ] **HTTPS**：生产环境强制启用
- [ ] **安全Header**：配置CSP、X-Frame-Options
- [ ] **错误处理**：错误中没有敏感数据
- [ ] **日志记录**：日志中没有敏感数据
- [ ] **依赖**：最新状态，无漏洞
- [ ] **Row Level Security**：Supabase中已启用
- [ ] **CORS**：适当配置
- [ ] **文件上传**：已验证（大小、类型）
- [ ] **钱包签名**：已验证（如为区块链）

## 参考资料

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Next.js Security](https://nextjs.org/docs/security)
- [Supabase Security](https://supabase.com/docs/guides/auth)
- [Web Security Academy](https://portswigger.net/web-security)

---

**记住**：安全不是可选项。一个漏洞可能危及整个平台。有疑问时，采取保守措施。
