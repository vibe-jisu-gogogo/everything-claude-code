---
description: 执行具有严格验证循环的实现计划
argument-hint: <path/to/plan.md>
---

> 改编自 Wirasm 的 PRPs-agentic-eng。PRP 工作流系列的一部分。

# PRP 实现

逐步执行计划文件并进行持续验证。每次变更立即验证 — 绝不累积损坏状态。

**核心理念**: 验证循环及早发现错误。每次变更后运行检查。立即修复问题。

**黄金法则**: 如果验证失败，在继续之前先修复它。绝不累积损坏状态。

---

## 阶段 0 — 检测

### 包管理器检测

| 存在的文件 | 包管理器 | 运行器 |
|---|---|---|
| `bun.lockb` | bun | `bun run` |
| `pnpm-lock.yaml` | pnpm | `pnpm run` |
| `yarn.lock` | yarn | `yarn` |
| `package-lock.json` | npm | `npm run` |
| `pyproject.toml` 或 `requirements.txt` | uv / pip | `uv run` 或 `python -m` |
| `Cargo.toml` | cargo | `cargo` |
| `go.mod` | go | `go` |

### 验证脚本

检查 `package.json`（或等效文件）中的可用脚本：

```bash
# 对于 Node.js 项目
cat package.json | grep -A 20 '"scripts"'
```

记录可用的命令：type-check、lint、test、build。

---

## 阶段 1 — 加载

读取计划文件：

```bash
cat "$ARGUMENTS"
```

从计划中提取以下部分：
- **摘要** — 正在构建什么
- **要遵循的模式** — 要遵循的代码约定
- **要更改的文件** — 要创建或修改的内容
- **分步任务** — 实现顺序
- **验证命令** — 如何验证正确性
- **验收标准** — 完成的定义

如果文件不存在或不是有效的计划：
```
错误：未找到计划文件或文件无效。
先运行 /prp-plan <feature-description> 创建计划。
```

**检查点**: 计划已加载。所有部分已识别。任务已提取。

---

## 阶段 2 — 准备

### Git 状态

```bash
git branch --show-current
git status --porcelain
```

### 分支决策

| 当前状态 | 操作 |
|---|---|
| 在功能分支上 | 使用当前分支 |
| 在 main 上，工作树干净 | 创建功能分支：`git checkout -b feat/{plan-name}` |
| 在 main 上，工作树脏 | **停止** — 要求用户先暂存或提交 |
| 在为此功能的 git worktree 中 | 使用该 worktree |

### 同步远程

```bash
git pull --rebase origin $(git branch --show-current) 2>/dev/null || true
```

**检查点**: 在正确的分支上。工作树已准备好。远程已同步。

---

## 阶段 3 — 执行

顺序处理计划中的每个任务。

### 每个任务的循环

对于**分步任务**中的每个任务：

1. **读取 MIRROR 引用** — 打开任务的 MIRROR 字段中引用的模式文件。在编写代码之前理解约定。

2. **实现** — 严格遵循模式编写代码。应用 GOTCHA 警告。使用指定的 IMPORTS。

3. **立即验证** — 每次文件变更后：
   ```bash
   # 运行 type-check（根据项目调整命令）
   [阶段 0 中的 type-check 命令]
   ```
   如果 type-check 失败 → 在移动到下一个文件之前修复错误。

4. **跟踪进度** — 记录：`[done] 任务 N: [任务名称] — 完成`

### 处理偏差

如果实现必须偏离计划：
- 记录**什么**改变了
- 记录**为什么**改变
- 用修正后的方法继续
- 这些偏差将被捕获在报告中

**检查点**: 所有任务已执行。偏差已记录。

---

## 阶段 4 — 验证

运行计划中的所有验证级别。在继续之前修复每个级别的问题。

### 级别 1: 静态分析

```bash
# 类型检查 — 要求零错误
[项目 type-check 命令]

# linting — 尽可能自动修复
[项目 lint 命令]
[项目 lint-fix 命令]
```

如果自动修复后仍有 lint 错误，手动修复。

### 级别 2: 单元测试

为每个新函数编写测试（如计划的测试策略中指定）。

```bash
[项目针对受影响区域的 test 命令]
```

- 每个函数至少需要一个测试
- 覆盖计划中列出的边界情况
- 如果测试失败 → 修复实现（而不是测试，除非测试本身错误）

### 级别 3: 构建检查

```bash
[项目 build 命令]
```

构建必须成功，零错误。

### 级别 4: 集成测试（如适用）

```bash
# 启动服务器，运行测试，停止服务器
[项目 dev server 命令] &
SERVER_PID=$!

# 等待服务器就绪（根据需要调整端口）
SERVER_READY=0
for i in $(seq 1 30); do
  if curl -sf http://localhost:PORT/health >/dev/null 2>&1; then
    SERVER_READY=1
    break
  fi
  sleep 1
done

if [ "$SERVER_READY" -ne 1 ]; then
  kill "$SERVER_PID" 2>/dev/null || true
  echo "错误：服务器在 30 秒内未能启动" >&2
  exit 1
fi

[集成 test 命令]
TEST_EXIT=$?

kill "$SERVER_PID" 2>/dev/null || true
wait "$SERVER_PID" 2>/dev/null || true

exit "$TEST_EXIT"
```

