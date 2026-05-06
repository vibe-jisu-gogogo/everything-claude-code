---
name: evolve
description: 分析本能并建议或生成进化后的结构
command: true
---

# Evolve 命令

## 实现

使用插件根路径运行 instinct CLI：

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/skills/continuous-learning-v2/scripts/instinct-cli.py" evolve [--generate]
```

或者如果`CLAUDE_PLUGIN_ROOT`未设置（手动安装）：

```bash
python3 ~/.claude/skills/continuous-learning-v2/scripts/instinct-cli.py evolve [--generate]
```

分析本能并将相关的本能聚类为更高级的结构：
- **Commands**: 当本能描述用户调用的操作时
- **Skills**: 当本能描述自动触发的行为时
- **Agents**: 当本能描述复杂的多步骤流程时

## 用法

```
/evolve                    # 分析所有本能并建议进化方向
/evolve --generate         # 同时在 evolved/{skills,commands,agents} 下生成文件
```

## 进化规则

### → Command（用户调用）
当本能描述用户明确请求的操作时：
- 多个关于"当用户要求..."的本能
- 带有诸如"当创建新的X"之类触发器的本能
- 遵循可重复序列的本能

示例：
- `new-table-step1`: "when adding a database table, create migration"
- `new-table-step2`: "when adding a database table, update schema"
- `new-table-step3`: "when adding a database table, regenerate types"

→ 生成：**new-table** command

### → Skill（自动触发）
当本能描述应该自动发生的行为时：
- 模式匹配触发器
- 错误处理响应
- 代码风格强制

示例：
- `prefer-functional`: "when writing functions, prefer functional style"
- `use-immutable`: "when modifying state, use immutable patterns"
- `avoid-classes`: "when designing modules, avoid class-based design"

→ 生成：`functional-patterns` skill

### → Agent（需要深度/隔离）
当本能描述受益于隔离的复杂多步骤流程时：
- 调试工作流
- 重构序列
- 研究任务

示例：
- `debug-step1`: "when debugging, first check logs"
- `debug-step2`: "when debugging, isolate the failing component"
- `debug-step3`: "when debugging, create minimal reproduction"
- `debug-step4`: "when debugging, verify fix with test"

→ 生成：**debugger** agent

## 执行步骤

1. 检测当前项目上下文
2. 读取项目 + 全局本能（ID冲突时项目优先）
3. 按触发器/领域模式对本能分组
4. 识别：
   - Skill 候选（包含2个以上本能的触发器集群）
   - Command 候选（高置信度工作流本能）
   - Agent 候选（更大的高置信度集群）
5. 适用时显示升级候选（项目 -> 全局）
6. 如果传递了`--generate`参数，将文件写入：
   - 项目范围：`~/.claude/homunculus/projects/<project-id>/evolved/`
   - 全局回退：`~/.claude/homunculus/evolved/`

## 输出格式

```
============================================================
  EVOLVE 分析 - 12个本能
  项目：my-app (a1b2c3d4e5f6)
  项目范围：8 | 全局：4
============================================================

高置信度本能（>=80%）：5

## SKILL 候选
1. 集群："adding tests"
   本能：3
   平均置信度：82%
   领域：testing
   范围：project

## COMMAND 候选（2个）
  /adding-tests
    来源：test-first-workflow [项目]
    置信度：84%

## AGENT 候选（1个）
  adding-tests-agent
    覆盖3个本能
    平均置信度：82%
```

## 标志

- `--generate`: 除分析输出外还生成进化后的文件

## 生成文件格式

### Command
```markdown
---
name: new-table
description: 创建新的数据库表，包括迁移、模式更新和类型生成
command: /new-table
evolved_from:
  - new-table-migration
  - update-schema
  - regenerate-types
---

# New Table 命令

[基于聚类本能生成的内容]

## 步骤
1. ...
2. ...
```

### Skill
```markdown
---
name: functional-patterns
description: 强制执行函数式编程模式
evolved_from:
  - prefer-functional
  - use-immutable
  - avoid-classes
---

# Functional Patterns Skill

[基于聚类本能生成的内容]
```

### Agent
```markdown
---
name: debugger
description: 系统调试agent
model: sonnet
evolved_from:
  - debug-check-logs
  - debug-isolate
  - debug-reproduce
---

# Debugger Agent

[基于聚类本能生成的内容]
```
