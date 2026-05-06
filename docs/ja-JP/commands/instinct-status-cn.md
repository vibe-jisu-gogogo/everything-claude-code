---
name: instinct-status
description: 显示所有已学习的instinct及其置信度水平
command: true
---

# Instinct状态命令

按域分组显示所有已学习的instinct及其置信度分数。

## 实现

使用插件根路径执行instinct CLI:

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/skills/continuous-learning-v2/scripts/instinct-cli.py" status
```

或者，如果未设置`CLAUDE_PLUGIN_ROOT`（手动安装）:

```bash
python3 ~/.claude/skills/continuous-learning-v2/scripts/instinct-cli.py status
```

## 使用方法

```
/instinct-status
/instinct-status --domain code-style
/instinct-status --low-confidence
```

## 执行内容

1. 从`~/.claude/homunculus/instincts/personal/`读取所有instinct文件
2. 从`~/.claude/homunculus/instincts/inherited/`读取继承的instinct
3. 按域分组，显示置信度条

## 输出格式

```
 instinct状态
==================

## 代码风格 (4 instincts)

### prefer-functional-style
触发: 编写新函数时
动作: 使用函数式模式而非类
置信度: ████████░░ 80%
来源: session-observation | 最后更新: 2025-01-22

### use-path-aliases
触发: 导入模块时
动作: 使用@/路径别名而非相对导入
置信度: ██████░░░░ 60%
来源: repo-analysis (github.com/acme/webapp)

## 测试 (2 instincts)

### test-first-workflow
触发: 添加新功能时
动作: 先写测试，再实现
置信度: █████████░ 90%
来源: session-observation

## 工作流 (3 instincts)

### grep-before-edit
触发: 修改代码时
动作: 先用Grep搜索，Read确认，然后Edit
置信度: ███████░░░ 70%
来源: session-observation

---
总计: 9 instincts (4个人, 5继承)
观察者: 运行中 (最后分析: 5分钟前)
```

## 标志

- `--domain <name>`: 按域过滤（code-style、testing、git等）
- `--low-confidence`: 仅显示置信度 < 0.5的instinct
- `--high-confidence`: 仅显示置信度 >= 0.7的instinct
- `--source <type>`: 按来源过滤（session-observation、repo-analysis、inherited）
- `--json`: 以JSON格式输出，供程序使用
