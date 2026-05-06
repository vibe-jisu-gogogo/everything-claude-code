---
name: instinct-import
description: 从队友、Skill Creator 和其他来源导入 instinct
command: true
---

# 本能导入命令

## 实现

使用插件根路径执行 instinct CLI：

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/skills/continuous-learning-v2/scripts/instinct-cli.py" import <file-or-url> [--dry-run] [--force] [--min-confidence 0.7]
```

或者，如果没有设置 `CLAUDE_PLUGIN_ROOT`（手动安装）：

```bash
python3 ~/.claude/skills/continuous-learning-v2/scripts/instinct-cli.py import <file-or-url>
```

可以从以下来源导入 instinct：
- 队友的导出文件
- Skill Creator（仓库分析）
- 社区集合
- 之前机器的备份

## 使用方法

```
/instinct-import team-instincts.yaml
/instinct-import https://github.com/org/repo/instincts.yaml
/instinct-import --from-skill-creator acme/webapp
```

## 执行内容

1. 获取 instinct 文件（本地路径或 URL）
2. 解析并验证格式
3. 检查与现有 instinct 的重复
4. 合并或添加新 instinct
5. 保存到 `~/.claude/homunculus/instincts/inherited/`

## 导入过程

```
 正在导入 instincts: team-instincts.yaml
================================================

发现 12 个 instincts。

正在分析冲突...

## 新增 instincts (8)
以下内容将被添加：
  ✓ use-zod-validation (confidence: 0.7)
  ✓ prefer-named-exports (confidence: 0.65)
  ✓ test-async-functions (confidence: 0.8)
  ...

## 重复 instincts (3)
类似的 instincts 已存在：
  WARNING: prefer-functional-style
     本地: 置信度 0.8, 12 次观测
     导入: 置信度 0.7
     → 保留本地版本（置信度更高）

  WARNING: test-first-workflow
     本地: 置信度 0.75
     导入: 置信度 0.9
     → 更新为导入版本（置信度更高）

## 冲突 instincts (1)
与本地 instincts 矛盾：
  FAIL: use-classes-for-services
     冲突: avoid-classes
     → 跳过（需要手动解决）

---
添加 8 个、更新 1 个、跳过 3 个？
```

## 合并策略

### 重复情况
当导入与现有 instinct 一致的 instinct 时：
- **高置信度优先**：保留置信度较高的版本
- **合并证据**：合并观测次数
- **更新时间戳**：标记为最近验证

### 冲突情况
当导入与现有 instinct 矛盾的 instinct 时：
- **默认跳过**：不导入冲突的 instinct
- **标记以供审查**：将两者都标记为需要注意
- **手动解决**：由用户决定保留哪一个

## 来源追踪

导入的 instinct 将按以下方式标记：
```yaml
source: "inherited"
imported_from: "team-instincts.yaml"
imported_at: "2025-01-22T10:30:00Z"
original_source: "session-observation"  # or "repo-analysis"
```

## Skill Creator 集成

从 Skill Creator 导入时：

```
/instinct-import --from-skill-creator acme/webapp
```

这将获取从仓库分析生成的 instinct：
- 来源: `repo-analysis`
- 高初始置信度（0.7 以上）
- 链接到源仓库

## 标志

- `--dry-run`: 预览而不执行导入
- `--force`: 即使有冲突也强制导入
- `--merge-strategy <higher|local|import>`: 处理重复的方式
- `--from-skill-creator <owner/repo>`: 从 Skill Creator 分析导入
- `--min-confidence <n>`: 仅导入置信度高于阈值的 instinct

## 输出

导入后：
```
PASS: 导入完成!

添加: 8 个 instincts
更新: 1 个 instinct
跳过: 3 个 instincts (2 个重复, 1 个冲突)

新 instincts 保存位置: ~/.claude/homunculus/instincts/inherited/

执行 /instinct-status 可查看所有 instincts。
```
