# ECC 2.0 Selective Install Discovery

## Purpose

本文档将 3 月 11 日 mega-plan 中的 selective-install 需求转化为具体的 ECC 2.0 discovery 设计。

目标不仅仅是"安装时复制更少的文件"。实际目标是建立一个能够确定性回答以下问题的安装系统：

- 请求了什么内容
- 解析了什么内容
- 复制或生成了什么内容
- 应用了哪些目标特定的转换
- ECC 拥有什么内容，并且后续可以安全地移除或修复

这正是 ECC 1.x 安装系统与 ECC 2.0 控制平面之间所缺失的契约。

## Current Implemented Foundation

首个 selective-install 基础架构已存在于仓库中：

- `manifests/install-modules.json`
- `manifests/install-profiles.json`
- `schemas/install-modules.schema.json`
- `schemas/install-profiles.schema.json`
- `schemas/install-state.schema.json`
- `scripts/ci/validate-install-manifests.js`
- `scripts/lib/install-manifests.js`
- `scripts/lib/install/request.js`
- `scripts/lib/install/runtime.js`
- `scripts/lib/install/apply.js`
- `scripts/lib/install-targets/`
- `scripts/lib/install-state.js`
- `scripts/lib/install-executor.js`
- `scripts/lib/install-lifecycle.js`
- `scripts/ecc.js`
- `scripts/install-apply.js`
- `scripts/install-plan.js`
- `scripts/list-installed.js`
- `scripts/doctor.js`

当前能力：

- 机器可读的 module 和 profile catalog
- CI 验证 manifest 条目指向真实的仓库路径
- 依赖扩展和目标过滤
- 适配器感知的操作规划
- 针对 legacy 和 manifest 安装模式的规范化请求标准化
- 从标准化请求到计划创建的显式 runtime dispatch
- legacy 和 manifest 安装都会写入持久化的 install-state
- 在任何变更前对安装计划进行只读检查
- 统一的 `ecc` CLI 路由安装、规划和生命周期命令
- 通过 `list-installed`、`doctor`、`repair` 和 `uninstall` 进行生命周期检查和变更

当前限制：

- 某些 modules 的目标特定合并/移除语义仍处于 scaffold 级别
- legacy `ecc-install` 兼容性仍指向 `install.sh`
- `package.json` 中的发布范围仍然过广

## Current Code Review

当前的安装程序栈已经比最初的 language-first shell 安装程序健康得多，但它仍然在少数文件中集中了过多责任。

### Current Runtime Path

当前的 runtime 流程是：

1. `install.sh`
   解析真实包根目录的精简 shell 包装器
2. `scripts/install-apply.js`
   面向用户的 legacy 和 manifest 模式安装 CLI
3. `scripts/lib/install/request.js`
   CLI 解析和标准化请求规范化
4. `scripts/lib/install/runtime.js`
   从标准化请求到安装计划的 runtime dispatch
5. `scripts/lib/install-executor.js`
   参数转换、legacy 兼容性、操作具体化、文件系统变更和 install-state 写入
6. `scripts/lib/install-manifests.js`
   module/profile catalog 加载和依赖扩展
7. `scripts/lib/install-targets/`
   目标根目录和目标路径 scaffold
8. `scripts/lib/install-state.js`
   schema 支持的 install-state 读写
9. `scripts/lib/install-lifecycle.js`
   从存储的操作派生的 doctor/repair/uninstall 行为

这足以证明 selective-install 基础架构的可行性，但还不足以让安装程序架构感觉稳定。

### Current Strengths

- 安装意图现在通过 `--profile` 和 `--modules` 显式表达
- 请求解析和请求规范化已从 CLI shell 中分离
- 目标根目录解析已适配器化
- 生命周期命令现在使用持久化的 install-state 而非猜测
- 仓库已通过 `ecc` 和 `install-apply.js` 拥有统一的 Node 入口点

### Current Coupling Still Present

1. `install-executor.js` 比以前小了，但仍同时承载了过多的规划和具体化层。
   请求边界已提取，但 legacy 请求转换、manifest-plan 扩展和操作具体化仍然共存。
2. target adapters 仍然过于单薄。
   目前它们主要解析根目录和构建目标路径。真正的安装语义仍存在于 executor 分支和路径启发式中。
3. planner/executor 边界还不够清晰。
   `install-manifests.js` 解析 modules，但最终的安装操作集仍部分在 executor 特定逻辑中构建。
