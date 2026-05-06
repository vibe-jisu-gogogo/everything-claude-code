---
name: configure-ecc
description: Everything Claude Code 的交互式安装程序 — 指导您在用户级别或项目级别目录中选择和安装技能与规则，验证路径，并根据需要优化已安装的文件。
---

# Configure Everything Claude Code (ECC)

Everything Claude Code 项目的交互式分步安装向导。使用 `AskUserQuestion` 指导用户选择性安装技能和规则，验证准确性，并提供优化建议。

## 启动时机

- 用户说 "configure ecc"、"install ecc"、"setup everything claude code" 等时
- 用户想要从该项目选择性安装技能或规则时
- 用户想要验证或修复现有 ECC 安装时
- 用户想要为项目优化已安装的技能或规则时

## 前提条件

启动此技能前必须可以从 Claude Code 访问。有两种引导方式：
1. **通过插件**: `/plugin install everything-claude-code@everything-claude-code` — 插件会自动加载此技能
2. **手动**: 仅将此技能复制到 `~/.claude/skills/configure-ecc/SKILL.md`，然后说 "configure ecc" 启动

---

## 步骤 0: 克隆 ECC 仓库

安装前，将最新的 ECC 源代码克隆到 `/tmp`：

```bash
rm -rf /tmp/everything-claude-code
git clone https://github.com/affaan-m/everything-claude-code.git /tmp/everything-claude-code
```

设置 `ECC_ROOT=/tmp/everything-claude-code` 作为后续所有复制操作的源。

如果克隆失败（如网络问题），使用 `AskUserQuestion` 要求用户提供现有 ECC 克隆的本地路径。

---

## 步骤 1: 选择安装级别

使用 `AskUserQuestion` 询问用户安装位置：

```
Question: "ECC 组件要安装到哪里？"
Options:
  - "User-level (~/.claude/)" — "应用于所有 Claude Code 项目"
  - "Project-level (.claude/)" — "仅应用于当前项目"
  - "Both" — "公共/共享项目使用用户级别，项目专用项目使用项目级别"
```

将选择保存为 `INSTALL_LEVEL`。设置目标目录：
- User-level: `TARGET=~/.claude`
- Project-level: `TARGET=.claude`（相对于当前项目根目录）
- Both: `TARGET_USER=~/.claude`、`TARGET_PROJECT=.claude`

目标目录不存在时创建：
```bash
mkdir -p $TARGET/skills $TARGET/rules
```

---

## 步骤 2: 选择并安装技能

### 2a: 选择技能类别

27 个技能分为 4 个类别。使用 `multiSelect: true` 的 `AskUserQuestion`：

```
Question: "要安装哪些技能类别？"
Options:
  - "Framework & Language" — "Django, Spring Boot, Go, Python, Java, Frontend, Backend 模式"
  - "Database" — "PostgreSQL, ClickHouse, JPA/Hibernate 模式"
  - "Workflow & Quality" — "TDD, 验证, 学习, 安全审查, 压缩"
  - "All skills" — "安装所有可用技能"
```

### 2b: 确认个别技能

对于每个选定的类别，显示以下完整的技能列表，并要求用户确认或取消选择特定项目。如果列表超过 4 项，将列表显示为文本，并在 `AskUserQuestion` 中使用"安装所有列出的项目"选项和用户粘贴特定名称的"其他"选项。

**类别: Framework & Language（16 技能）**

| 技能 | 说明 |
|-------|-------------|
| `backend-patterns` | 后端架构、API 设计、Node.js/Express/Next.js 服务端最佳实践 |
| `coding-standards` | TypeScript、JavaScript、React、Node.js 的通用编码标准 |
| `django-patterns` | Django 架构、使用 DRF 的 REST API、ORM、缓存、信号、中间件 |
| `django-security` | Django 安全性：认证、CSRF、SQL 注入、XSS 防护 |
| `django-tdd` | 使用 pytest-django、factory_boy、模拟、覆盖率进行 Django 测试 |
| `django-verification` | Django 验证循环：迁移、linting、测试、安全扫描 |
| `frontend-patterns` | React、Next.js、状态管理、性能、UI 模式 |
| `golang-patterns` | 惯用的 Go 模式、构建健壮 Go 应用的约定 |
| `golang-testing` | Go 测试：表驱动测试、子测试、基准测试、模糊测试 |
| `java-coding-standards` | Spring Boot 用 Java 编码标准：命名、不变性、Optional、流 |
| `python-patterns` | Pythonic 习惯用法、PEP 8、类型提示、最佳实践 |
| `python-testing` | 使用 pytest、TDD、fixtures、模拟、参数化进行 Python 测试 |
| `springboot-patterns` | Spring Boot 架构、REST API、分层服务、缓存、异步 |
| `springboot-security` | Spring Security：认证/授权、验证、CSRF、密钥、速率限制 |
| `springboot-tdd` | 使用 JUnit 5、Mockito、MockMvc、Testcontainers 进行 Spring Boot TDD |
| `springboot-verification` | Spring Boot 验证：构建、静态分析、测试、安全扫描 |

