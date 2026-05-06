---
name: evolve
description: 将相关的 instincts 聚类成 skills、commands 或 agents
command: true
---

# Evolve 命令

## 实现

使用插件根路径运行 instinct CLI：

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/skills/continuous-learning-v2/scripts/instinct-cli.py" evolve [--generate]
```

或者如果没有设置 `CLAUDE_PLUGIN_ROOT`（手动安装）：

```bash
python3 ~/.claude/skills/continuous-learning-v2/scripts/instinct-cli.py evolve [--generate]
```

分析 instincts，并将相关的 instincts 聚类成高层结构：
- **Commands**：当 instincts 描述用户调用的操作时
- **Skills**：当 instincts 描述自动触发的行为时
- **Agents**：当 instincts 描述复杂的多步骤过程时

## 使用方法

```
/evolve                    # 分析所有 instincts 并提出进化建议
/evolve --domain testing   # 仅进化 testing 领域的 instincts
/evolve --dry-run          # 显示将创建的内容而不实际创建
/evolve --threshold 5      # 聚类需要 5 个以上相关 instincts
```

## 进化规则

### → Command（用户调用）
当 instincts 描述用户明确要求的操作时：
- 关于"当用户要求……"的多个 instincts
- 具有"创建新 X 时"类触发条件的 instincts
- 遵循可重复序列的 instincts

示例：
- `new-table-step1`："添加数据库表时，创建迁移"
- `new-table-step2`："添加数据库表时，更新 schema"
- `new-table-step3`："添加数据库表时，重新生成类型"

→ 创建：`/new-table` 命令

### → Skill（自动触发）
当 instincts 描述应自动发生的行为时：
- 模式匹配触发
- 错误处理响应
- 代码风格强制

示例：
- `prefer-functional`："编写函数时，优先使用函数式风格"
- `use-immutable`："修改状态时，使用不可变模式"
- `avoid-classes`："设计模块时，避免基于类的设计"

→ 创建：`functional-patterns` skill

### → Agent（需要深度/分离）
当 instincts 描述受益于分离的复杂多步骤过程时：
- 调试工作流
- 重构序列
- 研究任务

示例：
- `debug-step1`："调试时，首先查看日志"
- `debug-step2`："调试时，隔离失败的组件"
- `debug-step3`："调试时，创建最小化重现"
- `debug-step4`："调试时，通过测试验证修复"

→ 创建：`debugger` agent

## 执行内容

1. 从 `~/.claude/homunculus/instincts/` 读取所有 instincts
2. 按以下方式分组 instincts：
   - 领域相似性
   - 触发模式重叠
   - 操作序列关系
3. 对于每个包含 3 个以上相关 instincts 的集群：
   - 确定进化类型（command/skill/agent）
   - 生成适当的文件
   - 保存到 `~/.claude/homunculus/evolved/{commands,skills,agents}/`
4. 将进化后的结构链接到源 instincts

## 输出格式

```
 进化分析
==================

发现 3 个准备进化的集群：

## 集群 1：数据库迁移工作流
Instincts: new-table-migration, update-schema, regenerate-types
类型: Command
置信度: 85%（基于 12 次观察）

创建: /new-table 命令
文件:
  - ~/.claude/homunculus/evolved/commands/new-table.md

## 集群 2：函数式代码风格
Instincts: prefer-functional, use-immutable, avoid-classes, pure-functions
类型: Skill
置信度: 78%（基于 8 次观察）

创建: functional-patterns skill
文件:
  - ~/.claude/homunculus/evolved/skills/functional-patterns.md

## 集群 3：调试流程
Instincts: debug-check-logs, debug-isolate, debug-reproduce, debug-verify
类型: Agent
置信度: 72%（基于 6 次观察）

创建: debugger agent
文件:
  - ~/.claude/homunculus/evolved/agents/debugger.md

---
请运行 `/evolve --execute` 来创建这些文件。
```

## 标志

- `--execute`：实际创建进化后的结构（默认为预览模式）
- `--dry-run`：不创建，仅预览
- `--domain <name>`：仅进化指定领域的 instincts
- `--threshold <n>`：形成集群所需的最小 instincts 数量（默认：3）
- `--type <command|skill|agent>`：仅创建指定类型

## 生成的文件格式

### Command
```markdown
---
name: new-table
description: 通过迁移、schema 更新和类型生成创建新的数据库表
command: /new-table
evolved_from:
  - new-table-migration
  - update-schema
  - regenerate-types
---

# New Table 命令

[基于聚类的 instincts 生成的内容]

## 步骤
1. ...
2. ...
```

### Skill
```markdown
---
name: functional-patterns
description: 强制函数式编程模式
evolved_from:
  - prefer-functional
  - use-immutable
  - avoid-classes
---

# Functional Patterns skill

[基于聚类的 instincts 生成的内容]
```

### Agent
```markdown
---
name: debugger
description: 系统化调试 agent
model: sonnet
evolved_from:
  - debug-check-logs
  - debug-isolate
  - debug-reproduce
---

# Debugger agent

[基于聚类的 instincts 生成的内容]
```