4. 生命周期行为更多依赖于低级记录的操作，而非稳定的 module 语义。
   这对于纯文件复制有效，但对于 merge/generate/remove 行为会变得脆弱。
5. compatibility mode 直接混入主安装程序 runtime。
   Legacy language 安装应该表现为请求适配器，而非并行安装程序架构。

## Proposed Modular Architecture Changes

下一个架构步骤是将安装程序分离为明确的层，每层返回稳定数据而非立即变更文件。

### Target State

期望的安装管道是：

1. CLI 界面
2. 请求规范化
3. module 解析
4. target 规划
5. 操作规划
6. 执行
7. install-state 持久化
8. 基于相同操作契约构建的生命周期服务

主要思想很简单：

- manifests 描述内容
- adapters 描述目标特定的落地语义
- planners 描述应该发生什么
- executors 应用这些计划
- 生命周期命令重用相同的计划/状态模型，而非重新发明

### Proposed Runtime Layers

#### 1. CLI Surface

职责：

- 仅解析用户意图
- 路由到 install、plan、doctor、repair、uninstall
- 渲染人类可读或 JSON 输出

不应拥有：

- legacy language 转换
- target 特定的安装规则
- 操作构建

建议文件：

```text
scripts/ecc.js
scripts/install-apply.js
scripts/install-plan.js
scripts/doctor.js
scripts/repair.js
scripts/uninstall.js
```

这些保留为入口点，但成为围绕库模块的薄包装器。

#### 2. Request Normalizer

职责：

- 将原始 CLI 标志转换为标准化安装请求
- 将 legacy language 安装转换为兼容性请求格式
- 尽早拒绝混合或模糊的输入

建议的标准化请求：

```json
{
  "mode": "manifest",
  "target": "cursor",
  "profile": "developer",
  "modules": [],
  "legacyLanguages": [],
  "dryRun": false
}
```

或者，在兼容性模式下：

```json
{
  "mode": "legacy-compat",
  "target": "claude",
  "profile": null,
  "modules": [],
  "legacyLanguages": ["typescript", "python"],
  "dryRun": false
}
```

这使管道的其余部分可以忽略请求来自旧的还是新的 CLI 语法。

#### 3. Module Resolver

职责：

- 加载 manifest catalogs
- 扩展依赖
- 拒绝冲突
- 按目标过滤不支持的 modules
- 返回标准化解析对象

这一层应保持纯碎和只读。

它不应知道：

- 目标文件系统路径
- 合并语义
- 复制策略

当前最近的文件：

- `scripts/lib/install-manifests.js`

建议拆分：

```text
scripts/lib/install/catalog.js
scripts/lib/install/resolve-request.js
scripts/lib/install/resolve-modules.js
```

#### 4. Target Planner

职责：

- 选择安装目标适配器
- 解析目标根目录
- 解析 install-state 路径
- 扩展 module-to-target 映射规则
- 发出目标感知的操作意图

这就是目标特定含义应该存在的地方。

示例：

- Claude 可能在 `~/.claude` 下保留原生层次结构
- Cursor 可能以不同于 rules 的方式同步捆绑的 `.cursor` 根子项
- 生成的配置可能需要根据目标进行合并或替换语义

当前最近的文件：

- `scripts/lib/install-targets/helpers.js`
- `scripts/lib/install-targets/registry.js`

建议演进：

```text
scripts/lib/install/targets/registry.js
scripts/lib/install/targets/claude-home.js
scripts/lib/install/targets/cursor-project.js
scripts/lib/install/targets/antigravity-project.js
```

每个适配器最终应暴露比 `resolveRoot` 更多的功能。它应该为其目标系列拥有路径和策略映射。

#### 5. Operation Planner

职责：

- 将 module 解析和适配器规则转换为类型化操作图
- 发出一等操作，例如：
  - `copy-file`
  - `copy-tree`
  - `merge-json`
  - `render-template`
  - `remove`
- 附加所有权和验证元数据

这是当前安装程序中缺失的架构接缝。

今天，操作部分处于 scaffold 级别，部分是 executor 特定的。ECC 2.0 应使操作规划成为独立阶段，以便：

- `plan` 成为执行的真实预览
- `doctor` 可以验证预期行为，而不仅仅是当前文件
- `repair` 可以安全地重建确切的缺失工作
- `uninstall` 可以仅撤销托管操作

