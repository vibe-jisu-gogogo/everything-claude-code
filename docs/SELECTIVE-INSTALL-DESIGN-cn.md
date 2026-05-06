# ECC Selective Install 设计

## 目的

本文档定义了 ECC 面向用户的选择性安装设计。

它是 `docs/SELECTIVE-INSTALL-ARCHITECTURE.md` 的补充，后者专注于内部运行时架构和代码边界。

本文档首先回答产品和运维问题：

- 用户如何选择 ECC 组件
- CLI 应该是什么样的体验
- 应该存在什么样的配置文件
- 跨 harness targets 的安装行为应该如何
- 设计如何映射到当前 ECC 代码库而无需重写

## 问题

如今，ECC 仍然让人感觉像是一个大型负载安装程序，尽管代码库现在已经有了初步的清单和生命周期支持。

用户需要更简单的心智模型：

- 安装基准版
- 添加他们实际使用的语言包
- 添加他们实际需要的框架配置
- 添加可选的功能包，如 security、research 或 orchestration

选择性安装系统应该让 ECC 感觉是可组合的，而不是全有或全无。

在当前基础架构中，面向用户的组件仍然是较粗粒度内部安装模块之上的别名层。这意味着 include/exclude 在模块选择级别已经有用，但在底层模块图被更精细地拆分之前，某些文件级边界仍然不够完善。

## 目标

1. 让用户能够快速安装较小的默认 ECC footprint。
2. 让用户能够从可重用的组件家族组合安装：
   - core rules
   - language packs
   - framework packs
   - capability packs
   - target/platform configs
3. 在 Claude、Cursor、Antigravity、Codex 和 OpenCode 之间保持一致的 UX。
4. 保持安装可检查、可修复和可卸载。
5. 在推出期间保持与当前 `ecc-install typescript` 风格的向后兼容性。

## 非目标

- 在第一阶段将 ECC 打包成多个 npm packages
- 构建远程 marketplace
- 同一阶段的完整 control-plane UI
- 在选择性安装发布前解决所有 skill-classification 问题

## 用户体验原则

### 1. 从小处开始

用户应该能够通过一个命令获得有用的 ECC 安装：

```bash
ecc install --target claude --profile core
```

默认体验不应该假设用户想要每个 skill family 和每个框架。

### 2. 按意图构建

用户应该这样思考：

- "我想要开发者基准版"
- "我需要 TypeScript 和 Python"
- "我想要 Next.js 和 Django"
- "我想要 security pack"

用户不应该需要知道原始的内部仓库路径。

### 3. 在变更前预览

每个安装路径都应该支持 dry-run 规划：

```bash
ecc install --target cursor --profile developer --with lang:typescript --with framework:nextjs --dry-run
```

计划应该清晰显示：

- 选定的组件
- 跳过的组件
- 目标根目录
- 托管路径
- 预期的 install-state 位置

### 4. 本地配置应该是一等公民

团队应该能够提交项目级安装配置并使用：

```bash
ecc install --config ecc-install.json
```

这允许在贡献者和 CI 之间进行确定性安装。

## 组件模型

当前清单已经使用安装模块和 profiles。面向用户的设计应该保留该内部结构，但将其呈现为四个主要组件家族。

近期实现说明：一些面向用户的组件 ID 仍然解析到共享的内部模块，特别是在 language/framework 层。目录可以立即改善 UX，同时在后续阶段保留通往更精细模块粒度的清晰路径。

### 1. Baseline

这些是默认的 ECC 构建块：

- core rules
- baseline agents
- core commands
- runtime hooks
- platform configs
- workflow quality primitives

当前内部模块示例：

- `rules-core`
- `agents-core`
- `commands-core`
- `hooks-runtime`
- `platform-configs`
- `workflow-quality`

### 2. Language Packs

语言包将语言生态系统的 rules、guidance 和 workflows 组合在一起。

示例：

- `lang:typescript`
- `lang:python`
- `lang:go`
- `lang:java`
- `lang:rust`

每个语言包应该解析到一个或多个内部模块以及特定于目标的资产。

### 3. Framework Packs

Framework packs 位于 language packs 之上，引入特定于框架的 rules、skills 和可选设置。

示例：

- `framework:react`
- `framework:nextjs`
- `framework:django`
- `framework:springboot`
- `framework:laravel`

Framework packs 应该在适当的地方依赖于正确的 language pack 或 baseline primitives。

### 4. Capability Packs

Capability packs 是跨领域的 ECC 功能包。

示例：

- `capability:security`
- `capability:research`
- `capability:orchestration`
- `capability:media`
- `capability:content`

这些应该映射到清单中已经引入的当前模块家族。

## Profiles

Profiles 仍然是最快的入门方式。

推荐的面向用户 profiles：

- `core`
  最小基准版，对大多数尝试 ECC 的用户来说是安全的默认值
- `developer`
  积极的软件工程工作的最佳默认值
- `security`
  基准版加上以 security 为重点的 guidance
- `research`
  基准版加上 research/content/investigation 工具
- `full`
  所有已分类和当前支持的内容

Profiles 应该可以通过额外的 `--with` 和 `--without` 标志进行组合。

示例：

```bash
ecc install --target claude --profile developer --with lang:typescript --with framework:nextjs --without capability:orchestration
```

## 建议的 CLI 设计

### 主要命令

```bash
ecc install
ecc plan
ecc list-installed
ecc doctor
ecc repair
ecc uninstall
ecc catalog
```

### Install CLI

推荐形式：

```bash
ecc install [--target <target>] [--profile <name>] [--with <component>]... [--without <component>]... [--config <path>] [--dry-run] [--json]
```

示例：

