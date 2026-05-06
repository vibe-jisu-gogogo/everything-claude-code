# 仓库与分叉评估 + 安装建议

**日期：** 2026-03-21

---

## 可用内容

### 仓库：`Infiniteyieldai/everything-claude-code`

这是**`affaan-m/everything-claude-code`的分叉**（上游项目拥有5万+星标，6千+分叉）。

| 属性 | 值 |
|-----------|-------|
| 版本 | 1.9.0（当前） |
| 状态 | 干净的分叉 — 领先上游`main`分支1个提交（本次会话中添加的EVALUATION.md文档） |
| 远程分支 | `main`、`claude/evaluate-repo-comparison-ASZ9Y` |
| 上游同步 | 完全同步 — 最后合并的上游提交是中文文档PR（#728） |
| 许可证 | MIT |

**这是适合使用的正确仓库。** 它是最新的上游版本，没有代码偏离或合并冲突。

---

### 当前`~/.claude/`安装情况

| 组件 | 已安装 | 仓库中可用 |
|-----------|-----------|-------------------|
| Agents | 0 | 28 |
| Skills | 0 | 116 |
| Commands | 0 | 59 |
| Rules | 0 | 60+文件（12种语言） |
| Hooks | 1（git停止检查） | 完整的PreToolUse/PostToolUse矩阵 |
| MCP配置 | 0 | 1（Context7） |

现有的Stop hook（`stop-hook-git-check.sh`）很可靠 — 当存在未提交/未推送的工作时会阻止会话结束。请保留它。

---

## 安装配置文件建议

仓库提供5种安装配置文件。请根据你的主要使用场景选择：

### 配置文件：`core`（最小可用安装）
> 安装速度最快。包含commands、core agents、hooks运行时和质量工作流。

**适用场景：** 试用ECC、最小资源占用，或受限环境。

```bash
node scripts/install-plan.js --profile core
node scripts/install-apply.js
```

**安装内容：** rules-core、agents-core、commands-core、hooks-runtime、platform-configs、workflow-quality

---

### 配置文件：`developer`（推荐日常开发使用）
> 面向大多数ECC用户的默认工程配置。

**适用场景：** 跨应用代码库的通用软件开发。

```bash
node scripts/install-plan.js --profile developer
node scripts/install-apply.js
```

**在core基础上新增：** framework-language skills、database patterns、orchestration commands

---

### 配置文件：`security`
> 基础运行时 + 安全专用的agents和rules。

**适用场景：** 安全相关工作流、代码审计、漏洞审查。

---

### 配置文件：`research`
> 调研、综合分析和发布工作流。

**适用场景：** 内容创作、投资者材料、市场调研、跨平台发布。

---

### 配置文件：`full`
> 全部内容 — 所有18个模块。

**适用场景：** 需要完整工具集的高级用户。

```bash
node scripts/install-plan.js --profile full
node scripts/install-apply.js
```

---

## 优先安装组件（高价值、低风险）

无论选择哪种配置文件，这些组件都能立即带来价值：

### 1. Core Agents（最高投入回报率）

| Agent | 价值说明 |
|-------|----------------|
| `planner.md` | 将复杂任务拆分为可执行的实现计划 |
| `code-reviewer.md` | 代码质量和可维护性审查 |
| `tdd-guide.md` | TDD工作流（红→绿→重构） |
| `security-reviewer.md` | 漏洞检测 |
| `architect.md` | 系统设计与可扩展性决策 |

### 2. 核心Commands

| Command | 价值说明 |
|---------|----------------|
| `/plan` | 编码前的实现规划 |
| `/tdd` | 测试驱动开发工作流 |
| `/code-review` | 按需代码审查 |
| `/build-fix` | 自动解决构建错误 |
| `/learn` | 从当前会话中提取模式 |

### 3. Hook升级（来自`hooks/hooks.json`）
仓库的hook系统在当前单个Stop hook基础上新增以下功能：

| Hook | 触发时机 | 价值 |
|------|---------|-------|
| `block-no-verify` | PreToolUse: Bash | 阻止滥用`--no-verify` git参数 |
| `pre-bash-git-push-reminder` | PreToolUse: Bash | 推送前审查提醒 |
| `doc-file-warning` | PreToolUse: Write | 非标准文档文件警告 |
| `suggest-compact` | PreToolUse: Edit/Write | 在合理时机建议内容压缩 |
| 持续学习观察器 | PreToolUse: * | 捕获工具使用模式以改进skill |

### 4. Rules（始终生效的指南）
`rules/common/`目录提供每个会话都会生效的基础指南：
- `security.md` — 安全护栏
- `testing.md` — 80%以上测试覆盖率要求
- `git-workflow.md` — 约定式提交、分支策略
- `coding-style.md` — 跨语言编码风格标准

---

## 分叉使用建议

### 选项A：作为上游跟踪仓库使用（当前状态）
保持分叉与`affaan-m/everything-claude-code`上游同步。定期合并上游变更：
```bash
git fetch upstream
git merge upstream/main
```
从本地克隆安装。这种方式干净且易维护。

### 选项B：自定义分叉
向分叉中添加个人的skills、agents或commands。适用于：
- 特定业务领域skills（你的行业领域）
- 团队专属编码规范
- 适配你的技术栈的自定义hooks

分叉中已经包含EVALUATION.md和REPO-ASSESSMENT.md文档 — 作为工作分叉这是没问题的。

### 选项C：从npm安装（新机器最简单的方式）
```bash
npx ecc-universal install --profile developer
```
无需克隆仓库。这是推荐给大多数用户的安装方式。

---

## 推荐安装步骤

1. **保留现有的Stop hook** — 它工作正常
2. **从本地分叉运行developer配置安装：**
   ```bash
   cd /path/to/everything-claude-code
   node scripts/install-plan.js --profile developer
   node scripts/install-apply.js
   ```
3. **为你的主要技术栈添加language rules**（TypeScript、Python、Go等）：
   ```bash
   node scripts/install-plan.js --add rules/typescript
   node scripts/install-apply.js
   ```
4. **启用MCP Context7**用于实时文档查询：
   - 将`mcp-configs/mcp-servers.json`复制到你项目的`.claude/`目录
5. **审查hooks** — 选择性启用`hooks/hooks.json`中的新增内容，从`block-no-verify`和`pre-bash-git-push-reminder`开始

---

## 总结

| 问题 | 答案 |
|----------|--------|
| 分叉状态是否正常？ | 是 — 与上游v1.9.0完全同步 |
| 有其他分叉值得考虑吗？ | 当前环境中没有可见的其他分叉；上游`affaan-m/everything-claude-code`是唯一可信来源 |
| 最佳安装配置？ | 日常开发使用`developer` |
| 当前安装的最大缺失？ | 0个agents已安装 — 至少添加：planner、code-reviewer、tdd-guide、security-reviewer |
| 最快见效的操作？ | 运行`node scripts/install-plan.js --profile core && node scripts/install-apply.js` |