#### 6. Execution Engine

职责：

- 应用类型化操作图
- 强制执行覆盖和所有权规则
- 安全地暂存写入
- 收集最终的已应用操作结果

这一层不应决定做什么。它只应决定如何安全地应用提供的操作类型。

当前最近的文件：

- `scripts/lib/install-executor.js`

建议的重构：

```text
scripts/lib/install/executor/apply-plan.js
scripts/lib/install/executor/apply-copy.js
scripts/lib/install/executor/apply-merge-json.js
scripts/lib/install/executor/apply-remove.js
```

这将 executor 逻辑从一个大的分支 runtime 转变为一组小的操作处理器。

#### 7. Install-State Store

职责：

- 验证并持久化 install-state
- 记录标准化请求、解析和已应用的操作
- 支持生命周期命令而不强迫它们反向工程安装

当前最近的文件：

- `scripts/lib/install-state.js`

这一层已接近正确的形态。主要的剩余变更是在 merge/generate 语义实现后存储更丰富的操作元数据。

#### 8. Lifecycle Services

职责：

- `list-installed`：仅检查状态
- `doctor`：比较期望/install-state 视图与当前文件系统
- `repair`：从状态重新生成计划并重新应用安全操作
- `uninstall`：仅移除 ECC 拥有的输出

当前最近的文件：

- `scripts/lib/install-lifecycle.js`

这一层最终应基于操作类型和所有权策略运行，而不仅仅是原始的 `copy-file` 记录。

## Proposed File Layout

干净的模块化最终状态大致如下：

```text
scripts/lib/install/
  catalog.js
  request.js
  resolve-modules.js
  plan-operations.js
  state-store.js
  targets/
    registry.js
    claude-home.js
    cursor-project.js
    antigravity-project.js
    codex-home.js
    opencode-home.js
  executor/
    apply-plan.js
    apply-copy.js
    apply-merge-json.js
    apply-render-template.js
    apply-remove.js
  lifecycle/
    discover.js
    doctor.js
    repair.js
    uninstall.js
```

这不是打包拆分。这是当前仓库内的代码所有权拆分，以便每层只有一个职责。

## Migration Map From Current Files

最低风险的迁移路径是渐进式的，而非重写。

### Keep

- `install.sh` 作为公共兼容性垫片
- `scripts/ecc.js` 作为统一 CLI
- `scripts/lib/install-state.js` 作为状态存储的起点
- 当前的目标适配器 ID 和状态位置

### Extract

- 从 `scripts/lib/install-executor.js` 中提取请求解析和兼容性转换
- 从 executor 分支中提取目标感知的操作规划，放入 target adapters 和 planner modules
- 从共享的生命周期单体中提取生命周期特定的分析，放入更小的服务

### Replace Gradually

- 用类型化操作替换广泛的路径复制启发式
- 用适配器拥有的语义替换仅 scaffold 的适配器规划
- 用 legacy 请求转换替换 legacy language 安装分支，进入相同的 planner/executor 管道

## Immediate Architecture Changes To Make Next

如果目标是 ECC 2.0 而不仅仅是"够用"，下一步的模块化步骤应该是：

1. 将 `install-executor.js` 拆分为请求规范化、操作规划和执行模块
2. 将目标特定的策略决策移动到适配器拥有的规划方法中
3. 使 `repair` 和 `uninstall` 基于类型化操作处理器运行，而不仅仅是普通的 `copy-file` 记录
4. 让 manifests 了解安装策略和所有权，以便 planner 不再依赖路径启发式
5. 仅在内部模块边界稳定后缩小 npm 发布范围

## Why The Current Model Is Not Enough

今天 ECC 的行为仍然像一个广泛的 payload 复制器：

- `install.sh` 是 language-first 且 target-branch-heavy
- targets 部分隐含在目录布局中
- uninstall、repair 和 doctor 现在存在，但仍是早期的生命周期命令
- 仓库无法证明先前安装实际写入了什么
- `package.json` 中的发布范围仍然过广

这造成了 mega plan 中已经指出的问题：

- 用户拉取比其 harness 或工作流需要更多的内容
- 支持和升级更加困难，因为安装未被记录
- 目标行为漂移，因为安装逻辑在 shell 分支中重复
- 未来的目标如 Codex 或 OpenCode 需要更多的特殊情况逻辑，而非重用稳定的安装契约

