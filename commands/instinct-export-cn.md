---
name: instinct-export
description: 导出项目/全局范围的instincts到文件
command: /instinct-export
---

# Instinct Export 命令

将instincts导出为可共享的格式。适用于：
- 与团队成员共享
- 迁移到新机器
- 为项目规范做贡献

## 使用方法

```
/instinct-export                           # 导出所有个人instincts
/instinct-export --domain testing          # 仅导出testing领域的instincts
/instinct-export --min-confidence 0.7      # 仅导出高置信度的instincts
/instinct-export --output team-instincts.yaml
/instinct-export --scope project --output project-instincts.yaml
```

## 执行步骤

1. 检测当前项目上下文
2. 按选定的scope加载instincts：
   - `project`: 仅当前项目
   - `global`: 仅全局
   - `all`: 项目 + 全局合并（默认）
3. 应用筛选条件（`--domain`、`--min-confidence`）
4. 将YAML格式的导出内容写入文件（如果没有提供输出路径则输出到stdout）

## 输出格式

创建一个YAML文件：

```yaml
# Instincts 导出
# 生成时间: 2025-01-22
# 来源: personal
# 数量: 12 instincts

---
id: prefer-functional-style
trigger: "when writing new functions"
confidence: 0.8
domain: code-style
source: session-observation
scope: project
project_id: a1b2c3d4e5f6
project_name: my-app
---

# 优先使用函数式风格

## 操作
使用函数式模式而非类。
```

## 选项

- `--domain <name>`: 仅导出指定domain的instincts
- `--min-confidence <n>`: 最低置信度阈值
- `--output <file>`: 输出文件路径（省略时打印到stdout）
- `--scope <project|global|all>`: 导出范围（默认: `all`）
