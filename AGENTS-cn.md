# Everything Claude Code (ECC) — 智能体使用说明

这是一个**production-ready AI coding plugin**，为软件开发提供48个专业智能体、182项技能、68条命令以及自动化hook工作流。

**版本:** 2.0.0-rc.1

## 核心原则

1. **Agent-First** — 领域任务优先委托给专业智能体处理
2. **Test-Driven** — 实现代码前先编写测试，要求覆盖率达到80%以上
3. **Security-First** — 永远不牺牲安全性，验证所有输入
4. **Immutability** — 始终创建新对象，永远不修改现有对象
5. **Plan Before Execute** — 编写代码前先规划复杂功能

## 可用智能体

| 智能体 | 用途 | 使用场景 |
|-------|---------|-------------|
| planner | 实现方案规划 | 复杂功能、重构 |
| architect | 系统设计与可扩展性 | 架构决策 |
| tdd-guide | 测试驱动开发 | 新功能、bug修复 |
| code-reviewer | 代码质量与可维护性审查 | 编写/修改代码后 |
| security-reviewer | 漏洞检测 | 提交前、敏感代码审查 |
| build-error-resolver | 构建/类型错误修复 | 构建失败时 |
| e2e-runner | Playwright端到端测试 | 核心用户流程 |
| refactor-cleaner | 死代码清理 | 代码维护 |
| doc-updater | 文档与代码映射更新 | 更新文档时 |
| cpp-reviewer | C/C++代码审查 | C和C++项目 |
| cpp-build-resolver | C/C++构建错误修复 | C和C++构建失败时 |
| docs-lookup | 基于Context7的文档查询 | API/文档相关问题 |
| go-reviewer | Go代码审查 | Go项目 |
| go-build-resolver | Go构建错误修复 | Go构建失败时 |
| kotlin-reviewer | Kotlin代码审查 | Kotlin/Android/KMP项目 |
| kotlin-build-resolver | Kotlin/Gradle构建错误修复 | Kotlin构建失败时 |
| database-reviewer | PostgreSQL/Supabase专家 | 表结构设计、查询优化 |
| python-reviewer | Python代码审查 | Python项目 |
| java-reviewer | Java和Spring Boot代码审查 | Java/Spring Boot项目 |
| java-build-resolver | Java/Maven/Gradle构建错误修复 | Java构建失败时 |
| loop-operator | 自动化循环执行 | 安全运行循环、监控停滞、介入处理 |
| harness-optimizer | Harness配置调优 | 可靠性、成本、吞吐量优化 |
| rust-reviewer | Rust代码审查 | Rust项目 |
| rust-build-resolver | Rust构建错误修复 | Rust构建失败时 |
| pytorch-build-resolver | PyTorch运行时/CUDA/训练错误修复 | PyTorch构建/训练失败时 |
| typescript-reviewer | TypeScript/JavaScript代码审查 | TypeScript/JavaScript项目 |

## 智能体编排

无需用户提示即可主动使用智能体：
- 复杂功能需求 → **planner**
- 刚编写/修改的代码 → **code-reviewer**
- Bug修复或新功能 → **tdd-guide**
- 架构决策 → **architect**
- 安全敏感代码 → **security-reviewer**
- 自动化循环 / 循环监控 → **loop-operator**
- Harness配置可靠性与成本优化 → **harness-optimizer**

独立操作使用并行执行 — 同时启动多个智能体。

## 安全指南

**任何提交前：**
- 无硬编码secrets（API keys、密码、tokens）
- 所有用户输入已验证
- SQL注入预防（参数化查询）
- XSS预防（HTML内容已清理）
- CSRF保护已启用
- 身份验证/授权已验证
- 所有接口已配置限流
- 错误信息不泄露敏感数据

**Secret管理：** 永远不要硬编码secrets。使用环境变量或secret管理器。启动时验证所需secrets。立即轮换任何泄露的secrets。

**如果发现安全问题：** 停止操作 → 使用security-reviewer智能体 → 修复CRITICAL级别的问题 → 轮换泄露的secrets → 检查代码库中是否存在类似问题。

