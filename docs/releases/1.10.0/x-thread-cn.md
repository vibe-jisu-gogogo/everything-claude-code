# X Thread 草稿 — ECC v1.10.0

ECC 的 Star 数量已突破 14 万，公开的功能面与实际的代码库差距越来越大。

因此 v1.10.0 是一个同步版本。

38 个 agents
156 个 skills
72 个 commands

修复了插件 metadata
修正了安装界面
文档和发布说明已更新至与当前代码库一致

还发布了从实际使用中成长起来的 operator / media 功能组：

- brand-voice
- social-graph-ranker
- connections-optimizer
- customer-billing-ops
- google-workspace-ops
- project-flow-ops
- workspace-surface-audit
- manim-video
- remotion-video-creation

最重要的是：

ECC 2.0 不再只是 roadmap 上的空谈。

`ecc2/` control-plane 的 alpha 版本已合入代码库，现在就能构建，并且已经提供：

- dashboard
- start
- sessions
- status
- stop
- resume
- daemon

暂时还不称之为 GA 版本。

称之为它实际的样子：

一个真正的 alpha control plane，构建在我们一直在公开构建的 harness/workflow 层之上。
