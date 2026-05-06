# 命令快速参考
> 全局安装了59个斜杠命令。在任意 Claude Code 会话中输入 `/` 即可调用。

---

## 核心工作流
| 命令 | 功能说明 |
|---------|-------------|
| `/plan` | 重述需求、评估风险、编写分步实现计划 — **在修改代码前会等待您的确认** |
| `/tdd` | 强制执行测试驱动开发：搭建接口框架 → 编写失败测试 → 实现功能 → 验证80%以上覆盖率 |
| `/code-review` | 对变更文件进行完整的代码质量、安全性和可维护性审查 |
| `/build-fix` | 检测并修复构建错误 — 自动委派给合适的构建解析代理 |
| `/verify` | 运行完整的验证流程：构建 → 代码检查 → 测试 → 类型检查 |
| `/quality-gate` | 对照项目标准进行质量门禁检查 |

---

## 测试相关
| 命令 | 功能说明 |
|---------|-------------|
| `/tdd` | 通用 TDD 工作流（支持任意语言） |
| `/e2e` | 生成并运行 Playwright 端到端测试，捕获截图/视频/追踪信息 |
| `/test-coverage` | 报告测试覆盖率，识别覆盖缺口 |
| `/go-test` | Go 语言 TDD 工作流（表驱动测试，通过 `go test -cover` 实现80%以上覆盖率） |
| `/kotlin-test` | Kotlin 语言 TDD 工作流（Kotest + Kover） |
| `/rust-test` | Rust 语言 TDD 工作流（cargo test，集成测试） |
| `/cpp-test` | C++ 语言 TDD 工作流（GoogleTest + gcov/lcov） |

---

## 代码审查
| 命令 | 功能说明 |
|---------|-------------|
| `/code-review` | 通用代码审查 |
| `/python-review` | Python 代码审查 — PEP 8 规范、类型提示、安全性、惯用模式 |
| `/go-review` | Go 代码审查 — 惯用模式、并发安全、错误处理 |
| `/kotlin-review` | Kotlin 代码审查 — 空安全、协程安全、整洁架构 |
| `/rust-review` | Rust 代码审查 — 所有权、生命周期、unsafe 用法 |
| `/cpp-review` | C++ 代码审查 — 内存安全、现代惯用语法、并发处理 |

---

## 构建修复
| 命令 | 功能说明 |
|---------|-------------|
| `/build-fix` | 自动检测语言并修复构建错误 |
| `/go-build` | 修复 Go 构建错误和 `go vet` 警告 |
| `/kotlin-build` | 修复 Kotlin/Gradle 编译器错误 |
| `/rust-build` | 修复 Rust 构建和借用检查器问题 |
| `/cpp-build` | 修复 C++ CMake 和链接器问题 |
| `/gradle-build` | 修复 Android / KMP 项目的 Gradle 错误 |

---

## 规划与架构
| 命令 | 功能说明 |
|---------|-------------|
| `/plan` | 包含风险评估的实现计划 |
| `/multi-plan` | 多模型协作规划 |
| `/multi-workflow` | 多模型协作开发 |
| `/multi-backend` | 聚焦后端的多模型开发 |
| `/multi-frontend` | 聚焦前端的多模型开发 |
| `/multi-execute` | 多模型协作执行 |
| `/orchestrate` | tmux/worktree 多代理编排指南 |
| `/devfleet` | 通过 DevFleet 编排并行的 Claude Code 代理 |

---

## 会话管理
| 命令 | 功能说明 |
|---------|-------------|
| `/save-session` | 将当前会话状态保存到 `~/.claude/session-data/` |
| `/resume-session` | 从标准会话存储中加载最近保存的会话，从上次中断处继续 |
| `/sessions` | 浏览、搜索和管理 `~/.claude/session-data/` 中的会话历史及别名（同时支持从 `~/.claude/sessions/` 读取历史数据） |
| `/checkpoint` | 在当前会话中标记一个检查点 |
| `/aside` | 回答快速的旁支问题，不丢失当前任务上下文 |
| `/context-budget` | 分析上下文窗口使用情况 — 查找 Token 开销，进行优化 |

---

## 学习与改进
| 命令 | 功能说明 |
|---------|-------------|
| `/learn` | 从当前会话中提取可复用模式 |
| `/learn-eval` | 提取模式并在保存前进行质量自评 |
| `/evolve` | 分析已学习的直觉模式，建议优化后的技能结构 |
| `/promote` | 将项目范围的直觉模式提升到全局范围 |
| `/instinct-status` | 显示所有已学习的直觉模式（项目 + 全局）及置信度评分 |
| `/instinct-export` | 将直觉模式导出到文件 |
| `/instinct-import` | 从文件或 URL 导入直觉模式 |
| `/skill-create` | 分析本地 git 历史 → 生成可复用技能 |
| `/skill-health` | 带数据分析的技能组合健康仪表盘 |
| `/rules-distill` | 扫描技能，提取跨领域原则，提炼为规则 |

---

## 重构与清理
| 命令 | 功能说明 |
|---------|-------------|
| `/refactor-clean` | 移除死代码、合并重复代码、清理结构 |
| `/prompt-optimize` | 分析提示词草稿，输出优化后的 ECC 增强版本 |

---

## 文档与研究
| 命令 | 功能说明 |
|---------|-------------|
| `/docs` | 通过 Context7 查找当前库/API 文档 |
| `/update-docs` | 更新项目文档 |
| `/update-codemaps` | 重新生成代码库的代码地图 |

---

## 循环与自动化
| 命令 | 功能说明 |
|---------|-------------|
| `/loop-start` | 按时间间隔启动周期性代理循环 |
| `/loop-status` | 检查运行中循环的状态 |
| `/claw` | 启动 NanoClaw v2 — 支持模型路由、技能热加载、分支和指标的持久化 REPL |

---

## 项目与基础设施
| 命令 | 功能说明 |
|---------|-------------|
| `/projects` | 列出已知项目及其直觉模式统计信息 |
| `/harness-audit` | 审计代理运行时配置的可靠性和成本 |
| `/eval` | 运行评估套件 |
| `/model-route` | 将任务路由到合适的模型（Haiku / Sonnet / Opus） |
| `/pm2` | PM2 进程管理器初始化 |
| `/setup-pm` | 配置包管理器（npm / pnpm / yarn / bun） |

---

## 快速决策指南
```
开始新功能开发？         → 先使用 /plan，然后 /tdd
刚写完代码？              → /code-review
构建失败？                   → /build-fix
需要实时文档？                 → /docs <library>
会话即将结束？           → /save-session 或 /learn-eval
次日恢复工作？              → /resume-session
上下文占用过高？          → 先 /context-budget 再 /checkpoint
想要提取所学内容？ → 先 /learn-eval 再 /evolve
运行重复任务？         → /loop-start
```