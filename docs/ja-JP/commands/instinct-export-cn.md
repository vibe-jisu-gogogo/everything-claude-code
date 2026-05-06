---
name: instinct-export
description: 导出 instinct 以与团队成员或其他项目共享
command: /instinct-export
---

# Instinct 导出命令

以可共享的格式导出 instinct。适用于以下场景：
- 与团队成员共享
- 迁移到新机器
- 为项目规约做贡献

## 使用方法

```
/instinct-export                           # 导出所有个人 instinct
/instinct-export --domain testing          # 仅导出测试相关的 instinct
/instinct-export --min-confidence 0.7      # 仅导出高可信度的 instinct
/instinct-export --output team-instincts.yaml
```

## 执行内容

1. 从 `~/.claude/homunculus/instincts/personal/` 读取 instinct
2. 根据标志位进行过滤
3. 排除敏感信息：
   - 删除会话 ID
   - 删除文件路径（仅保留模式）
   - 删除早于"上周"的时间戳
4. 生成导出文件

## 输出格式

创建 YAML 文件：

```yaml
# Instincts Export
# Generated: 2025-01-22
# Source: personal
# Count: 12 instincts

version: "2.0"
exported_by: "continuous-learning-v2"
export_date: "2025-01-22T10:30:00Z"

instincts:
  - id: prefer-functional-style
    trigger: "when writing new functions"
    action: "Use functional patterns over classes"
    confidence: 0.8
    domain: code-style
    observations: 8

  - id: test-first-workflow
    trigger: "when adding new functionality"
    action: "Write test first, then implementation"
    confidence: 0.9
    domain: testing
    observations: 12

  - id: grep-before-edit
    trigger: "when modifying code"
    action: "Search with Grep, confirm with Read, then Edit"
    confidence: 0.7
    domain: workflow
    observations: 6
```

## 隐私注意事项

导出包含的内容：
- PASS: 触发模式
- PASS: 操作
- PASS: 可信度分数
- PASS: 领域
- PASS: 观察次数

导出不包含的内容：
- FAIL: 实际代码片段
- FAIL: 文件路径
- FAIL: 会话记录
- FAIL: 个人身份信息

## 标志位

- `--domain <name>`: 仅导出指定领域
- `--min-confidence <n>`: 最小可信度阈值（默认: 0.3）
- `--output <file>`: 输出文件路径（默认: instincts-export-YYYYMMDD.yaml）
- `--format <yaml|json|md>`: 输出格式（默认: yaml）
- `--include-evidence`: 包含证据文本（默认: 排除）
