---
name: security-review
description: 在添加认证、处理用户输入、操作密钥、创建API端点、实现支付/敏感功能时使用此技能。提供全面的安全检查清单和模式。
---

# 安全审查技能

本技能确保所有代码遵循安全最佳实践，并识别潜在的漏洞。

## 启用时机

- 实现认证或授权
- 处理用户输入或文件上传
- 创建新的API端点
- 操作密钥或凭据
- 实现支付功能
- 存储或传输敏感数据
- 集成第三方API

## 安全检查清单

### 1. 密钥管理

#### FAIL: 绝对不要做
```typescript
const apiKey = "sk-proj-xxxxx"  // 硬编码的密钥
const dbPassword = "password123" // 在源代码中
```

#### PASS: 始终要做
```typescript
const apiKey = process.env.OPENAI_API_KEY
const dbUrl = process.env.DATABASE_URL

// 确认密钥存在
if (!apiKey) {
  throw new Error('OPENAI_API_KEY not configured')
}
```

#### 验证步骤
- [ ] 没有硬编码的API密钥、令牌、密码
- [ ] 所有密钥都使用环境变量
- [ ] `.env.local` 在 .gitignore 中
- [ ] Git历史中没有密钥
- [ ] 生产环境密钥存储在托管平台（Vercel、Railway）

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
  // 大小检查（最大5MB）
  const maxSize = 5 * 1024 * 1024
  if (file.size > maxSize) {
    throw new Error('File too large (max 5MB)')
  }

  // 类型检查
  const allowedTypes = ['image/jpeg', 'image/png', 'image/gif']
  if (!allowedTypes.includes(file.type)) {
    throw new Error('Invalid file type')
  }

  // 扩展名检查
  const allowedExtensions = ['.jpg', '.jpeg', '.png', '.gif']
  const extension = file.name.toLowerCase().match(/\.[^.]+$/)?.[0]
  if (!extension || !allowedExtensions.includes(extension)) {
    throw new Error('Invalid file extension')
  }

  return true
}
```

#### 验证步骤
- [ ] 所有用户输入都通过模式验证
- [ ] 文件上传限制（大小、类型、扩展名）
- [ ] 不直接在查询中使用用户输入
- [ ] 使用白名单验证（而非黑名单）
- [ ] 错误消息不泄露敏感信息

### 3. SQL注入防护

#### FAIL: 绝对不要拼接SQL
```typescript
// 危险 - SQL注入漏洞
const query = `SELECT * FROM users WHERE email = '${userEmail}'`
await db.query(query)
```

#### PASS: 始终使用参数化查询
```typescript
// 安全 - 参数化查询
const { data } = await supabase
  .from('users')
  .select('*')
  .eq('email', userEmail)

// 或者使用原生SQL
await db.query(
  'SELECT * FROM users WHERE email = $1',
  [userEmail]
)
```

#### 验证步骤
- [ ] 所有数据库查询都使用参数化查询
- [ ] 不使用字符串拼接SQL
- [ ] 正确使用ORM/查询构建器
- [ ] Supabase查询已正确清理

### 4. 认证与授权

#### JWT令牌处理
```typescript
// FAIL: 错误：localStorage（易受XSS攻击）
localStorage.setItem('token', token)

// PASS: 正确：httpOnly Cookie
res.setHeader('Set-Cookie',
  `token=${token}; HttpOnly; Secure; SameSite=Strict; Max-Age=3600`)
```

#### 授权检查
```typescript
export async function deleteUser(userId: string, requesterId: string) {
  // 始终首先检查授权
  const requester = await db.users.findUnique({
    where: { id: requesterId }
  })

  if (requester.role !== 'admin') {
    return NextResponse.json(
      { error: 'Unauthorized' },
      { status: 403 }
    )
  }

  // 继续删除
  await db.users.delete({ where: { id: userId } })
}
```

#### 行级安全（Supabase）
```sql
-- 对所有表启用RLS
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
- [ ] 令牌存储在httpOnly Cookie中（而非localStorage）
- [ ] 敏感操作前进行授权检查
- [ ] 在Supabase中启用Row Level Security
- [ ] 实现基于角色的访问控制
- [ ] 会话管理安全

### 5. XSS防护

#### 清理HTML
```typescript
import DOMPurify from 'isomorphic-dompurify'

// 始终清理用户提供的HTML
function renderUserContent(html: string) {
  const clean = DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'p'],
    ALLOWED_ATTR: []
  })
  return <div dangerouslySetInnerHTML={{ __html: clean }} />
}
```

#### 内容安全策略
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
- [ ] 清理用户提供的HTML
- [ ] 设置CSP头
- [ ] 不渲染未验证的动态内容
- [ ] 使用React内置的XSS防护

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

  // 处理请求
}
```

#### SameSite Cookie
```typescript
res.setHeader('Set-Cookie',
  `session=${sessionId}; HttpOnly; Secure; SameSite=Strict`)
