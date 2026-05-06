# Skill 开发指南

创建 Everything Claude Code (ECC) 有效 skill 的综合指南。

## 目录

- [什么是 Skill？](#什么是-skill)
- [Skill 架构](#skill-架构)
- [创建你的第一个 Skill](#创建你的第一个-skill)
- [Skill 分类](#skill-分类)
- [编写有效的 Skill 内容](#编写有效的-skill-内容)
- [最佳实践](#最佳实践)
- [常见模式](#常见模式)
- [测试你的 Skill](#测试你的-skill)
- [提交你的 Skill](#提交你的-skill)
- [示例展示](#示例展示)

---

## 什么是 Skill？

Skill 是 Claude Code 根据上下文加载的**知识模块**。它们提供：

- **领域专业知识**：框架模式、语言惯用法、最佳实践
- **工作流定义**：常见任务的分步流程
- **参考资料**：代码片段、检查清单、决策树
- **上下文注入**：在特定条件满足时激活

与 **agent**（专门的子助手）或 **command**（用户触发的动作）不同，skill 是被动知识，Claude Code 在相关时会引用它们。

### Skill 何时激活

Skill 在以下情况激活：
- 用户任务与 skill 的领域匹配
- Claude Code 检测到相关上下文
- 命令引用 skill
- Agent 需要领域知识

### Skill vs Agent vs Command

| 组件 | 目的 | 激活方式 |
|-----------|---------|------------|
| **Skill** | 知识仓库 | 基于上下文（自动） |
| **Agent** | 任务执行器 | 显式委派 |
| **Command** | 用户动作 | 用户调用（`/command`） |
| **Hook** | 自动化 | 事件触发 |
| **Rule** | 始终生效的指南 | 始终激活 |

---

## Skill 架构

### 文件结构

```
skills/
└── your-skill-name/
    ├── SKILL.md           # 必填：主 skill 定义
    ├── examples/          # 可选：代码示例
    │   ├── basic.ts
    │   └── advanced.ts
    └── references/        # 可选：外部参考
        └── links.md
```

### SKILL.md 格式

```markdown
---
name: skill-name
description: 在 skill 列表中显示的简要描述，用于自动激活
origin: ECC
---

# Skill 标题

本 skill 涵盖内容的简要概述。

## 何时激活

描述 Claude 应使用此 skill 的场景。

## 核心概念

主要模式和指南。

## 代码示例

```typescript
// 实用的、经过测试的示例
```

## 反模式

用具体示例说明不应该做什么。

## 最佳实践

- 可操作的指南
- 应该做和不应该做的事

## 相关 Skill

链接到互补的 skill。
```

### YAML Frontmatter 字段

| 字段 | 必填 | 描述 |
|-------|----------|-------------|
| `name` | 是 | 小写、连字符分隔的标识符（例如 `react-patterns`） |
| `description` | 是 | skill 列表和自动激活的单行描述 |
| `origin` | 否 | 来源标识符（例如 `ECC`、`community`、项目名称） |
| `tags` | 否 | 用于分类的标签数组 |
| `version` | 否 | 用于跟踪更新的 skill 版本 |

---

## 创建你的第一个 Skill

### 第 1 步：选择焦点

好的 skill 是**聚焦且可操作的**：

| 通过：好的焦点 | 失败：太宽泛 |
|---------------|--------------|
| `react-hook-patterns` | `react` |
| `postgresql-indexing` | `databases` |
| `pytest-fixtures` | `python-testing` |
| `nextjs-app-router` | `nextjs` |

### 第 2 步：创建目录

```bash
mkdir -p skills/your-skill-name
```

### 第 3 步：编写 SKILL.md

这是一个最小模板：

```markdown
---
name: your-skill-name
description: 何时使用此 skill 的简要描述
---

# 你的 Skill 标题

简要概述（1-2 句话）。

## 何时激活

- 场景 1
- 场景 2
- 场景 3

## 核心概念

### 概念 1

带示例的解释。

### 概念 2

另一个带代码的模式。

## 代码示例

```typescript
// 实用示例
```

## 最佳实践

- 做这个
- 避免那个

## 相关 Skill

- `related-skill-1`
- `related-skill-2`
```

### 第 4 步：添加内容

编写 Claude 可以**立即使用**的内容：

- 通过：可直接复制粘贴的代码示例
- 通过：清晰的决策树
- 通过：验证检查清单
- 失败：没有示例的模糊解释
- 失败：没有可操作指导的长篇散文

---

## Skill 分类

### 语言标准

专注于惯用代码、命名约定和特定语言模式。

**示例：** `python-patterns`、`golang-patterns`、`typescript-standards`

```markdown
---
name: python-patterns
description: Python 惯用法、最佳实践和整洁惯用代码的模式。
---

# Python 模式

## 何时激活

- 编写 Python 代码
- 重构 Python 模块
- Python 代码审查

## 核心概念

### 上下文管理器

```python
# 始终对资源使用上下文管理器
with open('file.txt') as f:
    content = f.read()
```
```

### 框架模式

专注于框架特定的约定、常见模式和反模式。

**示例：** `django-patterns`、`nextjs-patterns`、`springboot-patterns`

```markdown
---
name: django-patterns
description: Django 模型、视图、URL 和模板的最佳实践。
---

# Django 模式

## 何时激活

- 构建 Django 应用
- 创建模型和视图
- Django URL 配置
```

### 工作流 Skill

为常见开发任务定义分步流程。

**示例：** `tdd-workflow`、`code-review-workflow`、`deployment-checklist`

```markdown
---
name: code-review-workflow
description: 用于质量和安全的系统化代码审查流程。
---

# 代码审查工作流

## 步骤

1. **理解上下文** - 阅读 PR 描述和链接的 issue
2. **检查测试** - 验证测试覆盖率和质量
3. **审查逻辑** - 分析实现的正确性
4. **检查安全性** - 查找漏洞
5. **验证样式** - 确保代码遵循约定
```

### 领域知识

特定领域的专业知识（安全性、性能等）。

**示例：** `security-review`、`performance-optimization`、`api-design`

```markdown
---
name: api-design
description: REST 和 GraphQL API 设计模式、版本控制和最佳实践。
---

# API 设计模式

## RESTful 约定

| 方法 | 端点 | 用途 |
|--------|----------|---------|
| GET | /resources | 列出所有 |
| GET | /resources/:id | 获取单个 |
| POST | /resources | 创建 |
```

### 工具集成

使用特定工具、库或服务的指南。

**示例：** `supabase-patterns`、`docker-patterns`、`mcp-server-patterns`

---

## 编写有效的 Skill 内容

### 1. 从"何时激活"开始

这部分对自动激活至关重要。要具体：

```markdown
## 何时激活

- 创建新的 React 组件
- 重构现有组件
- 调试 React 状态问题
- 审查 React 代码的最佳实践
```

### 2. 使用"展示，而非讲述"

不好的：
```markdown
## 错误处理

始终在异步函数中正确处理错误。
```

好的：
```markdown
## 错误处理

```typescript
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
```

### 要点

- 解析前检查 `response.ok`
- 记录错误以便调试
- 抛出用户友好的消息
```

### 3. 包含反模式

展示不应该做什么：

```markdown
## 反模式

### 失败：直接状态突变

```typescript
// 永远不要这样做
user.name = 'New Name'
items.push(newItem)
```

### 通过：不可变更新

```typescript
// 始终这样做
const updatedUser = { ...user, name: 'New Name' }
const updatedItems = [...items, newItem]
```
```

### 4. 提供检查清单

检查清单是可操作且易于遵循的：

```markdown
## 部署前检查清单

- [ ] 所有测试通过
- [ ] 生产代码中没有 console.log
- [ ] 环境变量已记录
- [ ] 秘密未硬编码
- [ ] 错误处理完成
- [ ] 输入验证到位
```

### 5. 使用决策树

对于复杂决策：

```markdown
## 选择正确的方法

```
需要获取数据？
├── 单个请求 → 直接使用 fetch
├── 多个独立请求 → Promise.all()
├── 多个依赖请求 → 顺序 await
└── 需要缓存 → 使用 SWR 或 React Query
```
```

---

## 最佳实践

### 应该做

| 实践 | 示例 |
|----------|---------|
| **具体** | "对传递给子组件的事件处理程序使用 `useCallback`" |
| **展示示例** | 包含可直接复制粘贴的代码 |
| **解释原因** | "不可变性防止 React 状态中意外的副作用" |
| **链接相关 skill** | "另见：`react-performance`" |
| **保持聚焦** | 一个 skill = 一个领域/概念 |
| **使用章节** | 清晰的标题便于快速浏览 |

### 不应该做

| 实践 | 为什么不好 |
|----------|--------------|
| **模糊** | "写好代码" - 不可操作 |
| **长篇大论** | 难以解析，不如代码好 |
| **涵盖过多** | "Python、Django 和 Flask 模式" - 太宽泛 |
| **跳过示例** | 没有实践的理论用处不大 |
| **忽略反模式** | 学习不应该做什么很有价值 |

### 内容指南

1. **长度**：典型 200-500 行，最多 800 行
2. **代码块**：包含语言标识符
3. **标题**：使用 `##` 和 `###` 层级
4. **列表**：无序列表使用 `-`，有序列表使用 `1.`
5. **表格**：用于比较和参考

---

## 常见模式

### 模式 1：标准 Skill

```markdown
---
name: language-standards
description: [语言] 的编码标准和最佳实践。
---

# [语言] 编码标准

## 何时激活

- 编写 [语言] 代码
- 代码审查
- 设置 linting

## 命名约定

| 元素 | 约定 | 示例 |
|---------|------------|---------|
| 变量 | camelCase | userName |
| 常量 | SCREAMING_SNAKE | MAX_RETRY |
| 函数 | camelCase | fetchUser |
| 类 | PascalCase | UserService |

## 代码示例

[包含实用示例]

## Linting 设置

[包含配置]

## 相关 Skill

- `language-testing`
- `language-security`
```

### 模式 2：工作流 Skill

```markdown
---
name: task-workflow
description: [任务] 的分步工作流。
---

# [任务] 工作流

## 何时激活

- [触发器 1]
- [触发器 2]

## 前提条件

- [要求 1]
- [要求 2]

## 步骤

### 第 1 步：[名称]

[描述]

```bash
[命令]
```

### 第 2 步：[名称]

[描述]

## 验证

- [ ] [检查 1]
- [ ] [检查 2]

## 故障排除

| 问题 | 解决方案 |
|---------|----------|
| [问题] | [修复] |
```

### 模式 3：参考 Skill

```markdown
---
name: api-reference
description: [API/库] 的快速参考。
---

# [API/库] 参考

## 何时激活

- 使用 [API/库]
- 查找 [API/库] 语法

## 常见操作

### 操作 1

```typescript
// 基本用法
```

### 操作 2

```typescript
// 高级用法
```

## 配置

[包含配置示例]

## 错误处理

[包含错误模式]
```

---

## 测试你的 Skill

### 本地测试

1. **复制到 Claude Code skill 目录**：
   ```bash
   cp -r skills/your-skill-name ~/.claude/skills/
   ```

2. **用 Claude Code 测试**：
   ```
   你："我需要 [应该触发你 skill 的任务]"

   Claude 应该引用你 skill 的模式。
   ```

3. **验证激活**：
   - 让 Claude 解释你 skill 中的一个概念
   - 检查它是否使用你的示例和模式
   - 确保它遵循你的指南

### 验证检查清单

- [ ] **YAML frontmatter 有效** - 无语法错误
- [ ] **名称遵循约定** - 小写连字符分隔
- [ ] **描述清晰** - 说明何时使用
- [ ] **示例有效** - 代码可以编译和运行
- [ ] **链接有效** - 相关 skill 存在
- [ ] **无敏感数据** - 无 API 密钥、令牌、路径

### 代码示例测试

测试所有代码示例：

```bash
# 从仓库根目录
npx tsc --noEmit skills/your-skill-name/examples/*.ts

# 或从 skill 目录内
npx tsc --noEmit examples/*.ts

# 从仓库根目录
python -m py_compile skills/your-skill-name/examples/*.py

# 或从 skill 目录内
python -m py_compile examples/*.py

# 从仓库根目录
go build ./skills/your-skill-name/examples/...

# 或从 skill 目录内
go build ./examples/...
```

---

## 提交你的 Skill

### 1. Fork 和克隆

```bash
gh repo fork affaan-m/everything-claude-code --clone
cd everything-claude-code
```

### 2. 创建分支

```bash
git checkout -b feat/skill-your-skill-name
```

### 3. 添加你的 Skill

```bash
mkdir -p skills/your-skill-name
# 创建 SKILL.md
```

### 4. 验证

```bash
# 检查 YAML frontmatter
head -10 skills/your-skill-name/SKILL.md

# 验证结构
ls -la skills/your-skill-name/

# 如果可用则运行测试
npm test
```

### 5. 提交并推送

```bash
git add skills/your-skill-name/
git commit -m "feat(skills): add your-skill-name skill"
git push -u origin feat/skill-your-skill-name
```

### 6. 创建 Pull Request

使用此 PR 模板：

```markdown
## 摘要

skill 的简要描述及其价值。

## Skill 类型

- [ ] 语言标准
- [ ] 框架模式
- [ ] 工作流
- [ ] 领域知识
- [ ] 工具集成

## 测试

我如何在本地测试此 skill。

## 检查清单

- [ ] YAML frontmatter 有效
- [ ] 代码示例已测试
- [ ] 遵循 skill 指南
- [ ] 无敏感数据
- [ ] 清晰的激活触发器
```

---

## 示例展示

### 示例 1：语言标准

**文件：** `skills/rust-patterns/SKILL.md

```markdown
---
name: rust-patterns
description: Rust 惯用法、所有权模式和安全惯用代码的最佳实践。
origin: ECC
---

# Rust 模式

## 何时激活

- 编写 Rust 代码
- 处理所有权和借用
- 使用 Result/Option 处理错误
- 实现 trait

## 所有权模式

### 借用规则

```rust
// 通过：正确：不需要所有权时借用
fn process_data(data: &str) -> usize {
    data.len()
}

// 通过：正确：需要修改或消耗时获取所有权
fn consume_data(data: Vec<u8>) -> String {
    String::from_utf8(data).unwrap()
}
```

## 错误处理

### Result 模式

```rust
use thiserror::Error;

#[derive(Error, Debug)]
pub enum AppError {
    #[error("IO error: {0}")]
    Io(#[from] std::io::Error),

    #[error("Parse error: {0}")]
    Parse(#[from] std::num::ParseIntError),
}

pub type AppResult<T> = Result<T, AppError>;
```

## 相关 Skill

- `rust-testing`
- `rust-security`
```

### 示例 2：框架模式

**文件：** `skills/fastapi-patterns/SKILL.md

```markdown
---
name: fastapi-patterns
description: FastAPI 路由、依赖注入、验证和异步操作的模式。
origin: ECC
---

# FastAPI 模式

## 何时激活

- 构建 FastAPI 应用
- 创建 API 端点
- 实现依赖注入
- 处理异步数据库操作

## 项目结构

```
app/
├── main.py              # FastAPI 应用入口
├── routers/             # 路由处理器
│   ├── users.py
│   └── items.py
├── models/              # Pydantic 模型
│   ├── user.py
│   └── item.py
├── services/            # 业务逻辑
│   └── user_service.py
└── dependencies.py      # 共享依赖
```

## 依赖注入

```python
from fastapi import Depends
from sqlalchemy.ext.asyncio import AsyncSession

async def get_db() -> AsyncSession:
    async with AsyncSessionLocal() as session:
        yield session

@router.get("/users/{user_id}")
async def get_user(
    user_id: int,
    db: AsyncSession = Depends(get_db)
):
    # 使用 db 会话
    pass
```

## 相关 Skill

- `python-patterns`
- `pydantic-validation`
```

### 示例 3：工作流 Skill

**文件：** `skills/refactoring-workflow/SKILL.md

```markdown
---
name: refactoring-workflow
description: 在不改变行为的情况下提高代码质量的系统化重构工作流。
origin: ECC
---

# 重构工作流

## 何时激活

- 改进代码结构
- 减少技术债务
- 简化复杂代码
- 提取可复用组件

## 前提条件

- 所有测试通过
- Git 工作目录干净
- 已创建功能分支

## 工作流步骤

### 第 1 步：识别重构目标

- 查找代码坏味道（长方法、重复代码、大型类）
- 检查目标区域的测试覆盖率
- 记录当前行为

### 第 2 步：确保测试存在

```bash
# 运行测试以验证当前行为
npm test

# 检查目标文件的覆盖率
npm run test:coverage
```

### 第 3 步：进行小改动

- 一次一个重构
- 每次更改后运行测试
- 频繁提交

### 第 4 步：验证行为未改变

```bash
# 运行完整测试套件
npm test

# 运行 E2E 测试
npm run test:e2e
```

## 常见重构

| 坏味道 | 重构 |
|-------|-------------|
| 长方法 | 提取方法 |
| 重复代码 | 提取到共享函数 |
| 大型类 | 提取类 |
| 长参数列表 | 引入参数对象 |

## 检查清单

- [ ] 目标代码有测试
- [ ] 进行了小的、聚焦的更改
- [ ] 每次更改后测试通过
- [ ] 行为未改变
- [ ] 提交信息清晰
```

---

## 其他资源

- [CONTRIBUTING.md](../CONTRIBUTING.md) - 通用贡献指南
- [project-guidelines-template](./examples/project-guidelines-template.md) - 项目特定 skill 模板
- [coding-standards](../skills/coding-standards/SKILL.md) - 标准 skill 示例
- [tdd-workflow](../skills/tdd-workflow/SKILL.md) - 工作流 skill 示例
- [security-review](../skills/security-review/SKILL.md) - 领域知识 skill 示例

---

**记住**：好的 skill 是聚焦的、可操作的，并且立即可用。编写你自己想要使用的 skill。
