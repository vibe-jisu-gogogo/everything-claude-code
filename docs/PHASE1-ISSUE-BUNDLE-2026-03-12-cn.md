# Phase 1 Issue Bundle — 2026年3月12日

## 状态

这些 Issue 草稿是根据 3月11日 的总体规划和 3月12日 的交接准备的。我曾尝试直接在 GitHub 中创建它们，但由于 MCP 会话中缺少 GitHub 身份验证，Issue 创建被阻止了。

## GitHub 状态

这些草稿后来通过 `gh` 发布了：

- `#423` 为 ECC 实现 manifest 驱动的选择性安装 profiles
- `#421` 添加 ECC install-state 以及 uninstall / doctor / repair 生命周期
- `#424` 为 ECC 2.0 控制平面定义规范的 session adapter 契约
- `#422` 定义生成的 skill 放置位置和来源策略
- `#425` 定义工具调用之后的治理和可见性

以下内容作为用于创建 Issue 的本地源包保留。

## Issue 1

### 标题

为 ECC 实现 manifest 驱动的选择性安装 profiles

### 标签

- `enhancement`

### 正文

```md
## 问题

ECC 仍然主要按目标和语言进行安装。现在仓库已经有了初步的选择性安装 manifests 和一个非突变的计划解析器，但安装程序本身尚未使用这些 profiles。

当前已落地的基础工作：

- `manifests/install-modules.json`
- `manifests/install-profiles.json`
- `scripts/ci/validate-install-manifests.js`
- `scripts/lib/install-manifests.js`
- `scripts/install-plan.js`

这意味着缺失的步骤不再是设计探索。缺失的步骤是执行：将 profile/module 解析接入实际安装流程，同时保持向后兼容性。

## 范围

为当前 ECC 目标实现 manifest 驱动的安装执行：

- `claude`
- `cursor`
- `antigravity`

添加初步支持：

- `ecc-install --profile <name>`
- `ecc-install --modules <id,id,...>`
- 基于 module 目标支持的目标感知过滤
- 部署期间向后兼容的传统语言安装模式

## 非目标

- 在同一个 Issue 中实现完整的 uninstall/doctor/repair 生命周期
- 如果阻碍部署，第一阶段不包含 Codex/OpenCode 安装目标
- 将仓库重新组织为单独发布的包

## 验收标准

- `install.sh` 可以解析并安装命名的 profile
- `install.sh` 可以解析明确的 module ID
- 目标不支持的 modules 被确定性地跳过或拒绝
- 传统的基于语言的安装模式仍然有效
- 测试覆盖 profile 解析和安装程序行为
- 文档解释新的推荐 profile/module 安装路径
```

## Issue 2

### 标题

添加 ECC install-state 以及 uninstall / doctor / repair 生命周期

### 标签

- `enhancement`

### 正文

```md
## 问题

ECC 没有规范的已安装状态记录。这使得 uninstall、repair 和安装后检查变得不确定。

现在仓库可以分类可安装内容，但它仍然无法可靠地回答：

- 安装了什么 profile/modules
- 它们安装到什么目标
- ECC 拥有哪些路径
- 如何移除或修复仅由 ECC 管理的文件

没有 install-state，生命周期命令就是猜测。

## 范围

引入持久化的 install-state 契约和首批生命周期命令：

- `ecc list-installed`
- `ecc uninstall`
- `ecc doctor`
- `ecc repair`

建议的状态位置：

- Claude: `~/.claude/ecc/install-state.json`
- Cursor: `./.cursor/ecc-install-state.json`
- Antigravity: `./.agent/ecc-install-state.json`

状态文件至少应捕获：

- 安装版本
- 时间戳
- 目标
- profile
- 解析的 modules
- 复制/管理的路径
- 源仓库版本或包版本

## 非目标

- 从头开始重建安装程序架构
- 完整的远程/云控制平面功能
- 除非自然衍生，否则不扩展超出当前本地安装程序的目标支持

## 验收标准

- 成功的安装会确定性地写入 install-state
- `list-installed` 清晰报告目标/profile/modules/版本
- `doctor` 报告缺失或偏移的管理路径
- `repair` 从记录的 install-state 恢复缺失的管理文件
- `uninstall` 仅移除 ECC 管理的文件，保留不相关的本地文件
- 测试覆盖 install-state 创建和生命周期行为
```

## Issue 3

### 标题

为 ECC 2.0 控制平面定义规范的 session adapter 契约

### 标签

- `enhancement`

### 正文

```md
## 问题

ECC 现在有了真实的编排/会话底层，但它仍然是特定于实现的。

当前状态：

- tmux/worktree 编排存在
- 机器可读的会话快照存在
- Claude 本地会话历史命令存在

尚未存在的是一个与 harness 无关的 adapter 边界，它可以跨以下平台规范化会话/任务状态：

- tmux 编排的 workers
- 普通 Claude 会话
- Codex worktrees
- OpenCode 会话
- 未来的远程或 GitHub 集成的操作界面

没有那个 adapter 契约，任何未来的 ECC 2.0 operator shell 将被迫直接读取 tmux 特定和 markdown 协调的细节。

## 范围

定义并实现初步的规范 session adapter 层。

建议的交付物：

- adapter 注册表
- 规范的会话快照 schema
- 基于当前编排代码的 `dmux-tmux` adapter
- 基于当前会话历史工具的 `claude-history` adapter
- 用于规范会话快照的只读检查 CLI

## 非目标

- 在同一个 Issue 中实现完整的 ECC 2.0 UI
- 货币化/GitHub App 实现
- 远程多用户控制平面

## 验收标准

- 存在文档化的规范快照契约
- 当前 tmux 编排快照代码被包装为 adapter，而非顶级产品契约
- 存在第二个非 tmux adapter 以证明抽象是真实的
- 测试覆盖 adapter 选择和规范化的快照输出
- 设计清晰地将 adapter 关注点与编排和 UI 关注点分离
```

## Issue 4

### 标题

定义生成的 skill 放置位置和来源策略

### 标签

- `enhancement`

### 正文

```md
## 问题

ECC 现在有一个庞大且不断增长的 skill 表面，但生成的/导入的/学习的 skills 还没有明确的长期放置位置和来源策略。

这造成了几个问题：

- curated skills 和生成的/学习的 skills 之间的分离不明确
- 验证器关于可能存在或不存在的本地目录的噪音
- 导入或机器生成的 skill 内容的来源薄弱
- 对未来自动化学习输出应该放在哪里的不确定性

随着 ECC 的发展，仓库需要明确的规则来规定生成的 skill 产物的归属以及如何识别它们。

## 范围

为以下内容定义全仓库范围的策略：

- curated vs 生成 vs 导入的 skill 放置位置
- 来源元数据要求
- 可选/生成的 skill 目录的验证器行为
- 生成的 skills 是在安装/构建步骤中发货、忽略还是具体化

## 非目标

- 构建完整的外部 skill 市场
- 一次性重写所有现有 skill 内容
- 在同一个 Issue 中解决所有内容质量问题

## 验收标准

- 存在文档化的生成/导入 skills 放置策略
- 来源要求是明确的
- 验证器不再在可选/生成的 skill 位置周围产生模糊行为
- 策略明确说明什么是可发布的，什么是仅限本地的
- 后续实施工作被拆分为具体的、有边界的 PR 大小的步骤
```
