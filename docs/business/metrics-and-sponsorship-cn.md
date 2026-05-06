# Metrics and Sponsorship Playbook

本文档是赞助商电话会议和生态合作伙伴评审的实用脚本。

## 跟踪内容

每次更新使用四个类别：

1. **Distribution** — npm 包和 GitHub App 安装量
2. **Adoption** — stars、forks、贡献者、发布节奏
3. **Product surface** — commands/skills/agents 和跨平台支持
4. **Reliability** — 测试通过率和生产环境问题修复时间

## 获取实时指标

### npm 下载量

```bash
# 周下载量
curl -s https://api.npmjs.org/downloads/point/last-week/ecc-universal
curl -s https://api.npmjs.org/downloads/point/last-week/ecc-agentshield

# 近30天
curl -s https://api.npmjs.org/downloads/point/last-month/ecc-universal
curl -s https://api.npmjs.org/downloads/point/last-month/ecc-agentshield
```

### GitHub 仓库采纳情况

```bash
gh api repos/affaan-m/everything-claude-code \
  --jq '{stars:.stargazers_count,forks:.forks_count,contributors_url:.contributors_url,open_issues:.open_issues_count}'
```

### GitHub 流量（需要维护者权限）

```bash
gh api repos/affaan-m/everything-claude-code/traffic/views
gh api repos/affaan-m/everything-claude-code/traffic/clones
```

### GitHub App 安装量

GitHub App 安装数量目前在 Marketplace/App 仪表板中最可靠。
使用以下链接的最新值：

- [ECC Tools Marketplace](https://github.com/marketplace/ecc-tools)

## 目前无法公开测量的内容

- Claude 插件安装/下载数量目前未通过公共 API 暴露。
- 合作伙伴沟通时，使用 npm 指标 + GitHub App 安装量 + 仓库流量作为组合代理指标。

## 建议的赞助套餐

谈判时以此为起点：

- **Pilot Partner:** `$200/month`
  - 最适合首次合作验证和简单的月度赞助更新。
- **Growth Partner:** `$500/month`
  - 包含路线图沟通和实施反馈循环。
- **Strategic Partner:** `$1,000+/month`
  - 多维度协作、发布支持和深度运营对齐。

## 60 秒话术

通话时使用：

> ECC 现在定位为 agent harness 性能系统，而非配置仓库。
> 我们通过 npm 分发、GitHub App 安装和仓库增长来跟踪采纳情况。
> Claude 插件安装量在公开数据中存在结构性低估，因此我们使用混合指标模型。
> 项目支持 Claude Code、Cursor、OpenCode 和 Codex app/CLI，具备生产级 hook 可靠性和大型通过测试套件。

发布就绪的社交文案片段，请参阅 [`social-launch-copy.md`](./social-launch-copy.md)。
