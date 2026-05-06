---
name: instinct-import
description: 从文件或URL导入instinct到项目/全局作用域
command: true
---

# Instinct Import 命令

## 实现

运行使用插件根路径的instinct CLI：

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/skills/continuous-learning-v2/scripts/instinct-cli.py" import <file-or-url> [--dry-run] [--force] [--min-confidence 0.7] [--scope project|global]
```

如果未设置`CLAUDE_PLUGIN_ROOT`（手动安装场景）：

```bash
python3 ~/.claude/skills/continuous-learning-v2/scripts/instinct-cli.py import <file-or-url>
```

支持从本地文件路径或HTTP(S) URL导入instinct。

## 用法

```
/instinct-import team-instincts.yaml
/instinct-import https://github.com/org/repo/instincts.yaml
/instinct-import team-instincts.yaml --dry-run
/instinct-import team-instincts.yaml --scope global --force
```

## 操作流程

1. 获取instinct文件（本地路径或URL）
2. 解析并验证格式
3. 检查与现有instinct的重复项
4. 合并或添加新的instinct
5. 保存到继承instinct目录：
   - 项目作用域：`~/.claude/homunculus/projects/<project-id>/instincts/inherited/`
   - 全局作用域：`~/.claude/homunculus/instincts/inherited/`

## 导入过程展示

```
 Importing instincts from: team-instincts.yaml
================================================

Found 12 instincts to import.

Analyzing conflicts...

## New Instincts (8)
These will be added:
  ✓ use-zod-validation (confidence: 0.7)
  ✓ prefer-named-exports (confidence: 0.65)
  ✓ test-async-functions (confidence: 0.8)
  ...

## Duplicate Instincts (3)
Already have similar instincts:
  WARNING: prefer-functional-style
     Local: 0.8 confidence, 12 observations
     Import: 0.7 confidence
     → Keep local (higher confidence)

  WARNING: test-first-workflow
     Local: 0.75 confidence
     Import: 0.9 confidence
     → Update to import (higher confidence)

Import 8 new, update 1?
```

## 合并规则

导入ID已存在的instinct时：
- 置信度更高的导入版本将成为更新候选
- 置信度相等或更低的导入版本将被跳过
- 除非使用`--force`参数，否则需要用户确认操作

## 来源追踪

导入的instinct会被标记以下元数据：
```yaml
source: inherited
scope: project
imported_from: "team-instincts.yaml"
project_id: "a1b2c3d4e5f6"
project_name: "my-project"
```

## 可用参数

- `--dry-run`: 预览导入效果而不实际执行写入操作
- `--force`: 跳过确认提示直接执行导入
- `--min-confidence <n>`: 仅导入置信度高于指定阈值的instinct
- `--scope <project|global>`: 选择导入目标作用域（默认：`project`）

## 输出结果

导入完成后输出：
```
PASS: Import complete!

Added: 8 instincts
Updated: 1 instinct
Skipped: 3 instincts (equal/higher confidence already exists)

New instincts saved to: ~/.claude/homunculus/instincts/inherited/

Run /instinct-status to see all instincts.
```