```bash
ecc install --target claude --profile core
ecc install --target cursor --profile developer --with lang:typescript --with framework:nextjs
ecc install --target antigravity --with capability:security --with lang:python
ecc install --config ecc-install.json
```

### Plan CLI

推荐形式：

```bash
ecc plan [same selection flags as install]
```

目的：

- 生成无变更的预览
- 作为选择性安装的规范调试界面

### Catalog CLI

推荐形式：

```bash
ecc catalog profiles
ecc catalog components
ecc catalog components --family language
ecc catalog show framework:nextjs
```

目的：

- 让用户无需阅读文档即可发现有效的组件名称
- 保持配置编写简单易懂

### 兼容性 CLI

这些传统流程在迁移期间应该仍然有效：

```bash
ecc-install typescript
ecc-install --target cursor typescript
ecc typescript
```

在内部，这些应该规范化为新的请求模型，并以与现代安装相同的方式写入 install-state。

## 建议的配置文件

### 文件名

推荐默认值：

- `ecc-install.json`

可选的未来支持：

- `.ecc/install.json`

### 配置格式

```json
{
  "$schema": "./schemas/ecc-install-config.schema.json",
  "version": 1,
  "target": "cursor",
  "profile": "developer",
  "include": [
    "lang:typescript",
    "lang:python",
    "framework:nextjs",
    "capability:security"
  ],
  "exclude": [
    "capability:media"
  ],
  "options": {
    "hooksProfile": "standard",
    "mcpCatalog": "baseline",
    "includeExamples": false
  }
}
```

### 字段语义

- `target`
  选定的 harness target，如 `claude`、`cursor` 或 `antigravity`
- `profile`
  要开始的基准 profile
- `include`
  要添加的其他组件
- `exclude`
  要从 profile 结果中减去的组件
- `options`
  不改变组件身份的 target/runtime 调优标志

### 优先级规则

1. CLI 参数覆盖配置文件值。
2. 配置文件覆盖 profile 默认值。
3. profile 默认值覆盖内部模块默认值。

这保持行为可预测且易于解释。

## 模块化安装流程

面向用户的流程应该是：

1. 如果提供或自动检测到配置文件，则加载它
2. 将 CLI 意图合并到配置意图之上
3. 将请求规范化为规范选择
4. 将 profile 扩展为基准组件
5. 添加 `include` 组件
6. 减去 `exclude` 组件
7. 解析依赖关系和目标兼容性
8. 渲染计划
9. 如果不在 dry-run 模式下，则应用操作
10. 写入 install-state

重要的 UX 属性是完全相同的流程驱动：

- `install`
- `plan`
- `repair`
- `uninstall`

命令在操作上不同，但在 ECC 理解选定安装的方式上没有不同。

## 目标行为

选择性安装应该在所有目标之间保留相同的概念组件图，同时让目标适配器决定内容如何落地。

### Claude

最适合：

- home-scoped ECC 基准版
- commands、agents、rules、hooks、platform config、orchestration

### Cursor

最适合：

- project-scoped 安装
- rules 加上项目本地自动化和配置

### Antigravity

最适合：

- project-scoped agent/rule/workflow 安装

### Codex / OpenCode

应该保持为附加目标，而不是安装程序的特殊分支。

选择性安装设计应该使这些只是新的适配器加上新的目标特定映射规则，而不是新的安装程序架构。

## 技术可行性

这个设计是可行的，因为代码库已经有：

- 安装模块和 profile 清单
- 带有 install-state 路径的目标适配器
- 计划检查
- install-state 记录
- 生命周期命令
- 统一的 `ecc` CLI 界面

缺失的工作不是概念发明。缺失的工作是将当前基础架构产品化为更清晰的面向用户的组件模型。

### 第一阶段可行

- profile + include/exclude 选择
- `ecc-install.json` 配置文件解析
- catalog/discovery 命令
- 从面向用户的组件 ID 到内部模块集的别名映射
- dry-run 和 JSON planning

### 第二阶段可行

- 更丰富的目标适配器语义
- 配置类资产的合并感知操作
- 非复制操作的更强 repair/uninstall 行为

### 以后

- 减少发布范围
- 生成的精简 bundles
- 远程组件获取

## 映射到当前 ECC 清单

当前清单尚未公开真正的面向用户的 `lang:*` / `framework:*` / `capability:*` 分类法。这应该作为现有模块之上的表示层引入，而不是作为第二个安装程序引擎。

推荐方法：

- 保留 `install-modules.json` 作为内部解析目录
- 添加面向用户的组件目录，将友好的组件 ID 映射到一个或多个内部模块
- 让 profiles 在迁移窗口期间引用内部模块或面向用户的组件 ID

这避免了在改善 UX 的同时破坏当前的选择性安装基础架构。

## 建议的推出

### 第一阶段：设计和发现

- 最终确定面向用户的组件分类法
- 添加配置 schema
- 添加 CLI 设计和优先级规则

### 第二阶段：面向用户的解析层

- 实现组件别名
- 实现配置文件解析
- 实现 `include` / `exclude`
- 实现 `catalog`

### 第三阶段：更强的目标语义

- 将更多逻辑移至目标拥有的规划中
- 干净地支持 merge/generate 操作
- 提高 repair/uninstall 保真度

### 第四阶段：打包优化

- 缩小发布范围
- 评估生成的 bundles

## 建议

下一个实现步骤不应该是"重写安装程序"。

而应该是：

1. 保留当前的清单/运行时基础架构
2. 添加面向用户的组件目录和配置文件
3. 添加 `include` / `exclude` 选择和目录发现
4. 让现有的规划器和生命周期栈消费该模型

这是从当前 ECC 代码库到真正的选择性安装体验的最短路径，让人感觉像是 ECC 2.0，而不是一个大型的传统安装程序。
