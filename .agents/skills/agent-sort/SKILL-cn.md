---
name: agent-sort
description: 针对特定代码仓库构建基于证据的ECC安装方案，通过支持仓库感知的并行审核流程，将skills、commands、rules、hooks和其他组件分类到DAILY和LIBRARY两个分组中。当你需要对ECC进行裁剪，只保留项目实际需要的内容而非加载完整捆绑包时使用本skill。
---

# Agent Sort

当代码仓库需要项目专属的ECC界面而非默认的完整安装时使用本skill。

我们的目标不是猜测什么"看起来有用"，而是基于实际代码库的证据对ECC组件进行分类。

## 适用场景

- 项目仅需要ECC的一个子集，完整安装过于冗余
- 仓库技术栈清晰，但没有人希望手动逐个筛选skill
- 团队希望基于grep证据而非主观意见生成可复用的安装决策
- 你需要区分始终加载的日常workflow界面和可搜索的library/参考界面
- 仓库误用了错误的语言、rule或hook集，需要清理

## 不可违背的规则

- 以当前代码仓库作为唯一事实来源，而非通用偏好
- 所有DAILY分类决策都必须引用具体的仓库证据
- LIBRARY不意味着"删除"，而是"保持可访问性但默认不加载"
- 不要安装当前仓库无法使用的hook、rule或脚本
- 优先使用ECC原生界面，不要引入第二套安装系统

## 输出产物

按顺序生成以下产物：

1. DAILY清单
2. LIBRARY清单
3. 安装方案
4. 验证报告
5. 可选的`skill-library`路由（如果项目需要）

## 分类模型

仅使用两个分组：

- `DAILY`
  - 该仓库的每个会话都应该加载
  - 与仓库的语言、框架、workflow或操作界面高度匹配
- `LIBRARY`
  - 保留下来会很有用，但不值得默认加载
  - 应该可以通过搜索、路由skill或选择性手动使用访问

## 证据来源

在进行任何分类前优先使用仓库本地证据：

- 文件扩展名
- 包管理器和lockfile
- 框架配置
- CI和hook配置
- 构建/测试脚本
- 导入语句和依赖清单
- 明确描述技术栈的仓库文档

常用命令包括：

```bash
rg --files
rg -n "typescript|react|next|supabase|django|spring|flutter|swift"
cat package.json
cat pyproject.toml
cat Cargo.toml
cat pubspec.yaml
cat go.mod
```

## 并行审核流程

如果可以使用并行subagent，将审核拆分为以下流程：

1. Agents
   - 对`agents/*`进行分类
2. Skills
   - 对`skills/*`进行分类
3. Commands
   - 对`commands/*`进行分类
4. Rules
   - 对`rules/*`进行分类
5. Hooks和脚本
   - 对hook界面、MCP健康检查、辅助脚本和操作系统兼容性进行分类
6. 其他组件
   - 对上下文、示例、MCP配置、模板和指南文档进行分类

如果没有可用的subagent，顺序执行上述流程即可。

## 核心Workflow

### 1. 读取仓库信息

在对任何内容分类前先确认实际技术栈：

- 使用的编程语言
- 使用的框架
- 主要包管理器
- 测试技术栈
- lint/format技术栈
- 部署/runtime界面
- 已存在的操作集成

### 2. 构建证据表

对于每个候选界面，记录：

- 组件路径
- 组件类型
- 建议分组
- 仓库证据
- 简短说明

使用以下格式：

```text
skills/frontend-patterns | skill | DAILY | 84 .tsx files, next.config.ts present | core frontend stack
skills/django-patterns   | skill | LIBRARY | no .py files, no pyproject.toml       | not active in this repo
rules/typescript/*       | rules | DAILY | package.json + tsconfig.json            | active TS repo
rules/python/*           | rules | LIBRARY | zero Python source files             | keep accessible only
```

### 3. 确定DAILY和LIBRARY分类

符合以下条件时归入`DAILY`：

- 仓库明确使用匹配的技术栈
- 组件通用性足够，可为每个会话提供帮助
- 仓库已经依赖对应的runtime或workflow

符合以下条件时归入`LIBRARY`：

- 组件不属于当前技术栈
- 仓库后续可能需要，但无需每日使用
- 会增加上下文开销但没有即时相关性

### 4. 构建安装方案

将分类结果转换为可执行的操作：

- DAILY skill -> 安装或保留在`.claude/skills/`目录
- DAILY command -> 仅在仍然有用时保留为明确的shim
- DAILY rule -> 仅安装匹配的语言集
- DAILY hook/脚本 -> 仅保留兼容的内容
- LIBRARY界面 -> 保留为可通过搜索或`skill-library`访问

如果仓库已经使用选择性安装，更新现有方案即可，无需创建新系统。

### 5. 创建可选的library路由

如果项目需要可搜索的library界面，创建：

- `.claude/skills/skill-library/SKILL.md`

该路由应包含：

- DAILY和LIBRARY的简短说明
- 分组触发关键词
- library参考内容的存储位置

不要在路由中复制每个skill的完整内容。

### 6. 验证结果

应用方案后，验证：

- 所有DAILY文件都存放在预期位置
- 过时的语言rule未被保留为激活状态
- 不兼容的hook未被安装
- 最终安装结果实际匹配仓库技术栈

返回一份精简报告，包含：

- DAILY数量
- LIBRARY数量
- 已移除的过时界面
- 待解决问题

## 交接

如果下一步需要交互式安装或修复，交接给：

- `configure-ecc`

如果下一步需要重叠清理或目录审核，交接给：

- `skill-stocktake`

如果下一步需要更广泛的上下文裁剪，交接给：

- `strategic-compact`

## 输出格式

按以下顺序返回结果：

```text
STACK
- language/framework/runtime summary

DAILY
- always-loaded items with evidence

LIBRARY
- searchable/reference items with evidence

INSTALL PLAN
- what should be installed, removed, or routed

VERIFICATION
- checks run and remaining gaps
```