```

#### 验证步骤
- [ ] 状态变更操作使用CSRF令牌
- [ ] 所有Cookie使用SameSite=Strict
- [ ] 实现双重提交Cookie模式

### 7. 速率限制

#### API速率限制
```typescript
import rateLimit from 'express-rate-limit'

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15分钟
  max: 100, // 每个窗口100个请求
  message: 'Too many requests'
})

// 应用于路由
app.use('/api/', limiter)
```

#### 高成本操作
```typescript
// 搜索的积极速率限制
const searchLimiter = rateLimit({
  windowMs: 60 * 1000, // 1分钟
  max: 10, // 每分钟10个请求
  message: 'Too many search requests'
})

app.use('/api/search', searchLimiter)
```

#### 验证步骤
- [ ] 所有API端点都有速率限制
- [ ] 高成本操作有更严格的限制
- [ ] 基于IP的速率限制
- [ ] 基于用户的速率限制（已认证）

### 8. 敏感数据泄露

#### 日志记录
```typescript
// FAIL: 错误：记录敏感数据
console.log('User login:', { email, password })
console.log('Payment:', { cardNumber, cvv })

// PASS: 正确：编辑敏感数据
console.log('User login:', { email, userId })
console.log('Payment:', { last4: card.last4, userId })
```

#### 错误消息
```typescript
// FAIL: 错误：泄露内部详情
catch (error) {
  return NextResponse.json(
    { error: error.message, stack: error.stack },
    { status: 500 }
  )
}

// PASS: 正确：通用错误消息
catch (error) {
  console.error('Internal error:', error)
  return NextResponse.json(
    { error: 'An error occurred. Please try again.' },
    { status: 500 }
  )
}
```

#### 验证步骤
- [ ] 日志中不包含密码、令牌、密钥
- [ ] 用户看到通用错误消息
- [ ] 详细错误仅记录在服务器日志中
- [ ] 不向用户暴露堆栈跟踪

### 9. 区块链安全（Solana）

#### 钱包验证
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

#### 交易验证
```typescript
async function verifyTransaction(transaction: Transaction) {
  // 验证接收方
  if (transaction.to !== expectedRecipient) {
    throw new Error('Invalid recipient')
  }

  // 验证金额
  if (transaction.amount > maxAmount) {
    throw new Error('Amount exceeds limit')
  }

  // 确认用户有足够余额
  const balance = await getBalance(transaction.from)
  if (balance < transaction.amount) {
    throw new Error('Insufficient balance')
  }

  return true
}
```

#### 验证步骤
- [ ] 验证钱包签名
- [ ] 验证交易详情
- [ ] 交易前检查余额
- [ ] 不进行盲交易签名

### 10. 依赖项安全

#### 定期更新
```bash
# 检查漏洞
npm audit

# 修复可自动修复的问题
npm audit fix

# 更新依赖项
npm update

# 检查过时的包
npm outdated
```

#### 锁定文件
```bash
# 始终提交锁定文件
git add package-lock.json

# 在CI/CD中用于可重现构建
npm ci  # 替代 npm install
```

#### 验证步骤
- [ ] 依赖项是最新的
- [ ] 无已知漏洞（npm audit通过）
- [ ] 提交锁定文件
- [ ] 在GitHub上启用Dependabot
- [ ] 定期安全更新

## 安全测试

### 自动安全测试
```typescript
// 测试认证
test('requires authentication', async () => {
  const response = await fetch('/api/protected')
  expect(response.status).toBe(401)
})

// 测试授权
test('requires admin role', async () => {
  const response = await fetch('/api/admin', {
    headers: { Authorization: `Bearer ${userToken}` }
  })
  expect(response.status).toBe(403)
})

// 测试输入验证
test('rejects invalid input', async () => {
  const response = await fetch('/api/users', {
    method: 'POST',
    body: JSON.stringify({ email: 'not-an-email' })
  })
  expect(response.status).toBe(400)
})

// 测试速率限制
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

在所有生产部署前：

- [ ] **密钥**：无硬编码密钥，全部使用环境变量
- [ ] **输入验证**：验证所有用户输入
- [ ] **SQL注入**：所有查询参数化
- [ ] **XSS**：清理用户内容
- [ ] **CSRF**：启用保护
- [ ] **认证**：适当的令牌处理
- [ ] **授权**：设置角色检查
- [ ] **速率限制**：在所有端点启用
- [ ] **HTTPS**：生产环境强制
- [ ] **安全头**：设置CSP、X-Frame-Options
- [ ] **错误处理**：错误中无敏感数据
- [ ] **日志记录**：日志中无敏感数据
- [ ] **依赖项**：最新、无漏洞
- [ ] **Row Level Security**：在Supabase启用
- [ ] **CORS**：正确配置
- [ ] **文件上传**：已验证（大小、类型）
- [ ] **钱包签名**：已验证（区块链场景）

## 资源

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Next.js Security](https://nextjs.org/docs/security)
- [Supabase Security](https://supabase.com/docs/guides/auth)
- [Web Security Academy](https://portswigger.net/web-security)

---

**请记住**：安全不是可选项。一个漏洞就可能危及整个平台。如有疑问，请谨慎行事。