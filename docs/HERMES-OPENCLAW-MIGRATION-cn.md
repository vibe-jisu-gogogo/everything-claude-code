# Hermes / OpenClaw -> ECC 迁移指南

本文档是将 Hermes 或 OpenClaw 风格的 operator 设置迁移到当前 ECC 模型的公开迁移指南。

目标不是逐字节地复制私有 operator 工作区。

目标是保留有用的工作流层面：

- 可复用的 skills
- 稳定的自动化入口点
- 跨工具的可移植性
- 调度器 / 提醒 / 分发
- 持久化上下文和 operator 记忆

同时移除应保持私有的部分：

- 机密信息
- 个人数据集
- 账户令牌
- 仅本地使用的业务工件

## 迁移核心论点

将 Hermes 和 OpenClaw 视为源系统，而非最终运行时。

ECC 是持久化的公共系统：

- skills
- agents
- commands
- hooks
- 安装层面
- 会话适配器
- ECC 2.0 控制平面工作

Hermes 和 OpenClaw 是有用的输入，因为它们包含可提炼为 ECC 原生层面的重复 operator 工作流。

这意味着最短的安全路径是：

1. 提取可复用的行为
2. 将其转换为 ECC 原生的 skills、hooks、文档或适配器工作
3. 将机密信息和个人数据保留在仓库之外

## 当前工作区模型

一致地使用当前的工作区分割：

- 实时代码工作在 `~/GitHub` 下的克隆仓库中进行
- 仓库特定的活跃执行上下文位于仓库级别的 `WORKING-CONTEXT.md`
- 更广泛的非代码上下文可存放在知识库/归档层
- 跨机器的持久化事实应优先使用 GitHub、Linear 和知识库

不要在公共仓库内重建一个影子私有工作区。

## 转换映射

### 1. 调度器 / cron 层

源示例：

- `cron/scheduler.py`
- `jobs.py`
- 循环的就绪检查或问责循环

转换为：

- 可用时使用 Claude 原生调度
- 用于本地可重复性的 ECC hook / command 自动化
- issue `#1050` 下的 ECC 2.0 调度器工作

今天，仓库已经有了正确的公共框架：

- 用于低延迟仓库本地自动化的 hooks
- 用于显式 operator 操作的 commands
- ECC 2.0 作为未来长期存在的调度/控制平面

### 2. 网关 / 分发层

源示例：

- Hermes 网关
- 移动端分发 / 远程提醒
- 活跃会话之间的 operator 路由

转换为：

- ECC 会话适配器和控制平面工作
- 编排/会话检查 commands
- ECC 2.0 控制平面待办事项在以下编号下：
  - `#1045`
  - `#1046`
  - `#1047`
  - `#1048`

公共仓库应描述适配器边界和控制平面模型，而不是假装远程 operator shell 已经完全 GA。

### 3. 记忆层

源示例：

- `memory_tool.py`
- 本地 operator 记忆
- 业务 / 运营上下文存储

转换为：

- `knowledge-ops`
- 仓库 `WORKING-CONTEXT.md`
- GitHub / Linear / 知识库支持的持久化上下文
- `#1049` 下的未来深度记忆工作

重要的区别是：

- 仓库执行上下文应靠近仓库
- 更广泛的非代码记忆属于知识库/归档系统
- 公共仓库应记录边界，而不是存储私有记忆转储

### 4. Skill 层

源示例：

- Hermes skills
- OpenClaw skills
- 生成的 operator 操作手册

转换为：

- 当工作流可复用时，使用 ECC 原生的顶级 skills
- 当内容仅为模板时，使用 docs/examples
- 当行为是过程性的而非知识形态时，使用 hooks 或 commands

最近已通过这种方式回收的示例：

- `knowledge-ops`
- `github-ops`
- `hookify-rules`
- `automation-audit-ops`
- `email-ops`
- `finance-billing-ops`
- `messages-ops`
- `research-ops`
- `terminal-ops`
- `ecc-tools-cost-audit`

### 5. 工具 / 服务层

源示例：

- 自定义服务包装器
- API 密钥支持的本地工具
- 浏览器自动化粘合代码

转换为：

- 当存在连接器时，使用 MCP 支持的层面
- 当工作流逻辑是真正的资产时，使用 ECC 原生的 operator skills
- 当缺失的部分是会话/运行时协调时，使用适配器/控制平面工作

不要仅仅因为私有工作流依赖它们，就将不透明的第三方运行时导入 ECC。

如果一个工作流有价值：

1. 理解其行为
2. 重建最小化的 ECC 原生版本
3. 记录本地所需的认证/连接器

## 公开已存在的内容

当前仓库已经涵盖了迁移的重要部分：

- ECC 2.0 适配器/控制平面发现文档
- 编排/会话检查基础
- operator 工作流 skills
- 成本 / 账单 / 工作流审计 skills
- 跨工具安装层面
- 用于配置和 agent 层面扫描的 AgentShield

这意味着迁移问题不再是"从零开始"。

它主要是：

- 提炼缺失的私有工作流
- 澄清公共文档
- 继续 ECC 2.0 operator/控制平面构建

ECC 2.0 现在提供了一个有界的迁移审计入口点：

- `ecc migrate audit --source ~/.hermes`
- `ecc migrate plan --source ~/.hermes --output migration-plan.md`
- `ecc migrate scaffold --source ~/.hermes --output-dir migration-artifacts`
- `ecc migrate import-skills --source ~/.hermes --output-dir migration-artifacts/skills`
- `ecc migrate import-tools --source ~/.hermes --output-dir migration-artifacts/tools`
- `ecc migrate import-plugins --source ~/.hermes --output-dir migration-artifacts/plugins`
- `ecc migrate import-schedules --source ~/.hermes --dry-run`
- `ecc migrate import-remote --source ~/.hermes --dry-run`
- `ecc migrate import-env --source ~/.hermes --dry-run`
- `ecc migrate import-memory --source ~/.hermes`

首先使用它来盘点遗留工作区，并将检测到的层面映射到当前 ECC2 调度器、远程分发、记忆图、模板和手动转换通道上。

## 仍属于待办事项的内容

剩余的大型迁移主题已经被跟踪：

- `#1051` Hermes/OpenClaw 迁移
- `#1049` 深度记忆层
- `#1050` 自主调度
- `#1048` 通用工具兼容性层
- `#1046` agent 编排器
- `#1045` 多会话 TUI 管理器
- `#1047` 可视化工作树管理器

这是未解决的控制平面工作的正确位置。

不要仅仅因为公共文档存在就假装迁移"完成"了。

## 推荐的启动顺序

1. 保持公共 ECC 仓库作为规范的可复用层。
2. 一次一个通道地将可复用的 Hermes/OpenClaw 工作流移植到 ECC 原生 skills。
3. 将私有认证和个人上下文保留在仓库之外。
4. 使用 GitHub / Linear / 知识库系统作为持久化事实来源。
5. 将 ECC 2.0 视为通往原生 operator shell 的路径，而非成品。

## 决策规则

在审查 Hermes 或 OpenClaw 工件时，询问：

1. 这是否可在 operators 间复用，还是仅属于个人？
2. 该资产主要是知识、过程，还是运行时行为？
3. 它应该变成：
   - 一个 skill
   - 一个 command
   - 一个 hook
   - 一个文档/示例
   - 一个控制平面 issue
4. 公开发布是否会泄露机密信息、私有数据集或个人操作状态？

只发布可复用的层面。