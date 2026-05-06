# Eval 命令

管理基于评估的开发工作流。

## 使用法

`/eval [define|check|report|list|clean] [feature-name]`

## 评估定义

`/eval define feature-name`

创建新的评估定义：

1. 在 `.claude/evals/feature-name.md` 中生成模板：

```markdown
## EVAL: feature-name
Created: $(date)

### Capability Evals
- [ ] [功能 1 的描述]
- [ ] [功能 2 的描述]

### Regression Evals
- [ ] [原有行为 1 仍然正常工作]
- [ ] [原有行为 2 仍然正常工作]

### Success Criteria
- capability eval 需满足 pass@3 > 90%
- regression eval 需满足 pass^3 = 100%
```

2. 引导用户输入具体标准

## 评估检查

`/eval check feature-name`

执行功能评估：

1. 从 `.claude/evals/feature-name.md` 读取评估定义
2. 对每个 capability eval：
   - 尝试验证标准
   - 记录 PASS/FAIL
   - 在 `.claude/evals/feature-name.log` 中记录尝试
3. 对每个 regression eval：
   - 运行相关测试
   - 与基准比较
   - 记录 PASS/FAIL
4. 报告当前状态：

```
EVAL CHECK: feature-name
========================
Capability: X/Y passing
Regression: X/Y passing
Status: IN PROGRESS / READY
```

## 评估报告

`/eval report feature-name`

生成综合评估报告：

```
EVAL REPORT: feature-name
=========================
Generated: $(date)

CAPABILITY EVALS
----------------
[eval-1]: PASS (pass@1)
[eval-2]: PASS (pass@2) - 需要重试
[eval-3]: FAIL - 参阅备注

REGRESSION EVALS
----------------
[test-1]: PASS
[test-2]: PASS
[test-3]: PASS

METRICS
-------
Capability pass@1: 67%
Capability pass@3: 100%
Regression pass^3: 100%

NOTES
-----
[问题、边缘情况或观察事项]

RECOMMENDATION
--------------
[SHIP / NEEDS WORK / BLOCKED]
```

## 评估列表

`/eval list`

显示所有评估定义：

```
EVAL DEFINITIONS
================
feature-auth      [3/5 passing] IN PROGRESS
feature-search    [5/5 passing] READY
feature-export    [0/4 passing] NOT STARTED
```

## 参数

$ARGUMENTS:
- `define <name>` - 创建新评估定义
- `check <name>` - 执行并检查评估
- `report <name>` - 生成完整报告
- `list` - 显示所有评估
- `clean` - 清理旧评估日志（保留最近 10 次运行）
