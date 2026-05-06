# Eval 命令

管理评估驱动开发工作流。

## 使用方法

`/eval [define|check|report|list] [功能名]`

## 定义 Eval

`/eval define 功能名`

创建新的评估定义。

1. 使用模板创建 `.claude/evals/功能名.md`:

```markdown
## EVAL: 功能名
创建日期: $(date)

### 功能评估
- [ ] [功能1的描述]
- [ ] [功能2的描述]

### 回归评估
- [ ] [现有功能1正常运行]
- [ ] [现有功能2正常运行]

### 成功标准
- 功能评估: pass@3 > 90%
- 回归评估: pass^3 = 100%
```

2. 提示用户填写具体标准

## 检查 Eval

`/eval check 功能名`

执行功能评估。

1. 从 `.claude/evals/功能名.md` 读取评估定义
2. 对于每个功能评估:
   - 尝试验证标准
   - 记录 PASS/FAIL
   - 在 `.claude/evals/功能名.log` 中记录尝试
3. 对于每个回归评估:
   - 运行相关测试
   - 与基线比较
   - 记录 PASS/FAIL
4. 报告当前状态:

```
EVAL CHECK: 功能名
========================
功能评估: X/Y 合格
回归评估: X/Y 合格
状态: 进行中 / 准备就绪
```

## 报告 Eval

`/eval report 功能名`

生成综合评估报告。

```
EVAL REPORT: 功能名
=========================
生成时间: $(date)

功能评估
----------------
[eval-1]: PASS (pass@1)
[eval-2]: PASS (pass@2) - 需要重试
[eval-3]: FAIL - 请参阅备注

回归评估
----------------
[test-1]: PASS
[test-2]: PASS
[test-3]: PASS

指标
-------
功能评估 pass@1: 67%
功能评估 pass@3: 100%
回归评估 pass^3: 100%

备注
-----
[问题、边缘情况或观察事项]

建议
--------------
[可发布 / 需要修复 / 阻塞中]
```

## 列出 Eval

`/eval list`

显示所有评估定义。

```
EVAL 定义列表
================
feature-auth      [3/5 合格] 进行中
feature-search    [5/5 合格] 准备就绪
feature-export    [0/4 合格] 未开始
```

## 参数

$ARGUMENTS:
- `define <名称>` - 创建新的评估定义
- `check <名称>` - 执行评估并检查
- `report <名称>` - 生成完整报告
- `list` - 显示所有评估
- `clean` - 删除旧的评估日志（保留最新10条）
