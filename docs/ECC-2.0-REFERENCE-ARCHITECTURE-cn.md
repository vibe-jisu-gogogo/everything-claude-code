# ECC 2.0 Reference Architecture

竞争对手/参考分析研究摘要 (2026-03-22)。

## Competitive Landscape

| Project | Stars | Language | Type | Multi-Agent | Worktrees | Terminal-native |
|---------|-------|----------|------|-------------|-----------|-----------------|
| **ECC 2.0** | - | Rust | TUI | Yes | Yes | **Yes (SSH)** |
| superset-sh/superset | 7.7K | TypeScript | Electron | Yes | Yes | No (desktop) |
| standardagents/dmux | 1.2K | TypeScript | TUI (Ink) | Yes | Yes | Yes |
| opencode-ai/opencode | 11.5K | Go | TUI | No | No | Yes |
| smtg-ai/claude-squad | 6.5K | Go | TUI | Yes | Yes | Yes |

## Three-Layer Architecture

```
┌─────────────────────────────────┐
│        TUI Layer (ratatui)      │  用户仪表盘
│  Panes, diff viewer, hotkeys    │  通过 Unix socket 通信
├─────────────────────────────────┤
│     Runtime Layer (library)     │  工作空间运行时、代理注册表、
│  State persistence, detection   │  状态检测、SQLite
├─────────────────────────────────┤
│     Daemon Layer (process)      │  TUI 重启后仍保持持久化
│  Terminal sessions, git ops,    │  PTY 管理、心跳检测
│  agent process supervision      │
└─────────────────────────────────┘
```

## Patterns to Adopt

### From Superset (Electron, 7.7K stars)
- **Workspace Runtime Registry** — 基于 trait 的带能力标志的抽象
- **Persistent daemon terminal** — 会话通过 IPC 在重启后存活
- **Per-project mutex** 用于 git 操作（防止竞态条件）
- **Port allocation** 为每个工作空间分配开发服务器端口
- **Cold restore** 从序列化的终端回滚缓冲区恢复

### From dmux (Ink TUI, 1.2K stars)
- **Worker-per-pane status detection** — 终端输出指纹识别 + LLM 分类
- **Agent Registry** — 集中式代理定义（安装检查、启动命令、权限）
- **Retry strategies** — 针对破坏性操作与只读操作的不同策略
- **PaneLifecycleManager** — 排他锁防止并发 pane 竞态
- **Lifecycle hooks** — worktree_created、pre_merge、post_merge
- **Background cleanup queue** — 异步 worktree 删除

## ECC 2.0 Advantages
- Terminal-native（可通过 SSH 工作，与 Superset 不同）
- 与 116 个技能生态系统集成
- AgentShield 安全扫描
- 自我改进的技能演进 (continuous-learning-v2)
- Rust 单二进制文件 (3.4MB，无运行时依赖)
- 开源中首个基于 Rust 的 agentic IDE TUI
