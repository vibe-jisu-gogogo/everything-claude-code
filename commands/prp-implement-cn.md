---
description: 通过严格的验证循环执行实施计划
argument-hint: <path/to/plan.md>
---

> 改编自 Wirasm 的 PRPs-agentic-eng。PRP 工作流系列的一部分。

# PRP 实施

逐步执行计划文件并持续验证。每项更改都立即验证 — 绝不累积损坏状态。

**核心理念**：验证循环及早发现错误。每次更改后运行检查。立即修复问题。

**黄金法则**：如果验证失败，请先修复再继续。绝不累积损坏状态。

---

## 阶段 0 — 检测

### 包管理器检测

| 文件存在 | 包管理器 | 运行器 |
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

记录可用命令：type-check、lint、test、build。

---

## 阶段 1 — 加载

读取计划文件：

```bash
cat "$ARGUMENTS"
```

从计划中提取以下部分：
- **摘要** — 正在构建什么
- **需要镜像的模式** — 要遵循的代码约定
- **要更改的文件** — 要创建或修改的内容
- **逐步任务** — 实施顺序
- **验证命令** — 如何验证正确性
- **验收标准** — 完成定义

如果文件不存在或不是有效计划：
```
错误：未找到计划文件或文件无效。
先运行 /prp-plan <feature-description> 创建计划。
```

**检查点**：计划已加载。所有部分已识别。任务已提取。

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
| 在 main 上，工作区干净 | 创建功能分支：`git checkout -b feat/{plan-name}` |
| 在 main 上，工作区脏 | **停止** — 要求用户先 stash 或 commit |
| 在用于此功能的 git worktree 中 | 使用该 worktree |

### 同步远程

```bash
git pull --rebase origin $(git branch --show-current) 2>/dev/null || true
```

**检查点**：在正确的分支上。工作区已准备就绪。远程已同步。

---

## 阶段 3 — 执行

按顺序处理计划中的每个任务。

### 每个任务的循环

对于**逐步任务**中的每个任务：

1. **读取镜像参考** — 打开任务的 MIRROR 字段中引用的模式文件。编写代码前先理解约定。

2. **实施** — 严格遵循模式编写代码。应用 GOTCHA 警告。使用指定的 IMPORTS。

3. **立即验证** — 每次文件更改后：
   ```bash
   # 运行 type-check（根据项目调整命令）
   [来自阶段 0 的 type-check 命令]
   ```
   如果 type-check 失败 → 先修复错误再继续下一个文件。

4. **跟踪进度** — 记录：`[done] 任务 N: [任务名称] — 完成`

### 处理偏差

如果实施必须偏离计划：
- 记录**更改了什么**
- 记录**为什么**更改
- 继续使用更正后的方法
- 这些偏差将在报告中捕获

**检查点**：所有任务已执行。偏差已记录。

---

## 阶段 4 — 验证

运行计划中的所有验证级别。在继续之前修复每个级别的问题。

### 级别 1：静态分析

```bash
# 类型检查 — 要求零错误
[项目 type-check 命令]

# linting — 尽可能自动修复
[项目 lint 命令]
[项目 lint-fix 命令]
```

如果自动修复后仍有 lint 错误，请手动修复。

### 级别 2：单元测试

为每个新函数编写测试（如计划的测试策略中所指定）。

```bash
[受影响区域的项目 test 命令]
```

- 每个函数至少需要一个测试
- 覆盖计划中列出的边界情况
- 如果测试失败 → 修复实施（不是测试，除非测试有误）

### 级别 3：构建检查

```bash
[项目 build 命令]
```

构建必须成功，零错误。

### 级别 4：集成测试（如适用）

```bash
# 启动服务器，运行测试，停止服务器
[项目 dev server 命令] &
SERVER_PID=$!

# 等待服务器准备就绪（根据需要调整端口）
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

[集成测试命令]
TEST_EXIT=$?

kill "$SERVER_PID" 2>/dev/null || true
wait "$SERVER_PID" 2>/dev/null || true

exit "$TEST_EXIT"
```

### 级别 5：边界情况测试

遍历计划测试策略清单中的边界情况。

**检查点**：所有 5 个验证级别通过。零错误。

---

## 阶段 5 — 报告

### 创建实施报告

```bash
mkdir -p .claude/PRPs/reports
```

将报告写入 `.claude/PRPs/reports/{plan-name}-report.md`：

