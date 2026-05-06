---
description: 配置你偏好的 package manager (npm/pnpm/yarn/bun)
disable-model-invocation: true
---

# Package Manager 配置

为当前项目或全局配置你偏好的 package manager。

## 使用方法

```bash
# 检测当前的 package manager
node scripts/setup-package-manager.js --detect

# 设置全局偏好
node scripts/setup-package-manager.js --global pnpm

# 设置项目偏好
node scripts/setup-package-manager.js --project bun

# 列出可用的 package managers
node scripts/setup-package-manager.js --list
```

## 检测优先级

确定使用哪个 package manager 时，按以下顺序检查：

1. **环境变量**：`CLAUDE_PACKAGE_MANAGER`
2. **项目配置**：`.claude/package-manager.json`
3. **package.json**：`packageManager` 字段
4. **锁定文件**：是否存在 package-lock.json、yarn.lock、pnpm-lock.yaml 或 bun.lockb
5. **全局配置**：`~/.claude/package-manager.json`
6. **回退方案**：第一个可用的 package manager (pnpm > bun > yarn > npm)

## 配置文件

### 全局配置
```json
// ~/.claude/package-manager.json
{
  "packageManager": "pnpm"
}
```

### 项目配置
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

设置 `CLAUDE_PACKAGE_MANAGER` 可覆盖所有其他检测方法：

```bash
# Windows (PowerShell)
$env:CLAUDE_PACKAGE_MANAGER = "pnpm"

# macOS/Linux
export CLAUDE_PACKAGE_MANAGER=pnpm
```

## 运行检测

查看当前 package manager 检测结果，运行：

```bash
node scripts/setup-package-manager.js --detect
```
