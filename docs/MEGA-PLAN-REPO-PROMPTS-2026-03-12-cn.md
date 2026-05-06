# Mega Plan Repo Prompt List — 2026年3月12日

## 目的

使用这些提示按仓库拆分3月11日剩余的mega-plan工作。
它们是为并行agent编写的，假设3月12日的编排和
Windows CI通道已通过`#417`合并。

## 当前快照

- `everything-claude-code`已完成编排、Codex基线和
  Windows CI恢复通道。
- 下一个开放的ECC Phase 1项目是：
  - 审查`#399`
  - 将反复出现的讨论压力转化为跟踪的issues
  - 定义selective-install架构
  - 编写ECC 2.0发现文档
- `agentshield`、`ECC-website`和`skill-creator-app`都有脏的
  `main`工作树，不应直接在`main`上编辑。
- `applications/`不是独立的git仓库。它位于父工作区仓库
  的`<ECC_ROOT>`内。

## 仓库：`everything-claude-code`

### 提示A — PR `#399`审查和合并准备

```text
工作目录：<ECC_ROOT>/everything-claude-code

目标：
根据issue #398和3月11日mega plan中描述的实际循环问题，
审查PR #399（"fix(observe): 5层自动化会话防护，防止自循环观察"）。
不要假设PR上旧的失败CI仍然有意义，因为Windows基线已在#417中修复。

任务：
1. 完整阅读issue #398和PR #399。
2. 本地检查observe hook实现和测试。
3. 确定PR是否真正阻止了observer自观察、自动化会话观察
   和失控递归循环。
4. 识别任何缺失的基于环境的绕过、空闲门控或会话排除行为。
5. 生成合并建议，按严重程度排序发现。

约束：
- 不要自动合并。
- 不要重写不相关的hook行为。
- 如果进行代码更改，保持严格限定于observe行为和测试。

交付物：
- 审查摘要
- 带文件引用的确切发现
- 推荐的合并/返工决定
- 运行的测试命令
```

### 提示B — 路线图Issues提取

```text
工作目录：<ECC_ROOT>/everything-claude-code

目标：
将mega plan中反复出现的讨论压力转化为具体的GitHub issues。
专注于解除ECC 1.x和ECC 2.0阻塞的高信号路线图项目。

为以下内容创建issue草稿或准备好发布的issue包：
1. 选择性安装配置文件（selective install profiles）
2. 卸载/诊断/修复生命周期
3. 生成的skill放置和来源策略
4. 工具调用后的治理
5. ECC 2.0发现文档/adapter契约

任务：
1. 阅读3月11日的mega plan和3月12日的交接。
2. 与已开放的issues去重。
3. 起草issue标题、问题陈述、范围、非目标、验收标准
   和受影响的文件/系统区域。

约束：
- 不要创建填充性issues。
- 优先选择4-6个高价值issues，而不是大量的待办事项转储。
- 保持每个issue的范围合理，以便可以在一系列专注的PR中落地。

交付物：
- issue候选清单
- 准备好发布的issue正文
- 与现有issues的重复说明
```

### 提示C — ECC 2.0发现和Adapter规范

```text
工作目录：<ECC_ROOT>/everything-claude-code

目标：
将现有的ECC 2.0愿景转化为第一个具体的发现文档，
重点关注adapter契约、会话/任务状态、token计数和安全/策略事件。

任务：
1. 使用当前的编排/会话快照代码作为基线。
2. 为Claude Code、Codex、OpenCode以及后来的Cursor/GitHub App集成
   定义标准化的adapter契约。
3. 为sessions、tasks、worktrees、events、findings和approvals
   定义初始的SQLite支持的数据模型。
4. 定义哪些内容保留在ECC 1.x中，哪些属于ECC 2.0。
5. 将未解决的产品决策与实现要求分开列出。

约束：
- 将当前的tmux/worktree/session快照底层作为起点，
  而不是一张白纸。
- 保持文档面向实现。

交付物：
- 发现文档
- adapter契约草图
- 事件模型草图
- 未解决问题列表
```

## 仓库：`agentshield`

### 提示 — 误报审计和回归计划

