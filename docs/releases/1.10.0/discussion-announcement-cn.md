# ECC v1.10.0 正式发布

ECC 刚刚突破 **14 万星标**，而公开发布版本与实际仓库之间的差距已经过大。

因此 v1.10.0 是一次硬同步发布：

- **38 个 agents**
- **156 个 skills**
- **72 个 commands**
- 修正了插件/安装元数据
- 顶层文档和发布内容重新对齐

此版本还整合了围绕核心 harness 系统不断发展的 operator/media 分支：

- `brand-voice`
- `social-graph-ranker`
- `connections-optimizer`
- `customer-billing-ops`
- `google-workspace-ops`
- `project-flow-ops`
- `workspace-surface-audit`
- `manim-video`
- `remotion-video-creation`

在 2.0 方面：

ECC 2.0 现在作为 **alpha 版本的 control-plane** 已实际落地，位于仓库的 `ecc2/` 目录下。

它当前可构建并提供以下功能：

- `dashboard`
- `start`
- `sessions`
- `status`
- `stop`
- `resume`
- `daemon`

这**并不**意味着完整的 ECC 2.0 路线图已经完成。

这意味着 control-plane alpha 已经到来，可用，并且正脱离"仅停留在愿景"的阶段。

目前最诚实的简短说明：

- ECC 1.x 是经过实战检验的 harness/workflow 层，目前广泛发布
- ECC 2.0 是在其上成长的 alpha 版本 control-plane

如果您一直在等待：

- 更简洁的安装界面
- 更强的跨 harness 一致性
- operator 工作流而非仅编码原语
- 真正的 control-plane 方向而非零散的笔记

那么这个版本让仓库再次变得连贯一致。
