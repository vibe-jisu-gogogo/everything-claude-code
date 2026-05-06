# Session Adapter Contract

本文档定义了 `ecc.session.v1` 的标准 ECC 会话快照契约。

该契约在 `scripts/lib/session-adapters/canonical-session.js` 中实现。本文档是适配器和消费者的规范性规范。

## 目的

ECC 有多个会话来源：

- tmux 编排的 worktree 会话
- Claude 本地会话历史
- 未来的 harness 和控制平面后端

适配器将这些来源标准化为一种控制平面安全的快照格式，因此检查、持久化和未来的 UI 层不依赖于 harness 特定的文件或运行时细节。

## 标准快照

每个适配器必须返回一个可 JSON 序列化的对象，具有以下顶层结构：

```json
{
  "schemaVersion": "ecc.session.v1",
  "adapterId": "dmux-tmux",
  "session": {
    "id": "workflow-visual-proof",
    "kind": "orchestrated",
    "state": "active",
    "repoRoot": "/tmp/repo",
    "sourceTarget": {
      "type": "session",
      "value": "workflow-visual-proof"
    }
  },
  "workers": [
    {
      "id": "seed-check",
      "label": "seed-check",
      "state": "running",
      "health": "healthy",
      "branch": "feature/seed-check",
      "worktree": "/tmp/worktree",
      "runtime": {
        "kind": "tmux-pane",
        "command": "codex",
        "pid": 1234,
        "active": false,
        "dead": false
      },
      "intent": {
        "objective": "Inspect seeded files.",
        "seedPaths": ["scripts/orchestrate-worktrees.js"]
      },
      "outputs": {
        "summary": [],
        "validation": [],
        "remainingRisks": []
      },
      "artifacts": {
        "statusFile": "/tmp/status.md",
        "taskFile": "/tmp/task.md",
        "handoffFile": "/tmp/handoff.md"
      }
    }
  ],
  "aggregates": {
    "workerCount": 1,
    "states": {
      "running": 1
    },
    "healths": {
      "healthy": 1
    }
  }
}
```

## 必需字段

### 顶层

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `schemaVersion` | string | 此契约必须精确为 `ecc.session.v1` |
| `adapterId` | string | 稳定的适配器标识符，如 `dmux-tmux` 或 `claude-history` |
| `session` | object | 标准会话元数据 |
| `workers` | array | 标准 worker 记录；可以为空 |
| `aggregates` | object | 派生的 worker 计数 |

### `session`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 适配器域内的稳定标识符 |
| `kind` | string | 高级会话类别，如 `orchestrated` 或 `history` |
| `state` | string | 标准会话状态 |
| `sourceTarget` | object | 打开会话的目标的来源信息 |

### `session.sourceTarget`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `type` | string | 查找类别，如 `plan`、`session`、`claude-history`、`claude-alias` 或 `session-file` |
| `value` | string | 原始目标值或解析后的路径 |

### `workers[]`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 适配器范围内的稳定 worker 标识符 |
| `label` | string | 面向操作者的标签 |
| `state` | string | 标准 worker 状态（生命周期） |
| `health` | string | 标准 worker 健康状况（运行条件） |
| `runtime` | object | 执行/运行时元数据 |
| `intent` | object | 此 worker/会话存在的原因 |
| `outputs` | object | 结构化的结果和检查 |
| `artifacts` | object | 适配器拥有的文件/路径引用 |

### `workers[].runtime`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `kind` | string | 运行时类别，如 `tmux-pane` 或 `claude-session` |
| `active` | boolean | 运行时当前是否处于活动状态 |
| `dead` | boolean | 运行时是否已知已死亡/完成 |

### `workers[].intent`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `objective` | string | 主要目标或标题 |
| `seedPaths` | string[] | 与 worker/会话关联的 seed 或上下文路径 |

### `workers[].outputs`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `summary` | string[] | 已完成的输出或摘要项 |
| `validation` | string[] | 验证证据或检查 |
| `remainingRisks` | string[] | 未解决的风险、后续行动或备注 |

### `aggregates`

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `workerCount` | integer | 必须等于 `workers.length` |
| `states` | object | 从 `workers[].state` 派生的计数映射 |
| `healths` | object | 从 `workers[].health` 派生的计数映射 |

## 可选字段

