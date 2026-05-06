---
description: 设置优先的包管理器（npm/pnpm/yarn/bun）
disable-model-invocation: true
---

# 包管理器设置

设置项目或全局优先使用的包管理器。

## 使用方法

```bash
# 检测当前包管理器
node scripts/setup-package-manager.js --detect

# 指定全局设置
node scripts/setup-package-manager.js --global pnpm

# 指定项目设置
node scripts/setup-package-manager.js --project bun

# 列出可用的包管理器
node scripts/setup-package-manager.js --list
```

## 检测优先级

确定使用的包管理器时，按以下顺序检查：

1. **环境变量**: `CLAUDE_PACKAGE_MANAGER`
2. **项目设置**: `.claude/package-manager.json`
3. **package.json**: `packageManager` 字段
4. **锁文件**: package-lock.json、yarn.lock、pnpm-lock.yaml、bun.lockb 的存在
5. **全局设置**: `~/.claude/package-manager.json`
6. **回退**: 第一个可用的包管理器（pnpm > bun > yarn > npm）

## 配置文件

### 全局设置
```json
// ~/.claude/package-manager.json
{
  "packageManager": "pnpm"
}
```

### 项目设置
```json
// .claude/package-manager.json
{
  "packageManager": "bun"
}
```

### package.json
```json
{
  "packageManager": "pnpm@8.6.0"
}
```

## 环境变量

设置 `CLAUDE_PACKAGE_MANAGER` 将覆盖所有其他检测方法：

```bash
# Windows (PowerShell)
$env:CLAUDE_PACKAGE_MANAGER = "pnpm"

# macOS/Linux
export CLAUDE_PACKAGE_MANAGER=pnpm
```

## 运行检测

确认当前包管理器检测结果，执行以下命令：

```bash
node scripts/setup-package-manager.js --detect
```
