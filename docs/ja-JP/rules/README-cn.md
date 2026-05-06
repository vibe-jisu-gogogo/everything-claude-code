# 规则

## 结构

规则由 **common** 层和 **语言特定** 目录组成：

```
rules/
├── common/          # 语言无关原则（始终安装）
│   ├── coding-style.md
│   ├── git-workflow.md
│   ├── testing.md
│   ├── performance.md
│   ├── patterns.md
│   ├── hooks.md
│   ├── agents.md
│   └── security.md
├── typescript/      # TypeScript/JavaScript 特定
├── python/          # Python 特定
└── golang/          # Go 特定
```

- **common/** 包含通用原则，不包含语言特定的代码示例。
- **语言目录** 使用框架特定的模式、工具和代码示例扩展 common 规则。每个文件引用对应的 common 文件。

## 安装

### 选项 1: 安装脚本（推荐）

```bash
# 安装 common + 一个或多个语言特定规则集
./install.sh typescript
./install.sh python
./install.sh golang

# 一次安装多个语言
./install.sh typescript python
```

### 选项 2: 手动安装

> **重要:** 请复制整个目录。不要使用 `/*` 展平。
> Common 和语言特定目录包含同名文件。
> 将它们展平到单个目录会导致语言特定文件覆盖 common 规则，
> 并破坏语言特定文件使用的相对路径 `../common/` 引用。

```bash
# 安装 common 规则（所有项目必需）
cp -r rules/common ~/.claude/rules/common

# 根据项目技术栈安装语言特定规则
cp -r rules/typescript ~/.claude/rules/typescript
cp -r rules/python ~/.claude/rules/python
cp -r rules/golang ~/.claude/rules/golang

# 注意！请根据实际项目需求配置。这里的配置仅供参考。
```

## 规则 vs 技能

- **规则** 定义广泛适用的标准、约定、检查清单（例如："80% 测试覆盖率"、"无硬编码的 secrets"）。
- **技能**（`skills/` 目录）为特定任务提供详细的可执行参考资料（例如：`python-patterns`、`golang-testing`）。

语言特定的规则文件根据需要引用相关的技能。规则说明 *做什么*，技能说明 *如何做*。

## 添加新语言

要添加对新语言（例如：`rust/`）的支持：

1. 创建 `rules/rust/` 目录
2. 添加扩展 common 规则的文件：
   - `coding-style.md` — 格式化工具、惯用法、错误处理模式
   - `testing.md` — 测试框架、覆盖率工具、测试配置
   - `patterns.md` — 语言特定的设计模式
   - `hooks.md` — 用于格式化工具、linter、类型检查器的 PostToolUse hooks
   - `security.md` — secrets 管理、安全扫描工具
3. 每个文件以以下内容开头：
   ```
   > 此文件使用 <语言> 特定内容扩展 [common/xxx.md](../common/xxx.md)。
   ```
4. 请引用现有的可用技能，或在 `skills/` 下创建新技能。