**类别: Database（3 技能）**

| 技能 | 说明 |
|-------|-------------|
| `clickhouse-io` | ClickHouse 模式、查询优化、分析、数据工程 |
| `jpa-patterns` | JPA/Hibernate 实体设计、关系、查询优化、事务 |
| `postgres-patterns` | PostgreSQL 查询优化、架构设计、索引创建、安全性 |

**类别: Workflow & Quality（8 技能）**

| 技能 | 说明 |
|-------|-------------|
| `continuous-learning` | 从会话中自动提取可重用模式作为已学习技能 |
| `continuous-learning-v2` | 具有置信度评分的本能学习，进化为技能/命令/代理 |
| `eval-harness` | 用于评估驱动开发（EDD）的正式评估框架 |
| `iterative-retrieval` | 解决子代理上下文问题的渐进式上下文改进 |
| `security-review` | 安全检查清单：认证、输入、密钥、API、支付功能 |
| `strategic-compact` | 在逻辑间隔建议手动上下文压缩 |
| `tdd-workflow` | 强制 TDD 实现 80%+ 覆盖率：单元、集成、E2E |
| `verification-loop` | 验证和质量循环模式 |

**独立**

| 技能 | 说明 |
|-------|-------------|
| `docs/examples/project-guidelines-template.md` | 创建项目特定技能的模板 |

### 2c: 执行安装

对于每个选定的技能，从正确的源根复制整个技能目录：

```bash
# 核心技能位于 .agents/skills/ 下
cp -R "$ECC_ROOT/.agents/skills/<skill-name>" "$TARGET/skills/"

# 细分技能位于 skills/ 下
cp -R "$ECC_ROOT/skills/<skill-name>" "$TARGET/skills/"
```

处理 glob 获取的源目录时，不要直接将带 trailing slash 的源传递给 `cp`。在目标名称中明确指定目录名：

```bash
cp -R "${src%/}" "$TARGET/skills/$(basename "${src%/}")"
```

注意: `continuous-learning` 和 `continuous-learning-v2` 有附加文件（config.json、hooks、脚本）— 确保不仅复制 SKILL.md，还复制整个目录。

---

## 步骤 3: 选择并安装规则

使用 `multiSelect: true` 的 `AskUserQuestion`：

```
Question: "要安装哪些规则集？"
Options:
  - "Common rules (Recommended)" — "语言无关原则：编码风格、git 工作流、测试、安全等（8 个文件）"
  - "TypeScript/JavaScript" — "TS/JS 模式、hooks、使用 Playwright 测试（5 个文件）"
  - "Python" — "Python 模式、pytest、black/ruff 格式化（5 个文件）"
  - "Go" — "Go 模式、表驱动测试、gofmt/staticcheck（5 个文件）"
```

执行安装：
```bash
# 公共规则（扁平复制到 rules/）
cp -r $ECC_ROOT/rules/common/* $TARGET/rules/

# 语言特定规则（扁平复制到 rules/）
cp -r $ECC_ROOT/rules/typescript/* $TARGET/rules/   # 如果已选择
cp -r $ECC_ROOT/rules/python/* $TARGET/rules/        # 如果已选择
cp -r $ECC_ROOT/rules/golang/* $TARGET/rules/        # 如果已选择
```

**重要**: 如果用户选择了语言特定规则但未选择公共规则，发出警告：
> "语言特定规则扩展了公共规则。不安装公共规则可能导致覆盖不完整。是否也安装公共规则？"

---

## 步骤 4: 安装后验证

安装后，执行以下自动检查：

### 4a: 文件存在确认

列出所有已安装的文件，确认它们存在于目标位置：
```bash
ls -la $TARGET/skills/
ls -la $TARGET/rules/
```

