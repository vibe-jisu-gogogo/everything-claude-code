---
name: agent-sort
description: 通过并行的仓库感知审查流程，将技能、命令、规则、钩子和扩展内容分类到DAILY和LIBRARY桶中，为特定仓库构建有证据支持的ECC安装计划。当需要将ECC裁剪为项目实际需要的内容而非加载完整包时使用。
---

# 代理分类

当仓库需要项目特定的ECC界面而非默认的完整安装时使用此技能。

我们的目标不是猜测什么"看起来有用"，而是根据实际代码库中的证据对ECC组件进行分类。

## 适用场景

- 项目只需要ECC的子集，完整安装过于冗余
- 仓库技术栈清晰，但没有人愿意逐个手动整理技能
- 团队希望有可重复的安装决策，基于grep证据而非主观意见
- 你需要区分始终加载的日常工作流界面和可搜索的库/参考界面
- 仓库使用了错误的语言、规则或钩子集，需要清理

## 不可违背的规则

- 使用当前仓库作为唯一真实来源，而非通用偏好
- 每个DAILY分类决策都必须引用具体的仓库证据
- LIBRARY不意味着"删除"，它意味着"保持可访问，但不默认加载"
- 不要安装当前仓库无法使用的钩子、规则或脚本
- 优先使用ECC原生界面，不要引入第二套安装系统

## 输出产物

按顺序生成以下产物：

1. DAILY 清单
2. LIBRARY 清单
3. 安装计划
4. 验证报告
5. 可选的`skill-library`路由（如果项目需要）

## 分类模型

仅使用两个分类桶：

- `DAILY`
  - 应该在此仓库的每个会话中加载
  - 与仓库的语言、框架、工作流或操作界面高度匹配
- `LIBRARY`
  - 保留下来很有用，但不值得默认加载
  - 应该可以通过搜索、路由技能或选择性手动使用来访问

## 证据来源

在做出任何分类之前，优先使用仓库本地证据：

- 文件扩展名
- 包管理器和锁文件
- 框架配置
- CI和钩子配置
- 构建/测试脚本
- 导入语句和依赖清单
- 明确描述技术栈的仓库文档

有用的命令包括：

```bash
rg --files
rg -n "typescript|react|next|supabase|django|spring|flutter|swift"
cat package.json
cat pyproject.toml
cat Cargo.toml
cat pubspec.yaml
cat go.mod
```

## 并行审查流程

如果有并行子代理可用，将审查拆分为以下流程：

1. 代理
   - 分类`agents/*`
2. 技能
   - 分类`skills/*`
3. 命令
   - 分类`commands/*`
4. 规则
   - 分类`rules/*`
5. 钩子和脚本
   - 分类钩子界面、MCP健康检查、辅助脚本和操作系统兼容性
6. 扩展内容
   - 分类上下文、示例、MCP配置、模板和指导文档

如果子代理不可用，按顺序运行相同的流程。

## 核心工作流

### 1. 读取仓库信息

在分类任何内容之前确定真实技术栈：

- 使用的语言
- 使用的框架
- 主包管理器
- 测试技术栈
- lint/格式化技术栈
- 部署/运行时环境
- 已存在的操作集成

### 2. 构建证据表

对于每个候选界面，记录：

- 组件路径
- 组件类型
- 建议分类桶
- 仓库证据
- 简短理由

使用此格式：

```text
skills/frontend-patterns | skill | DAILY | 84 .tsx files, next.config.ts present | core frontend stack
skills/django-patterns   | skill | LIBRARY | no .py files, no pyproject.toml       | not active in this repo
rules/typescript/*       | rules | DAILY | package.json + tsconfig.json            | active TS repo
rules/python/*           | rules | LIBRARY | zero Python source files             | keep accessible only
```

### 3. 决定DAILY或LIBRARY分类

符合以下条件时升级为`DAILY`：

- 仓库明确使用匹配的技术栈
- 组件足够通用，可以为每个会话提供帮助
- 仓库已经依赖相应的运行时或工作流

符合以下条件时降级为`LIBRARY`：

- 组件不属于当前技术栈
- 仓库可能以后需要它，但不是每天都需要
- 它会增加上下文开销，但没有即时相关性

### 4. 构建安装计划

将分类转换为操作：

- DAILY技能 -> 安装或保留在`.claude/skills/`中
- DAILY命令 -> 仅在仍然有用时保留为显式shim
- DAILY规则 -> 仅安装匹配的语言集
- DAILY钩子/脚本 -> 仅保留兼容的部分
- LIBRARY界面 -> 保持可通过搜索或`skill-library`访问

如果仓库已经使用选择性安装，更新现有计划而非创建另一个系统。

### 5. 创建可选的库路由

如果项目需要可搜索的库界面，创建：

- `.claude/skills/skill-library/SKILL.md`

该路由应包含：

- DAILY和LIBRARY的简短说明
- 分组的触发关键词
- 库参考的存放位置

不要在路由中复制每个技能的完整内容。

### 6. 验证结果

应用计划后，验证：

- 每个DAILY文件都存在于预期位置
- 过时的语言规则没有保持激活状态
- 不兼容的钩子没有被安装
- 最终安装结果实际匹配仓库技术栈

返回简洁报告，包含：

- DAILY数量
- LIBRARY数量
- 已移除的过时界面
- 未解决的问题

## 交接

如果下一步是交互式安装或修复，交接给：

- `configure-ecc`

如果下一步是重叠清理或目录审查，交接给：

- `skill-stocktake`

如果下一步是更广泛的上下文裁剪，交接给：

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
