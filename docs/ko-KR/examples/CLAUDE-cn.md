# 项目 CLAUDE.md 示例

项目级 CLAUDE.md 文件示例。请放置在项目根目录。

## 项目概述

[项目的简短描述 - 功能、技术栈]

## 核心规则

### 1. 代码结构

- 优先使用多个小文件，而不是少数大文件
- 高内聚、低耦合
- 通常 200-400 行，单文件最多 800 行
- 按功能/领域组织，而非按类型组织

### 2. 代码风格

- 代码、注释、文档中禁止使用 emoji
- 始终保持不可变性 - 不直接修改对象或数组
- 生产代码中禁止使用 console.log
- 使用 try/catch 进行适当的错误处理
- 使用 Zod 或类似库进行输入验证

### 3. 测试

- TDD：先编写测试
- 最低 80% 覆盖率
- 工具函数的单元测试
- API 的集成测试
- 核心流程的 E2E 测试

### 4. 安全

- 禁止硬编码密钥
- 敏感数据使用环境变量
- 所有用户输入必须验证
- 仅使用参数化查询
- 启用 CSRF 保护

## 文件结构

```
src/
|-- app/              # Next.js app router
|-- components/       # 可复用的 UI 组件
|-- hooks/            # 自定义 React hooks
|-- lib/              # 工具库
|-- types/            # TypeScript 类型定义
```

## 主要模式

### API 响应格式

```typescript
interface ApiResponse<T> {
  success: boolean
  data?: T
  error?: string
}
```

### 错误处理

```typescript
try {
  const result = await operation()
  return { success: true, data: result }
} catch (error) {
  console.error('Operation failed:', error)
  return { success: false, error: 'User-friendly message' }
}
```

## 环境变量

```bash
# 必需
DATABASE_URL=
API_KEY=

# 可选
DEBUG=false
```

## 可用命令

- `/tdd` - 测试驱动开发工作流
- `/plan` - 创建实现计划
- `/code-review` - 代码质量审查
- `/build-fix` - 修复构建错误

## Git 工作流

- Conventional commits: `feat:`, `fix:`, `refactor:`, `docs:`, `test:`
- 禁止直接提交到 main 分支
- PR 必须经过审查
- 合并前必须通过所有测试
