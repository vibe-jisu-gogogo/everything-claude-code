---
name: security-scan
description: 使用 AgentShield 扫描 Claude Code 配置（.claude/ 目录）的安全漏洞、配置错误和注入风险。检查 CLAUDE.md、settings.json、MCP 服务器、hooks 和代理定义。
---

# Security Scan Skill

使用 [AgentShield](https://github.com/affaan-m/agentshield) 审计 Claude Code 配置的安全问题。

## 使用时机

- 新 Claude Code 项目设置时
- 修改 `.claude/settings.json`、`CLAUDE.md` 或 MCP 配置后
- 提交配置更改前
- 加入具有现有 Claude Code 配置的新仓库时
- 定期安全卫生检查

## 扫描对象

| 文件 | 检查内容 |
|------|--------|
| `CLAUDE.md` | 硬编码的密钥、自动执行指令、提示注入模式 |
| `settings.json` | 过于宽松的允许列表、缺失的拒绝列表、危险的绕过标志 |
| `mcp.json` | 有风险的 MCP 服务器、硬编码的环境密钥、npx 供应链风险 |
| `hooks/` | 插值导致的命令注入、数据泄露、静默错误抑制 |
| `agents/*.md` | 无限制的工具访问、提示注入面、缺失的模型规格 |

## 前提条件

需要安装 AgentShield。检查并在必要时安装：

```bash
# 检查是否已安装
npx ecc-agentshield --version

# 全局安装（推荐）
npm install -g ecc-agentshield

# 或通过 npx 直接运行（无需安装）
npx ecc-agentshield scan .
```

## 使用方法

### 基本扫描

对当前项目的 `.claude/` 目录执行扫描：

```bash
# 扫描当前项目
npx ecc-agentshield scan

# 扫描特定路径
npx ecc-agentshield scan --path /path/to/.claude

# 使用最低严重程度过滤器扫描
npx ecc-agentshield scan --min-severity medium
```

### 输出格式

```bash
# 终端输出（默认）— 带等级的彩色报告
npx ecc-agentshield scan

# JSON — 用于 CI/CD 集成
npx ecc-agentshield scan --format json

# Markdown — 用于文档
npx ecc-agentshield scan --format markdown

# HTML — 自包含的暗色主题报告
npx ecc-agentshield scan --format html > security-report.html
```

### 自动修复

自动应用安全修复（仅应用标记为可自动修复的项目）：

```bash
npx ecc-agentshield scan --fix
```

这将执行以下操作：
- 将硬编码的密钥替换为环境变量引用
- 将通配符权限收紧为有范围的替代方案
- 不更改仅手动建议的项目

### Opus 4.6 深度分析

执行用于更深入分析的对抗性三代理流水线：

```bash
# 需要 ANTHROPIC_API_KEY
export ANTHROPIC_API_KEY=your-key
npx ecc-agentshield scan --opus --stream
```

这将执行以下操作：
1. **攻击者（红队）** — 发现攻击向量
2. **防御者（蓝队）** — 建议强化措施
3. **审计员（最终判定）** — 整合双方观点

### 安全配置初始化

从零构建新的安全 `.claude/` 配置：

```bash
npx ecc-agentshield init
```

创建内容：
- 具有范围权限和拒绝列表的 `settings.json`
- 包含安全最佳实践的 `CLAUDE.md`
- `mcp.json` 占位符

### GitHub Action

添加到 CI 流水线：

```yaml
- uses: affaan-m/agentshield@v1
  with:
    path: '.'
    min-severity: 'medium'
    fail-on-findings: true
```

## 严重程度等级

| 等级 | 分数 | 含义 |
|-------|-------|---------|
| A | 90-100 | 安全配置 |
| B | 75-89 | 轻微问题 |
| C | 60-74 | 需要注意 |
| D | 40-59 | 重大风险 |
| F | 0-39 | 严重漏洞 |

## 结果解读

### 严重发现（立即修复）
- 配置文件中的硬编码 API 密钥或令牌
- 允许列表中的 `Bash(*)`（无限制 shell 访问）
- hooks 中通过 `${file}` 插值导致的命令注入
- 执行 shell 的 MCP 服务器

### 高风险发现（生产前修复）
- CLAUDE.md 中的自动执行指令（提示注入向量）
- 权限中缺失的拒绝列表
- 具有不必要 Bash 访问的代理

### 中等发现（建议）
- hooks 中的静默错误抑制（`2>/dev/null`、`|| true`）
- 缺失的 PreToolUse 安全 hook
- MCP 服务器配置中的 `npx -y` 自动安装

### 信息发现（知晓即可）
- MCP 服务器缺失的描述
- 正确标记的禁止指令（良好实践）

## 链接

- **GitHub**: [github.com/affaan-m/agentshield](https://github.com/affaan-m/agentshield)
- **npm**: [npmjs.com/package/ecc-agentshield](https://www.npmjs.com/package/ecc-agentshield)