## ECC 2.0 Design Thesis

Selective install 应建模为：

1. 将请求的意图解析为标准化模块图
2. 通过目标适配器转换该图
3. 执行确定性的安装操作集
4. 将 install-state 写入作为持久的真相来源

这意味着 ECC 2.0 需要两个契约，而非一个：

- 内容契约
  存在哪些 modules 以及它们如何相互依赖
- 目标契约
  这些 modules 如何落地到 Claude、Cursor、Antigravity、Codex 或 OpenCode

当前仓库只有早期形式的前半部分。
当前仓库现在有第一个完整的垂直切片，但没有完整的目标特定语义。

## Design Constraints

1. 保留 `everything-claude-code` 作为规范的源仓库。
2. 在迁移期间保留现有的 `install.sh` 流程。
3. 从同一个 planner 支持 home-scoped 和 project-scoped 的 targets。
4. 无需猜测即可实现 uninstall/repair/doctor。
5. 避免 per-target 复制逻辑泄漏回 module 定义。
6. 保持未来的 Codex 和 OpenCode 支持是附加的，而非重写。

## Canonical Artifacts

### 1. Module Catalog

Module catalog 是规范的内容图。

已实现的当前字段：

- `id`
- `kind`
- `description`
- `paths`
- `targets`
- `dependencies`
- `defaultInstall`
- `cost`
- `stability`

ECC 2.0 仍需要的字段：

- `installStrategy`
  例如 `copy`、`flatten-rules`、`generate`、`merge-config`
- `ownership`
  ECC 是否完全拥有目标路径，或仅拥有其下的生成文件
- `pathMode`
  例如 `preserve`、`flatten`、`target-template`
- `conflicts`
  无法在一个目标上共存的 modules 或路径族
- `publish`
  module 是否默认打包、可选或安装后生成

建议的未来形态：

```json
{
  "id": "hooks-runtime",
  "kind": "hooks",
  "paths": ["hooks", "scripts/hooks"],
  "targets": ["claude", "cursor", "opencode"],
  "dependencies": [],
  "installStrategy": "copy",
  "pathMode": "preserve",
  "ownership": "managed",
  "defaultInstall": true,
  "cost": "medium",
  "stability": "stable"
}
```

### 2. Profile Catalog

Profiles 保持精简。

它们应表达用户意图，而非复制目标逻辑。

已实现的当前示例：

- `core`
- `developer`
- `security`
- `research`
- `full`

仍需要的字段：

- `defaultTargets`
- `recommendedFor`
- `excludes`
- `requiresConfirmation`

这让 ECC 2.0 可以表达诸如：

- `developer` 是 Claude 和 Cursor 的推荐默认值
- `research` 对于狭窄的本地安装可能过重
- `full` 是允许的但非默认值

### 3. Target Adapters

这是主要缺失的层。

Module 图不应知道：

- Claude home 位于哪里
- Cursor 如何扁平化或重新映射内容
- 哪些配置文件需要合并语义而非盲目复制

这属于目标适配器。

建议的接口：

```ts
type InstallTargetAdapter = {
  id: string;
  kind: "home" | "project";
  supports(target: string): boolean;
  resolveRoot(input?: string): Promise<string>;
  planOperations(input: InstallOperationInput): Promise<InstallOperation[]>;
  validate?(input: InstallOperationInput): Promise<ValidationIssue[]>;
};
```

建议的首批适配器：

1. `claude-home`
   写入 `~/.claude/...`
2. `cursor-project`
   写入 `./.cursor/...`
3. `antigravity-project`
   写入 `./.agent/...`
4. `codex-home`
   稍后
5. `opencode-home`
   稍后

这与 session-adapter discovery 文档中已经提出的模式相同：首先是规范契约，然后是 harness 特定适配器。

## Install Planning Model

当前的 `scripts/install-plan.js` CLI 证明仓库可以将请求的 modules 解析为过滤后的 module 集。

ECC 2.0 需要下一层：操作规划。

建议的阶段：

1. 输入规范化
   - 解析 `--target`
   - 解析 `--profile`
   - 解析 `--modules`
   - 可选地转换 legacy language 参数
2. module 解析
   - 扩展依赖
   - 拒绝冲突
   - 按支持的目标过滤
3. adapter 规划
   - 解析目标根目录
   - 派确切的复制或生成操作
   - 识别配置合并和目标重映射