### 4b: 检查路径引用

扫描所有已安装的 `.md` 文件中的路径引用：
```bash
grep -rn "~/.claude/" $TARGET/skills/ $TARGET/rules/
grep -rn "../common/" $TARGET/rules/
grep -rn "skills/" $TARGET/skills/
```

**对于项目级别安装**，标记对 `~/.claude/` 路径的引用：
- 如果技能引用 `~/.claude/settings.json` — 这通常没问题（设置始终在用户级别）
- 如果技能引用 `~/.claude/skills/` 或 `~/.claude/rules/` — 如果仅安装在项目级别，这可能已损坏
- 如果技能通过名称引用另一个技能 — 确认被引用的技能也已安装

### 4c: 检查技能间相互引用

某些技能引用其他技能。验证这些依赖关系：
- `django-tdd` 可能引用 `django-patterns`
- `springboot-tdd` 可能引用 `springboot-patterns`
- `continuous-learning-v2` 引用 `~/.claude/homunculus/` 目录
- `python-testing` 可能引用 `python-patterns`
- `golang-testing` 可能引用 `golang-patterns`
- 语言特定规则引用 `common/` 中的对应项

### 4d: 报告问题

对于发现的每个问题，报告：
1. **文件**: 包含有问题引用的文件
2. **行**: 行号
3. **问题**: 什么出错了（例如："引用了 ~/.claude/skills/python-patterns，但 python-patterns 未安装"）
4. **推荐修复**: 应该做什么（例如："安装 python-patterns 技能" 或 "将路径更新为 .claude/skills/"）

---

## 步骤 5: 优化已安装文件（可选）

使用 `AskUserQuestion`：

```
Question: "是否要为项目优化已安装的文件？"
Options:
  - "Optimize skills" — "删除无关部分、调整路径、根据技术栈调整"
  - "Optimize rules" — "调整覆盖率目标、添加项目特定模式、自定义工具设置"
  - "Optimize both" — "对所有已安装文件进行完全优化"
  - "Skip" — "保持一切原样"
```

### 优化技能时：
1. 读取每个已安装的 SKILL.md
2. 询问用户项目的技术栈（如果尚未知道）
3. 对每个技能建议删除无关部分
4. 在安装位置（而非源代码仓库）就地编辑 SKILL.md 文件
5. 修复步骤 4 中发现的路径问题

### 优化规则时：
1. 读取每个已安装的规则 .md 文件
2. 询问用户关于设置的问题：
   - 测试覆盖率目标（默认 80%）
   - 首选格式化工具
   - Git 工作流约定
   - 安全要求
3. 在安装位置就地编辑规则文件

**重要**: 仅更改安装位置（`$TARGET/`）的文件，绝不更改源 ECC 仓库（`$ECC_ROOT/`）的文件。

---

## 步骤 6: 安装摘要

清理从 `/tmp` 克隆的仓库：

```bash
rm -rf /tmp/everything-claude-code
```

然后输出摘要报告：

```
## ECC 安装完成

### 安装位置
- 级别: [user-level / project-level / both]
- 路径: [目标路径]

### 已安装的技能（[数量]）
- skill-1, skill-2, skill-3, ...

### 已安装的规则（[数量]）
- common（8 个文件）
- typescript（5 个文件）
- ...

### 验证结果
- 发现 [数量] 个问题，已修复 [数量] 个
- [列出剩余问题]

### 应用的优化
- [列出所做的更改，或"无"]
```

---

## 故障排除

### "技能未被 Claude Code 识别"
- 确认技能目录包含 `SKILL.md` 文件（不仅仅是松散的 .md 文件）
- 用户级别时: 确认 `~/.claude/skills/<skill-name>/SKILL.md` 存在
- 项目级别时: 确认 `.claude/skills/<skill-name>/SKILL.md` 存在

### "规则不工作"
- 规则是扁平文件，不在子目录中: `$TARGET/rules/coding-style.md`（正确） vs `$TARGET/rules/common/coding-style.md`（在扁平安装中不正确）
- 安装规则后，重新启动 Claude Code

### "项目级别安装后的路径引用错误"
- 某些技能假定使用 `~/.claude/` 路径。执行步骤 4 的验证以发现并修复这些问题。
- 对于 `continuous-learning-v2`，`~/.claude/homunculus/` 目录始终在用户级别 — 这是预期行为，不是错误。
