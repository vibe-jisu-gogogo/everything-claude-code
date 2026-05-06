# Antigravity 安装和使用指南

Google 的 [Antigravity](https://antigravity.dev) 是一款 AI 编程 IDE，使用 `.agent/` 目录约定进行配置。ECC 通过其选择性安装系统为 Antigravity 提供一流支持。

## 快速开始

```bash
# 安装带有 Antigravity 目标的 ECC
./install.sh --target antigravity typescript

# 或者使用多语言模块
./install.sh --target antigravity typescript python go
```

这会将 ECC 组件安装到项目的 `.agent/` 目录中，供 Antigravity 读取使用。

## 安装映射如何工作

ECC 重新映射其组件结构以匹配 Antigravity 的预期布局：

| ECC 源路径 | Antigravity 目标路径 | 内容 |
|------------|---------------------|------|
| `rules/` | `.agent/rules/` | 语言规则和编码标准（扁平化） |
| `commands/` | `.agent/workflows/` | 斜杠命令变为 Antigravity workflows |
| `agents/` | `.agent/skills/` | 代理定义变为 Antigravity skills |

> **关于 `.agents/` vs `.agent/` vs `agents/` 的说明**：安装程序仅明确处理三个源路径：`rules` → `.agent/rules/`、`commands` → `.agent/workflows/`、以及 `agents`（无前导点）→ `.agent/skills/`。ECC 仓库中带点前缀的 `.agents/` 目录是 Codex/Antigravity skill 定义和 `openai.yaml` 配置的**静态布局**——安装程序不会直接映射它。任何 `.agents/` 路径都会回退到默认脚手架操作。如果你希望 `.agents/skills/` 内容在 Antigravity 运行时可用，必须手动将其复制到 `.agent/skills/`。

### 与 Claude Code 的主要区别

- **规则是扁平化的**：Claude Code 将规则嵌套在子目录下（`rules/common/`、`rules/typescript/`）。Antigravity 需要扁平的 `rules/` 目录——安装程序会自动处理。
- **命令变为 workflows**：ECC 的 `/command` 文件放置在 `.agent/workflows/` 中，这是 Antigravity 中等同于斜杠命令的功能。
- **代理变为 skills**：ECC 代理定义映射到 `.agent/skills/`，这是 Antigravity 查找 skill 配置的位置。

## 安装后的目录结构

```
your-project/
├── .agent/
│   ├── rules/
│   │   ├── coding-standards.md
│   │   ├── testing.md
│   │   ├── security.md
│   │   └── typescript.md          # 语言特定规则
│   ├── workflows/
│   │   ├── plan.md
│   │   ├── code-review.md
│   │   ├── tdd.md
│   │   └── ...
│   ├── skills/
│   │   ├── planner.md
│   │   ├── code-reviewer.md
│   │   ├── tdd-guide.md
│   │   └── ...
│   └── ecc-install-state.json     # 跟踪 ECC 安装的内容
```

## `openai.yaml` 代理配置

`.agents/skills/` 下的每个 skill 目录都包含一个 `agents/openai.yaml` 文件，路径为 `.agents/skills/<skill-name>/agents/openai.yaml`，用于为 Antigravity 配置 skill：

```yaml
interface:
  display_name: "API Design"
  short_description: "REST API design patterns and best practices"
  brand_color: "#F97316"
  default_prompt: "Design REST API: resources, status codes, pagination"
policy:
  allow_implicit_invocation: true
```

| 字段 | 用途 |
|------|------|
| `display_name` | Antigravity UI 中显示的人类可读名称 |
| `short_description` | skill 功能的简要描述 |
| `brand_color` | skill 视觉徽章的十六进制颜色 |
| `default_prompt` | 手动调用 skill 时的建议提示词 |
| `allow_implicit_invocation` | 当为 `true` 时，Antigravity 可以根据上下文自动激活 skill |

## 管理你的安装

### 检查已安装内容

```bash
node scripts/list-installed.js --target antigravity
```

### 修复损坏的安装

```bash
# 首先，诊断问题
node scripts/doctor.js --target antigravity

# 然后，恢复缺失或变更的文件
node scripts/repair.js --target antigravity
```

### 卸载

```bash
node scripts/uninstall.js --target antigravity
```

### 安装状态

安装程序会写入 `.agent/ecc-install-state.json` 来跟踪 ECC 拥有哪些文件。这支持安全卸载和修复——ECC 永远不会接触它没有创建的文件。

## 为 Antigravity 添加自定义 Skills

如果你正在贡献一个新 skill，并希望它在 Antigravity 上可用：

1. 像往常一样在 `skills/your-skill-name/SKILL.md` 下创建 skill
2. 在 `agents/your-skill-name.md` 添加一个代理定义——这是安装程序在运行时映射到 `.agent/skills/` 的路径，使你的 skill 在 Antigravity 环境中可用
3. 在 `.agents/skills/your-skill-name/agents/openai.yaml` 添加 Antigravity 代理配置——这是 Codex 用于隐式调用元数据的静态仓库布局
4. 将 `SKILL.md` 内容镜像到 `.agents/skills/your-skill-name/SKILL.md`——这个静态副本供 Codex 使用，并作为 Antigravity 的参考
5. 在你的 PR 中说明你添加了 Antigravity 支持

> **关键区别**：安装程序部署 `agents/`（无点前缀）→ `.agent/skills/`——这是使 skills 在运行时可用的原因。带点前缀的 `.agents/` 目录是 Codex `openai.yaml` 配置的单独静态布局，安装程序不会自动部署。

完整的贡献指南请参阅 [CONTRIBUTING.md](../CONTRIBUTING.md)。

## 与其他目标的比较

| 功能 | Claude Code | Cursor | Codex | Antigravity |
|------|-------------|--------|-------|-------------|
| 安装目标 | `claude-home` | `cursor-project` | `codex-home` | `antigravity` |
| 配置根目录 | `~/.claude/` | `.cursor/` | `~/.codex/` | `.agent/` |
| 作用范围 | 用户级别 | 项目级别 | 用户级别 | 项目级别 |
| 规则格式 | 嵌套目录 | 扁平 | 扁平 | 扁平 |
| 命令 | `commands/` | N/A | N/A | `workflows/` |
| 代理/Skills | `agents/` | N/A | N/A | `skills/` |
| 安装状态 | `ecc-install-state.json` | `ecc-install-state.json` | `ecc-install-state.json` | `ecc-install-state.json` |

## 故障排除

### Skills 在 Antigravity 中不加载

- 验证 `.agent/` 目录存在于你的项目根目录（不是主目录）
- 检查 `ecc-install-state.json` 是否已创建——如果缺失，重新运行安装程序
- 确保文件具有 `.md` 扩展名和有效的 frontmatter

### 规则不生效

- 规则必须在 `.agent/rules/` 中，不能嵌套在子目录中
- 运行 `node scripts/doctor.js --target antigravity` 验证安装

### Workflows 不可用

- Antigravity 在 `.agent/workflows/` 中查找 workflows，而不是 `commands/`
- 如果你手动复制了 ECC 命令，请重命名目录

## 相关资源

- [选择性安装架构](./SELECTIVE-INSTALL-ARCHITECTURE.md) — 安装系统的内部工作原理
- [选择性安装设计](./SELECTIVE-INSTALL-DESIGN.md) — 设计决策和目标适配器契约
- [CONTRIBUTING.md](../CONTRIBUTING.md) — 如何贡献 skills、agents 和 commands