4. dry-run 输出
   - 显示选定的 modules
   - 显示跳过的 modules
   - 显示确切的文件操作
5. 变更
   - 执行操作计划
6. 状态写入
   - 仅在成功完成后持久化 install-state

建议的操作形态：

```json
{
  "kind": "copy",
  "moduleId": "rules-core",
  "source": "rules/common/coding-style.md",
  "destination": "/Users/example/.claude/rules/ecc/common/coding-style.md",
  "ownership": "managed",
  "overwritePolicy": "replace"
}
```

其他操作类型：

- `copy`
- `copy-tree`
- `flatten-copy`
- `render-template`
- `merge-json`
- `merge-jsonc`
- `mkdir`
- `remove`

## Install-State Contract

Install-state 是 ECC 1.x 缺失的持久契约。

建议的路径约定：

- Claude 目标：
  `~/.claude/ecc/install-state.json`
- Cursor 目标：
  `./.cursor/ecc-install-state.json`
- Antigravity 目标：
  `./.agent/ecc-install-state.json`
- 未来的 Codex 目标：
  `~/.codex/ecc-install-state.json`

建议的 payload：

```json
{
  "schemaVersion": "ecc.install.v1",
  "installedAt": "2026-03-13T00:00:00Z",
  "lastValidatedAt": "2026-03-13T00:00:00Z",
  "target": {
    "id": "claude-home",
    "root": "/Users/example/.claude"
  },
  "request": {
    "profile": "developer",
    "modules": ["orchestration"],
    "legacyLanguages": ["typescript", "python"]
  },
  "resolution": {
    "selectedModules": [
      "rules-core",
      "agents-core",
      "commands-core",
      "hooks-runtime",
      "platform-configs",
      "workflow-quality",
      "framework-language",
      "database",
      "orchestration"
    ],
    "skippedModules": []
  },
  "source": {
    "repoVersion": "2.0.0-rc.1",
    "repoCommit": "git-sha",
    "manifestVersion": 1
  },
  "operations": [
    {
      "kind": "copy",
      "moduleId": "rules-core",
      "destination": "/Users/example/.claude/rules/ecc/common/coding-style.md",
      "digest": "sha256:..."
    }
  ]
}
```

状态要求：

- 足够的细节让 uninstall 仅移除 ECC 托管的输出
- 足够的细节让 repair 比较期望与实际安装的文件
- 足够的细节让 doctor 解释漂移而非猜测

## Lifecycle Commands

以下命令是 install-state 的生命周期界面：

1. `ecc list-installed`
2. `ecc uninstall`
3. `ecc doctor`
4. `ecc repair`

当前实现状态：

- `ecc list-installed` 路由到 `node scripts/list-installed.js`
- `ecc uninstall` 路由到 `node scripts/uninstall.js`
- `ecc doctor` 路由到 `node scripts/doctor.js`
- `ecc repair` 路由到 `node scripts/repair.js`
- 遗留脚本入口点在迁移期间保持可用

### `list-installed`

职责：

- 显示 target id 和 root
- 显示请求的 profile/modules
- 显示已解析的 modules
- 显示源版本和安装时间

### `uninstall`

职责：

- 加载 install-state
- 仅移除状态中记录的 ECC 托管目标
- 保留用户编写的无关文件不变
- 仅在成功清理后删除 install-state

### `doctor`

职责：

- 检测缺失的托管文件
- 检测意外的配置漂移
- 检测不再存在的目标根目录
- 检测 manifest/version 不匹配

### `repair`

职责：

- 从 install-state 重建期望的操作计划
- 重新复制缺失或漂移的托管文件
- 如果请求的 modules 不再存在于当前 manifest 中则拒绝修复，除非存在兼容性映射

## Legacy Compatibility Layer

当前的 `install.sh` 接受：

- `--target <claude|cursor|antigravity>`
- language 名称列表

这种行为不能一刀切地消失，因为用户已经依赖它。

ECC 2.0 应将 legacy language 参数转换为兼容性请求。

建议的方法：

1. 为 legacy 模式保留现有的 CLI 形态
2. 将 language 名称映射到 module 请求，例如：
   - `rules-core`
   - 目标兼容的 rule 子集
3. 即使对于 legacy 安装也写入 install-state
4. 将请求标记为 `legacyMode: true`

示例：