可选字段可以省略，但如果发出，必须保留文档中记录的类型：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `session.repoRoot` | `string \| null` | 已知时的仓库/worktree 根目录 |
| `workers[].branch` | `string \| null` | 已知时的分支名称 |
| `workers[].worktree` | `string \| null` | 已知时的 worktree 路径 |
| `workers[].runtime.command` | `string \| null` | 已知时的活动命令 |
| `workers[].runtime.pid` | `number \| null` | 已知时的进程 ID |
| `workers[].artifacts.*` | 适配器定义 | 适配器拥有的文件路径或结构化引用 |

适配器特定的可选字段属于 `runtime`、`artifacts` 或其他文档化的嵌套对象内。适配器不得在未更新此契约的情况下发明新的顶层字段。

## 状态语义

契约有意保持 `session.state` 和 `workers[].state` 足够灵活，适用于多个 harness，但当前适配器使用这些值：

- `dmux-tmux`
  - 会话状态：`active`、`completed`、`failed`、`idle`、`missing`
  - worker 状态：从 worker 状态文件派生，例如 `running` 或 `completed`
- `claude-history`
  - 会话状态：`recorded`
  - worker 状态：`recorded`

消费者必须将未知的状态字符串视为有效的适配器特定值并优雅降级。

## 版本控制策略

`schemaVersion` 是唯一的兼容性门控。消费者必须基于它进行分支。

### 在 `ecc.session.v1` 中允许的操作

- 添加新的可选嵌套字段
- 添加新的适配器 ID
- 添加新的状态字符串值
- 添加新的健康字符串值
- 在 `workers[].artifacts` 内添加新的 artifact 键

### 需要新模式版本的操作

- 删除必需字段
- 重命名字段
- 更改字段类型
- 以不兼容的方式更改现有字段的含义
- 保持相同版本字符串的同时将数据从一个字段移动到另一个字段

如果发生以上任何情况，生产者必须发出新的版本字符串，如 `ecc.session.v2`。

## 适配器合规性要求

每个 ECC 会话适配器必须：

1. 精确发出 `schemaVersion: "ecc.session.v1"`。
2. 返回满足所有必需字段和类型的快照。
3. 对于未知的可选标量值使用 `null`，对于未知的列表值使用空数组。
4. 将适配器特定的细节嵌套在 `runtime`、`artifacts` 或其他文档化的嵌套对象下。
5. 确保 `aggregates.workerCount === workers.length`。
6. 确保 `aggregates.states` 与发出的 worker 状态匹配。
7. 确保 `aggregates.healths` 与发出的 worker 健康值匹配。
8. 仅生成纯 JSON 可序列化值。
9. 在持久化或下游使用之前验证标准结构。
10. 通过会话记录垫片持久化规范化的标准快照。在此仓库中，该垫片首先尝试 `scripts/lib/state-store`，仅当状态存储模块尚不可用时才回退到 JSON 记录文件。

## 消费者期望

消费者应该：

- 仅依赖 `ecc.session.v1` 的文档化字段
- 忽略未知的可选字段
- 将 `adapterId`、`session.kind` 和 `runtime.kind` 视为路由提示，而非详尽的枚举
- 期望 `workers[].artifacts` 内有适配器特定的 artifact 键

消费者不得：

- 从未文档化的字段推断 harness 特定的行为
- 假设所有适配器都有 tmux pane、git worktree 或 markdown 协调文件
- 仅因为状态字符串不熟悉而拒绝快照

## 当前适配器映射

### `dmux-tmux`

- 来源：`scripts/lib/orchestration-session.js`
- 会话 ID：编排会话名称
- 会话 kind：`orchestrated`
- 会话源目标：计划路径或会话名称
- Worker runtime kind：`tmux-pane`
- Artifacts：`statusFile`、`taskFile`、`handoffFile`

### `claude-history`

- 来源：`scripts/lib/session-manager.js`
- 会话 ID：存在时使用 Claude short id，否则使用会话文件名派生的 ID
- 会话 kind：`history`
- 会话源目标：显式历史目标、别名或 `.tmp` 会话文件
- Worker runtime kind：`claude-session`
- Intent seed 路径：从 `### Context to Load` 解析
- Artifacts：`sessionFile`、`context`

## 验证参考

仓库实现验证：

- 必需的对象结构
- 必需的字符串字段
- 布尔运行时标志
- 字符串数组输出和 seed 路径
- 聚合计数一致性

适配器应将验证失败视为契约错误，而非用户输入错误。

## 记录回退行为

JSON 回退记录器是在专用状态存储落地之前的临时兼容性垫片。其行为是：

- 最新快照始终原地替换
- 历史记录仅记录不同的快照主体
- 未更改的重复读取不会追加重复的历史条目

这防止了 `session-inspect` 和其他轮询式读取为同一未更改的会话快照增长无限制的历史记录。
