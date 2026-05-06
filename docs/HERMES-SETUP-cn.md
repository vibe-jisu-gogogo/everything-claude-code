# Hermes x ECC Setup

Hermes 是操作外壳。ECC 是其背后的可重用系统。

本指南是 Hermes 技术栈的公开净化版本，用于在一个终端原生界面中运行内容、外展、研究、销售运营、财务检查和工程工作流。

## 公开交付内容

- 本仓库中的 ECC skills、agents、commands、hooks 和 MCP 配置
- 足够稳定可重用的 Hermes 生成工作流 skills
- 聊天、crons、工作空间内存和分发流程的文档化操作拓扑
- 用于公开分享技术栈的发布附属材料

本指南不包含私密密钥、实时令牌、个人数据或原始的 `~/.hermes` 导出。

## 架构

将 Hermes 作为前门，ECC 作为可重用工作流基础。

```text
Telegram / CLI / TUI
        ↓
      Hermes
        ↓
 ECC skills + hooks + MCPs + generated workflow packs
        ↓
 Google Drive / GitHub / browser automation / research APIs / media tools / finance tools
```

## 公开工作空间映射

使用此作为最小化界面来重现设置，同时不泄露私密状态。

- `~/.hermes/config.yaml`
  - 模型路由
  - MCP 服务器注册
  - 插件加载
- `~/.hermes/skills/ecc-imports/`
  - 复制的 ECC skills，供 Hermes 原生使用
- `skills/hermes-generated/`
  - 从重复 Hermes 会话中提取的操作模式
- `~/.hermes/plugins/`
  - 用于 hooks、提醒和工作流特定工具粘合的桥接插件
- `~/.hermes/cron/jobs.json`
  - 带有显式提示和频道的定时自动化运行
- `~/.hermes/workspace/`
  - 业务、运营、健康、内容和内存工件

## 推荐能力栈

### 核心

- Hermes 用于聊天、cron、编排和工作空间状态
- ECC 用于 skills、规则、提示和跨工具约定
- GitHub + Context7 + Exa + Firecrawl + Playwright 作为基线 MCP 层

### 内容

- FFmpeg 用于本地编辑和组装
- Remotion 用于可编程剪辑
- fal.ai 用于图像/视频生成
- ElevenLabs 用于语音、清理和音频打包
- CapCut 或 VectCutAPI 用于最终社交平台原生润色

### 业务运营

- Google Drive 作为文档、表格、演示文稿和研究转储的记录系统
- Stripe 用于收入和支付运营
- GitHub 用于工程执行
- Telegram 和 iMessage 风格的频道用于紧急提醒和审批

## 仍需本地认证的内容

这些保留在本地，应按操作人员配置：

- Google OAuth token，用于 Drive / Docs / Sheets / Slides
- X / LinkedIn / 外部分发凭据
- Stripe keys
- 浏览器自动化凭据和隐身/代理设置
- 任何 CRM 或项目系统凭据，如 Linear 或 Apollo
- 若启用健康自动化，Apple Health 导出或摄取路径

## 建议启动顺序

0. 首先运行 `ecc migrate audit --source ~/.hermes` 来清点遗留工作空间，查看哪些部分已映射到 ECC2。
0.5. 在导入任何内容之前，规划和搭建迁移工件：
   - 使用 `ecc migrate plan` 和 `ecc migrate scaffold` 生成可审查的计划
   - 使用 `ecc migrate import-skills --output-dir migration-artifacts/skills` 搭建可重用的遗留 skills
   - 使用 `ecc migrate import-tools --output-dir migration-artifacts/tools` 搭建工具翻译模板
   - 使用 `ecc migrate import-plugins --output-dir migration-artifacts/plugins` 搭建桥接插件模板
   - 使用 `ecc migrate import-schedules --dry-run` 预览定时任务
   - 使用 `ecc migrate import-remote --dry-run` 预览网关调度
   - 使用 `ecc migrate import-env --dry-run` 预览安全环境/服务上下文
   - 使用 `ecc migrate import-memory` 导入净化的工作空间内存
1. 安装 ECC 并使用 `node tests/run-all.js` 验证基线工具设置；预期结果是零失败的测试摘要。
2. 安装 Hermes 并将其指向 ECC 导入的 skills。
3. 注册你每天实际使用的 MCP 服务器。
4. 首先认证 Google Drive，然后是 GitHub，然后是分发渠道。
5. 从小型 cron 界面开始：就绪检查、内容问责、收件箱分类、收入监控。
6. 只有在此之后，再添加更重的个人工作流，如健康、关系图谱或外联排序。

## 相关文档

- [Hermes/OpenClaw 迁移指南](HERMES-OPENCLAW-MIGRATION.md)
- [跨工具架构](architecture/cross-harness.md)

## 为什么选择 Hermes x ECC

此技术栈在以下情况下很有用：

- 一个终端原生的地方来运行业务和工程操作
- 可重用的 skills，而非一次性提示
- 能够提醒、审计和升级的自动化
- 一个公开仓库，展示系统形态而不暴露你的私有操作状态

## 公开发布候选版本范围

ECC v2.0.0-rc.1 目前记录了 Hermes 界面并交付了发布附属材料。

剩余的私密部分可以稍后分层：

- 额外的净化模板
- 更丰富的公开示例
- 更多生成的工作流包
- 更紧密的 CRM 和 Google Workspace 集成