```text
工作目录：<ECC_ROOT>/agentshield

目标：
推进mega plan中的AgentShield Phase 2工作流：减少误报，
特别是在声明性拒绝规则、block hooks、文档示例或
配置片段被错误分类为可执行风险的情况下。

重要仓库状态：
- 当前分支是main
- CLAUDE.md和README.md中有脏文件
- 在更广泛更改之前分类或搁置现有编辑

任务：
1. 检查当前围绕以下内容的误报行为：
   - .claude hook配置
   - AGENTS.md / CLAUDE.md
   - .cursor规则
   - .opencode插件配置
   - 示例拒绝列表模式
2. 分离声明性模式与可执行命令的解析器行为。
3. 建议添加回归覆盖范围和所需的确切fixture集。
4. 如果在分支设置后安全，则实施分类器修复的第一遍。

约束：
- 不要直接在脏的main上工作
- 保持修复限于解析器/分类器范围
- 明确记录任何剩余的歧义

交付物：
- 分支建议
- 误报分类
- 建议的或已落地的回归测试
- 剩余的边缘情况
```

## 仓库：`ECC-website`

### 提示 — 着陆页重写和产品框架

```text
工作目录：<ECC_ROOT>/ECC-website

目标：
通过重写着陆页/产品框架来执行mega plan中的网站通道，
从"配置仓库"转向"开放agent harness系统"加上
未来的控制平面方向。

重要仓库状态：
- 当前分支是main
- favicon资产和多个页面/组件文件中有脏文件
- 在有意义的工作之前分支，保留现有编辑，除非明确分类为过时

任务：
1. 分类脏的main工作树状态。
2. 围绕以下内容重写着陆页叙述：
   - 开放agent harness系统
   - 运行时防护
   - 跨harness parity
   - 操作员可见性和安全性
3. 定义或更新下一个关键页面：
   - /skills
   - /security
   - /platforms
   - /system或/dashboard
4. 保持页面视觉上有意图且面向产品，不是通用的SaaS。

约束：
- 不要静默覆盖现有的脏工作
- 在现有设计系统连贯的地方保留它
- 清楚区分ECC 1.x工具包与ECC 2.0控制平面

交付物：
- 分支建议
- 着陆页重写diff或内容规范
- 后续页面地图
- 部署准备说明
```

## 仓库：`skill-creator-app`

### 提示 — Skill导入管道和产品适配

```text
工作目录：<ECC_ROOT>/skill-creator-app

目标：
使skill-creator-app与mega plan的外部skill来源和
经过审计的导入管道工作流保持一致。

重要仓库状态：
- 当前分支是main
- README.md和src/lib/github.ts中有脏文件
- 在更广泛工作之前分类或搁置现有更改

任务：
1. 评估应用是否应支持：
   - 盘点外部skills
   - 来源标记
   - 依赖/风险审计字段
   - ECC约定适配工作流
2. 检查src/lib/github.ts中现有的GitHub集成表面。
3. 为经过审计的导入管道生成具体的产品/技术范围。
4. 如果在分支后安全，落地最小的支持性更改，
   用于元数据捕获或GitHub摄取。

约束：
- 不要将其变成通用的prompt构建器
- 专注于经过审计的skill摄取和ECC兼容输出

交付物：
- 产品适配摘要
- v1的推荐范围
- 导入管道的数据字段/工作流步骤
- 如果小且明确合理则进行代码更改
```

## 仓库：`ECC`工作区（`applications/`、`knowledge/`、`tasks/`）

### 提示 — 示例应用和工作流可靠性证明

```text
工作目录：<ECC_ROOT>

目标：
使用父ECC工作区支持mega plan的托管/工作流通道。
这不是独立的应用程序仓库；它是包含applications/、
knowledge/、tasks/和相关规划资产的伞形工作区。

任务：
1. 盘点applications/中哪些是真实产品代码，哪些是占位符。
2. 确定示例仓库或演示应用应该放置在哪里：
   - GitHub App工作流证明
   - ECC 2.0原型尖峰
   - 示例安装/设置可靠性检查
3. 提出一个干净的工作区结构，使产品代码、
   研究和规划不再相互渗透。
4. 推荐应首先构建哪个概念验证。

约束：
- 不要盲目移动大型目录
- 区分仓库结构建议与即时代码更改
- 保持建议与当前的多仓库ECC设置兼容

交付物：
- 工作区盘点
- 建议的结构
- 第一个演示/应用建议
- 后续分支/工作树计划
```

## 本地延续

当前工作树应保持在ECC原生的Phase 1工作上，
不接触这里现有的脏skill-file更改。
最佳的下一个本地任务是：

1. selective-install架构
2. ECC 2.0发现文档
3. PR `#399`审查