### 级别 5: 边界情况测试

运行计划的测试策略清单中的边界情况。

**检查点**: 所有 5 个验证级别通过。零错误。

---

## 阶段 5 — 报告

### 创建实现报告

```bash
mkdir -p .claude/PRPs/reports
```

将报告写入 `.claude/PRPs/reports/{plan-name}-report.md`：

```markdown
# 实现报告：[功能名称]

## 摘要
[实现了什么]

## 评估与现实对比

| 指标 | 预测（计划） | 实际 |
|---|---|---|
| 复杂度 | [来自计划] | [实际] |
| 信心 | [来自计划] | [实际] |
| 更改的文件 | [来自计划] | [实际数量] |

## 已完成的任务

| # | 任务 | 状态 | 备注 |
|---|---|---|---|
| 1 | [任务名称] | [done] 完成 | |
| 2 | [任务名称] | [done] 完成 | 偏差 — [原因] |

## 验证结果

| 级别 | 状态 | 备注 |
|---|---|---|
| 静态分析 | [done] 通过 | |
| 单元测试 | [done] 通过 | 编写了 N 个测试 |
| 构建 | [done] 通过 | |
| 集成 | [done] 通过 | 或 N/A |
| 边界情况 | [done] 通过 | |

## 更改的文件

| 文件 | 操作 | 行数 |
|---|---|---|
| `path/to/file` | 已创建 | +N |
| `path/to/file` | 已更新 | +N / -M |

## 与计划的偏差
[列出任何偏差，包括什么和为什么，或"无"]

## 遇到的问题
[列出任何问题以及它们是如何解决的，或"无"]

## 编写的测试

| 测试文件 | 测试数 | 覆盖率 |
|---|---|---|
| `path/to/test` | N 个测试 | [覆盖的区域] |

## 下一步
- [ ] 通过 `/code-review` 进行代码审查
- [ ] 通过 `/prp-pr` 创建 PR
```

### 更新 PRD（如适用）

如果此实现是针对 PRD 阶段的：
1. 将阶段状态从 `in-progress` 更新为 `complete`
2. 添加报告路径作为参考

### 归档计划

```bash
mkdir -p .claude/PRPs/plans/completed
mv "$ARGUMENTS" .claude/PRPs/plans/completed/
```

**检查点**: 报告已创建。PRD 已更新。计划已归档。

---

## 阶段 6 — 输出

向用户报告：

```
## 实现完成

- **计划**: [计划文件路径] → 已归档到 completed/
- **分支**: [当前分支名称]
- **状态**: [done] 所有任务完成

### 验证摘要

| 检查 | 状态 |
|---|---|
| Type Check | [done] |
| Lint | [done] |
| Tests | [done]（编写了 N 个） |
| Build | [done] |
| Integration | [done] 或 N/A |

### 更改的文件
- 创建了 [N] 个文件，更新了 [M] 个文件

### 偏差
[摘要或"无 — 完全按计划实现"]

### 工件
- 报告：`.claude/PRPs/reports/{name}-report.md`
- 已归档的计划：`.claude/PRPs/plans/completed/{name}.plan.md`

### PRD 进度（如适用）
| 阶段 | 状态 |
|---|---|
| 阶段 1 | [done] 完成 |
| 阶段 2 | [next] |
| ... | ... |

> 下一步：运行 `/prp-pr` 创建拉取请求，或先运行 `/code-review` 审查更改。
```

---

## 处理失败

### Type Check 失败
1. 仔细阅读错误消息
2. 修复源文件中的类型错误
3. 重新运行 type-check
4. 仅在干净时继续

### 测试失败
1. 确定 bug 是在实现中还是在测试中
2. 修复根本原因（通常是实现）
3. 重新运行测试
4. 仅在绿色时继续

### Lint 失败
1. 先运行自动修复
2. 如果仍有错误，手动修复
3. 重新运行 lint
4. 仅在干净时继续

### 构建失败
1. 通常是类型或导入问题 — 检查错误消息
2. 修复有问题的文件
3. 重新运行构建
4. 仅在成功时继续

### 集成测试失败
1. 检查服务器是否正确启动
2. 验证端点/路由是否存在
3. 检查请求格式是否符合预期
4. 修复并重新运行

---

## 成功标准

- **TASKS_COMPLETE**: 计划中的所有任务已执行
- **TYPES_PASS**: 零类型错误
- **LINT_PASS**: 零 lint 错误
- **TESTS_PASS**: 所有测试通过，编写了新测试
- **BUILD_PASS**: 构建成功
- **REPORT_CREATED**: 实现报告已保存
- **PLAN_ARCHIVED**: 计划已移动到 `completed/`

---

## 下一步

- 运行 `/code-review` 在提交前审查更改
- 运行 `/prp-commit` 提交具有描述性消息的更改
- 运行 `/prp-pr` 创建拉取请求
- 如果 PRD 有更多阶段，运行 `/prp-plan <next-phase>`
