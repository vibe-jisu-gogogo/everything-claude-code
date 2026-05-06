# SaaS 应用 — 项目 CLAUDE.md

> 使用 Next.js + Supabase + Stripe 构建 SaaS 应用的真实示例。
> 复制到项目根目录并根据你的技术栈进行自定义。

## 项目概述

**技术栈:** Next.js 15 (App Router), TypeScript, Supabase (auth + DB), Stripe (billing), Tailwind CSS, Playwright (E2E)

**架构:** 默认使用 Server Components。仅在需要交互时使用 Client Components。API routes 用于 webhooks，server actions 用于数据变更操作。

## 关键规则

### 数据库

- 所有查询使用启用 RLS 的 Supabase 客户端 — 永远不要绕过 RLS
- 迁移文件位于 `supabase/migrations/` — 永远不要直接修改数据库
- 使用带有明确列名列表的 `select()`，不要使用 `select('*')`
- 所有面向用户的查询必须包含 `.limit()` 以避免无限制的结果返回

### 认证

- 在 Server Components 中使用 `@supabase/ssr` 的 `createServerClient()`
- 在 Client Components 中使用 `@supabase/ssr` 的 `createBrowserClient()`
- 受保护的路由检查 `getUser()` — 永远不要只依赖 `getSession()` 进行认证
- `middleware.ts` 中的中间件在每次请求中刷新认证令牌

### Billing

- Stripe webhook 处理器位于 `app/api/webhooks/stripe/route.ts`
- 永远不要信任客户端的价格 — 始终在服务端从 Stripe 获取
- 订阅状态通过 `subscription_status` 列检查，由 webhook 同步
- 免费层用户：3 个项目，每天 100 次 API 调用

### 代码风格

- 代码或注释中不使用 emoji
- 仅使用不可变模式 — 使用 spread operator，永远不要直接修改
- Server Components：没有 `'use client'` 指令，没有 `useState`/`useEffect`
- Client Components：顶部添加 `'use client'`，尽量少用 — 将逻辑提取到 hooks 中
- 所有输入验证优先使用 Zod schemas（API routes、表单、环境变量）

## 文件结构

```
src/
  app/
    (auth)/          # Auth 页面 (login, signup, forgot-password)
    (dashboard)/     # 受保护的 dashboard 页面
    api/
      webhooks/      # Stripe, Supabase webhooks
    layout.tsx       # 带有 providers 的根布局
  components/
    ui/              # Shadcn/ui 组件
    forms/           # 带验证的表单组件
    dashboard/       # Dashboard 专用组件
  hooks/             # 自定义 React hooks
  lib/
    supabase/        # Supabase 客户端工厂
    stripe/          # Stripe 客户端和辅助函数
    utils.ts         # 通用工具函数
  types/             # 共享 TypeScript 类型
supabase/
  migrations/        # 数据库迁移
  seed.sql           # 开发环境种子数据
```

## 关键模式

### API 响应格式

```typescript
type ApiResponse<T> =
  | { success: true; data: T }
  | { success: false; error: string; code?: string }
```

### Server Action 模式

```typescript
'use server'

import { z } from 'zod'
import { createServerClient } from '@/lib/supabase/server'

const schema = z.object({
  name: z.string().min(1).max(100),
})

export async function createProject(formData: FormData) {
  const parsed = schema.safeParse({ name: formData.get('name') })
  if (!parsed.success) {
    return { success: false, error: parsed.error.flatten() }
  }

  const supabase = await createServerClient()
  const { data: { user } } = await supabase.auth.getUser()
  if (!user) return { success: false, error: 'Unauthorized' }

  const { data, error } = await supabase
    .from('projects')
    .insert({ name: parsed.data.name, user_id: user.id })
    .select('id, name, created_at')
    .single()

  if (error) return { success: false, error: 'Failed to create project' }
  return { success: true, data }
}
```

## 环境变量

```bash
# Supabase
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=     # 仅服务端使用，永不暴露给客户端

# Stripe
STRIPE_SECRET_KEY=
STRIPE_WEBHOOK_SECRET=
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=

# App
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## 测试策略

```bash
/tdd                    # 新功能的单元 + 集成测试
/e2e                    # Playwright 测试认证流程、billing、dashboard
/test-coverage          # 验证 80%+ 覆盖率
```

### 关键 E2E 流程

1. 注册 → 邮箱验证 → 创建第一个项目
2. 登录 → dashboard → CRUD 操作
3. 套餐升级 → Stripe checkout → 订阅激活
4. Webhook：订阅取消 → 降级到免费层

## ECC 工作流程

```bash
# 规划功能
/plan "Add team invitations with email notifications"

# 使用 TDD 开发
/tdd

# 提交前
/code-review
/security-scan

# 发布前
/e2e
/test-coverage
```

## Git 流程

- `feat:` 新功能，`fix:` Bug 修复，`refactor:` 代码变更
- 功能分支从 `main` 创建，必须通过 PR 合并
- CI 运行：lint、type-check、单元测试、E2E 测试
- 部署：PR 时 Vercel 预览部署，合并到 `main` 时生产部署