```json
{
  "request": {
    "legacyMode": true,
    "legacyLanguages": ["typescript", "python"]
  }
}
```

这保留了旧行为，同时将所有安装移至相同的状态契约。

## Publish Boundary

当前的 npm package 仍通过 `package.json` 发布广泛的 payload。

ECC 2.0 应谨慎改进这一点。

推荐的顺序：

1. 首先保留一个规范的 npm package
2. 在更改发布形态之前使用 manifests 驱动安装时选择
3. 稍后才考虑在安全的地方减少打包范围

原因：

- selective install 可以在积极的包手术之前发布
- uninstall 和 repair 更多依赖于 install-state 而非发布更改
- 如果包源保持统一，Codex/OpenCode 支持会更容易

可能的后续方向：

- 按 profile 生成的 slim bundles
- 生成目标特定的 tarballs
- 可选的远程获取 heavy modules

这些是第 3 阶段或更晚，不是 profile-aware 安装的先决条件。

## File Layout Recommendation

建议的后续文件：

```text
scripts/lib/install-targets/
  claude-home.js
  cursor-project.js
  antigravity-project.js
  registry.js
scripts/lib/install-state.js
scripts/ecc.js
scripts/install-apply.js
scripts/list-installed.js
scripts/uninstall.js
scripts/doctor.js
scripts/repair.js
tests/lib/install-targets.test.js
tests/lib/install-state.test.js
tests/lib/install-lifecycle.test.js
```

`install.sh` 可以在迁移期间保留为面向用户的入口点，但它应该成为围绕基于 Node 的 planner 和 executor 的薄 shell，而非继续增长 per-target shell 分支。

## Implementation Sequence

### Phase 1: Planner To Contract

1. 保留当前的 manifest schema 和 resolver
2. 在已解析的 modules 之上添加操作规划
3. 定义 `ecc.install.v1` 状态 schema
4. 在成功安装时写入 install-state

### Phase 2: Target Adapters

1. 将 Claude 安装行为提取到 `claude-home` adapter
2. 将 Cursor 安装行为提取到 `cursor-project` adapter
3. 将 Antigravity 安装行为提取到 `antigravity-project` adapter
4. 将 `install.sh` 简化为参数解析和适配器调用

### Phase 3: Lifecycle

1. 为 config-like modules 添加更强的目标特定合并/移除语义
2. 扩展非复制操作的 repair/uninstall 覆盖范围
3. 将包发布范围缩小到 module 图而非广泛的文件夹
4. 决定 `ecc-install` 何时应成为 `ecc install` 的薄别名

### Phase 4: Publish And Future Targets

1. 评估安全减少 `package.json` 发布范围
2. 添加 `codex-home`
3. 添加 `opencode-home`
4. 如果打包压力仍然很高，考虑生成的 profile bundles

## Immediate Repo-Local Next Steps

此仓库中最高信号的下一步实现行动是：

1. 为 config-like modules 添加目标特定的合并/移除语义
2. 将 repair 和 uninstall 扩展到简单的 copy-file 操作之外
3. 将包发布范围缩小到 module 图而非广泛的文件夹
4. 决定 `ecc-install` 是保持独立还是成为 `ecc install`
5. 添加测试以锁定：
   - 目标特定的合并/移除行为
   - 非复制操作的 repair 和 uninstall 安全性
   - 统一的 `ecc` CLI 路由和兼容性保证

## Open Questions

1. rules 应该永远在 legacy mode 中保持 language-addressable，还是仅在迁移窗口期间？
2. `platform-configs` 应该总是随 `core` 安装，还是拆分为更小的目标特定 modules？
3. 配置合并语义应该记录在操作级别还是仅在适配器逻辑中？
4. heavy skill 系列最终是否应该移动到按需获取而非打包时包含？
5. Codex 和 OpenCode 目标适配器是否应该仅在 Claude/Cursor 生命周期命令稳定后才发布？

## Recommendation

将当前的 manifest resolver 视为安装的适配器 `0`：

1. 保留当前的安装界面
2. 将真实的复制行为移到目标适配器后面
3. 为每次成功安装写入 install-state
4. 让 uninstall、doctor 和 repair 仅依赖于 install-state
5. 只有那时才收缩打包或添加更多目标

这是从 ECC 1.x 安装程序蔓延到确定性、可支持和可扩展的 ECC 2.0 安装/控制契约的最短路径。
