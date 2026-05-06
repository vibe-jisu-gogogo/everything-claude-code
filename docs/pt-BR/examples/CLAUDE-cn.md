# 项目 CLAUDE.md 示例

这是项目级 CLAUDE.md 文件的示例。将其放在项目根目录。

## 项目概述

[项目的简要描述 - 功能、技术栈]

## 关键规则

### 1. 代码组织

- 多个小文件而非少数大文件
- 高内聚，低耦合
- 典型文件 200-400 行，最多 800 行
- 按功能/领域组织，而非按类型组织

### 2. 代码风格

- 代码、注释或文档中不使用表情符号
- 始终保持不可变性 - 永不修改对象或数组
- 生产代码中无 console.log
- 使用 try/catch 进行适当的错误处理
- 使用 Zod 或类似工具进行输入验证

### 3. 测试

- TDD：先写测试
- 最低 80% 覆盖率
- 工具函数使用单元测试
- API 使用集成测试
- 关键流程使用 E2E 测试

### 4. 安全

- 无硬编码密钥
- 敏感数据使用环境变量
- 验证所有用户输入
- 仅使用参数化查询
- 启用 CSRF 保护

## 文件结构

```
src/
|-- app/              # Next.js app router
|-- components/       # Reusable UI components
|-- hooks/            # Custom React hooks
|-- lib/              # Utility libraries
|-- types/            # TypeScript definitions
```

## 关键模式

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
# Required
DATABASE_URL=
API_KEY=

# Optional
DEBUG=false
```

## 可用命令

- `/tdd` - 测试驱动开发流程
- `/plan` - 创建实施计划
- `/code-review` - 代码质量审查
- `/build-fix` - 修复构建错误

## Git 流程

- Conventional commits: `feat:`, `fix:`, `refactor:`, `docs:`, `test:`
- 永不直接提交到 main
- PR 需要代码审查
- 合并前所有测试必须通过
