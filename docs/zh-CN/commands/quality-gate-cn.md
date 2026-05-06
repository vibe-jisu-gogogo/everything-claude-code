---
description: 按需对文件或项目范围运行 ECC quality pipeline 并报告 remediation steps。
---

# Quality Gate 命令

按需对文件或项目范围运行 ECC quality pipeline。

## 用法

`/quality-gate [path|.] [--fix] [--strict]`

- 默认目标：当前目录 (`.`)
- `--fix`：在已配置的地方允许 auto-format/fix
- `--strict`：在支持的地方警告即失败

## Pipeline

1. 检测目标的 language/tooling。
2. 运行 formatter checks。
3. 在可用时运行 lint/type checks。
4. 生成简洁的 remediation list。

## 备注

此命令镜像 hook behavior，但由操作员调用。

## 参数

$ARGUMENTS:
- `[path|.]` 可选的 target path
- `--fix` 可选
- `--strict` 可选
