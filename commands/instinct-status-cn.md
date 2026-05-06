---
name: instinct-status
description: 显示已学习的本能规则（项目级 + 全局级）及置信度
command: true
---

# Instinct Status 命令

显示当前项目的已学习本能规则和全局本能规则，按领域分组。

## 实现方式

使用插件根路径运行 instinct CLI：

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/skills/continuous-learning-v2/scripts/instinct-cli.py" status
```

如果未设置 `CLAUDE_PLUGIN_ROOT`（手动安装），使用：

```bash
python3 ~/.claude/skills/continuous-learning-v2/scripts/instinct-cli.py status
```

## 使用方法

```
/instinct-status
```

## 功能说明

1. 检测当前项目上下文（git remote/路径哈希）
2. 从 `~/.claude/homunculus/projects/<project-id>/instincts/` 读取项目级本能规则
3. 从 `~/.claude/homunculus/instincts/` 读取全局级本能规则
4. 按优先级规则合并（ID冲突时项目级覆盖全局级）
5. 按领域分组显示，包含置信度条和观测统计数据

## 输出格式

```
============================================================
  INSTINCT STATUS - 共 12 条
============================================================

  项目: my-app (a1b2c3d4e5f6)
  项目级本能规则: 8 条
  全局级本能规则: 4 条

## 项目范围 (my-app)
  ### 工作流 (3)
    ███████░░░  70%  grep-before-edit [project]
              触发条件: 修改代码时

## 全局范围 (应用于所有项目)
  ### 安全 (2)
    █████████░  85%  validate-user-input [global]
              触发条件: 处理用户输入时
```