## 编码风格

**Immutability (CRITICAL)：** 始终创建新对象，永远不修改。返回应用了变更的新副本。

**文件组织：** 多个小文件优于少量大文件。典型大小200-400行，最大800行。按功能/领域组织，而非按类型。高内聚，低耦合。

**错误处理：** 在每个层级处理错误。UI代码提供用户友好的提示。服务端记录详细上下文。永远不要静默忽略错误。

**输入验证：** 在系统边界验证所有用户输入。使用schema-based验证。快速失败并返回清晰信息。永远不要信任外部数据。

**代码质量检查清单：**
- 函数短小（<50行），文件职责单一（<800行）
- 无深层嵌套（>4层）
- 适当的错误处理，无硬编码值
- 标识符可读性强、命名规范

## 测试要求

**最低覆盖率：80%**

测试类型（全部要求）：
1. **Unit tests** — 独立函数、工具类、组件
2. **Integration tests** — API endpoints、数据库操作
3. **E2E tests** — 核心用户流程

**TDD workflow (mandatory)：**
1. 写测试先（RED） — 测试应该失败
2. 编写最简化实现（GREEN） — 测试应该通过
3. 重构（IMPROVE） — 验证覆盖率达到80%以上

故障排查：检查测试隔离性 → 验证mocks → 修复实现（而非测试，除非测试本身错误）。

## 开发工作流

1. **Plan** — 使用planner智能体，识别依赖和风险，拆分为多个阶段
2. **TDD** — 使用tdd-guide智能体，先写测试，再实现，最后重构
3. **Review** — 立即使用code-reviewer智能体，处理CRITICAL/HIGH级别的问题
4. **在正确的位置沉淀知识**
   - 个人调试笔记、偏好和临时上下文 → auto memory
   - 团队/项目知识（架构决策、API变更、运行手册） → 项目现有文档结构
   - 如果当前任务已经生成了相关文档或代码注释，不要在其他位置重复相同信息
   - 如果没有明确的项目文档存放位置，创建新顶级文件前先询问
5. **Commit** — Conventional commits格式，PR摘要内容全面

## 工作流展示策略

- `skills/` 是标准工作流展示目录
- 新的工作流贡献应该首先提交到`skills/`
- `commands/` 是遗留斜杠命令兼容层，只有当需要迁移或跨harness兼容的垫片时才应新增或更新

## Git工作流

**Commit format:** `<type>: <description>` — Types: feat, fix, refactor, docs, test, chore, perf, ci

**PR workflow:** 分析完整commit历史 → 编写全面的摘要 → 包含测试计划 → 使用`-u`标志推送

## 架构模式

**API response format:** 统一的封装结构，包含成功标识、数据负载、错误信息和分页元数据。

**Repository pattern:** 将数据访问封装在标准接口（findAll, findById, create, update, delete）之后。业务逻辑依赖抽象接口，而非具体存储机制。

**Skeleton projects:** 搜索经过实战验证的模板，使用并行智能体评估（安全性、可扩展性、相关性），克隆最佳匹配项，在经过验证的结构内迭代。

## 性能

**Context management:** 大型重构和多文件功能避免使用context window最后20%的空间。低敏感度任务（单次编辑、文档、简单修复）可以容忍更高的使用率。

**Build troubleshooting:** 使用build-error-resolver智能体 → 分析错误 → 增量修复 → 每次修复后验证。

## 项目结构

```
agents/          — 48个专业子智能体
skills/          — 182个工作流技能和领域知识
commands/        — 68个斜杠命令
hooks/           — 基于触发器的自动化
rules/           — 必须遵守的指南（通用 + 特定语言）
scripts/         — 跨平台Node.js工具
mcp-configs/     — 14个MCP服务器配置
tests/           — 测试套件
```

`commands/` 保留在仓库中用于兼容，但长期方向是skills-first。

## 成功指标

- 所有测试通过，覆盖率达到80%以上
- 无安全漏洞
- 代码可读性和可维护性良好
- 性能符合要求
- 满足用户需求