```markdown
# 实施报告：[功能名称]

## 摘要
[实施了什么]

## 预测与实际评估

| 指标 | 预测（计划） | 实际 |
|---|---|---|
| 复杂度 | [来自计划] | [实际] |
| 信心 | [来自计划] | [实际] |
| 更改文件数 | [来自计划] | [实际数量] |

## 已完成任务

| # | 任务 | 状态 | 备注 |
|---|---|---|---|
| 1 | [任务名称] | [done] 完成 | |
| 2 | [任务名称] | [done] 完成 | 偏差 — [原因] |

## 验证结果

| 级别 | 状态 | 备注 |
|---|---|---|
| 静态分析 | [done] 通过 | |
| 单元测试 | [done] 通过 | 已编写 N 个测试 |
| 构建 | [done] 通过 | |
| 集成 | [done] 通过 | 或 N/A |
| 边界情况 | [done] 通过 | |

## 更改的文件

| 文件 | 操作 | 行数 |
|---|---|---|
| `path/to/file` | 已创建 | +N |
| `path/to/file` | 已更新 | +N / -M |

## 与计划的偏差
[列出任何偏差及其内容和原因，或"无"]

## 遇到的问题
[列出任何问题及其解决方式，或"无"]

## 编写的测试

| 测试文件 | 测试数量 | 覆盖率 |
|---|---|---|
| `path/to/test` | N 个测试 | [覆盖区域] |

## 下一步
- [ ] 通过 `/code-review` 进行代码审查
- [ ] 通过 `/prp-pr` 创建 PR
```

### 更新 PRD（如适用）

如果此实施是针对 PRD 阶段的：
1. 将阶段状态从 `in-progress` 更新为 `complete`
2. 添加报告路径作为参考

### 归档计划

```bash
mkdir -p .claude/PRPs/plans/completed
mv "$ARGUMENTS" .claude/PRPs/plans/completed/
```

**检查点**：报告已创建。PRD 已更新。计划已归档。

---

## 阶段 6 — 输出

向用户报告：

```
## 实施完成

- **计划**：[计划文件路径] → 已归档到 completed/
- **分支**：[当前分支名称]
- **状态**：[done] 所有任务已完成

### 验证摘要

| 检查 | 状态 |
|---|---|
| Type Check | [done] |
| Lint | [done] |
| Tests | [done]（已编写 N 个） |
| Build | [done] |
| Integration | [done] 或 N/A |

### 更改的文件
- 已创建 [N] 个文件，已更新 [M] 个文件

### 偏差
[摘要或"无 — 完全按计划实施"]

### 工件
- 报告：`.claude/PRPs/reports/{name}-report.md`
- 已归档计划：`.claude/PRPs/plans/completed/{name}.plan.md`

### PRD 进度（如适用）
| 阶段 | 状态 |
|---|---|
| 阶段 1 | [done] 完成 |
| 阶段 2 | [next] |
| ... | ... |

> 下一步：运行 `/prp-pr` 创建 pull request，或先运行 `/code-review` 审查更改。
```

---

## 处理失败

### Type Check 失败
1. 仔细阅读错误消息
2. 修复源文件中的类型错误
3. 重新运行 type-check
4. 仅在无错误时继续

### 测试失败
1. 确定错误是在实施中还是在测试中
2. 修复根本原因（通常是实施）
3. 重新运行测试
4. 仅在通过时继续

### Lint 失败
1. 先运行自动修复
2. 如果仍有错误，手动修复
3. 重新运行 lint
4. 仅在无错误时继续

### 构建失败
1. 通常是类型或导入问题 — 检查错误消息
2. 修复有问题的文件
3. 重新运行构建
4. 仅在成功时继续

### 集成测试失败
1. 检查服务器是否正确启动
2. 验证 endpoint/route 存在
3. 检查请求格式是否符合预期
4. 修复并重新运行

---

## 成功标准

- **TASKS_COMPLETE**：计划中的所有任务已执行
- **TYPES_PASS**：零类型错误
- **LINT_PASS**：零 lint 错误
- **TESTS_PASS**：所有测试通过，已编写新测试
- **BUILD_PASS**：构建成功
- **REPORT_CREATED**：实施报告已保存
- **PLAN_ARCHIVED**：计划已移至 `completed/`

---

## 下一步

- 运行 `/code-review` 在提交前审查更改
- 运行 `/prp-commit` 使用描述性消息提交
- 运行 `/prp-pr` 创建 pull request
- 如果 PRD 有更多阶段，运行 `/prp-plan <next-phase>`
