---
name: build-fix
description: 用最小的安全变更渐进式修复 build 和类型错误。
---

# Build 错误修复

用最小的安全变更渐进式修复 build 和类型错误。

## 第1步：检测 Build 系统

识别项目的 build 工具并执行 build：

| 识别标准 | Build 命令 |
|-----------|---------------|
| `package.json` 包含 `build` 脚本 | `npm run build` 或 `pnpm build` |
| `tsconfig.json` (仅 TypeScript) | `npx tsc --noEmit` |
| `Cargo.toml` | `cargo build 2>&1` |
| `pom.xml` | `mvn compile` |
| `build.gradle` | `./gradlew compileJava` |
| `go.mod` | `go build ./...` |
| `pyproject.toml` | `python -m compileall .` 或 `mypy .` |

## 第2步：解析并分组错误

1. 执行 build 命令并捕获 stderr
2. 按文件路径对错误进行分组
3. 按依赖顺序排序（先修复 import/类型错误，再修复逻辑错误）
4. 统计总错误数以跟踪进度

## 第3步：修复循环（一次一个错误）

对每个错误：

1. **读取文件** — 使用 Read 工具查看错误前后 10 行的上下文
2. **诊断** — 识别根本原因（缺失的 import、错误的类型、语法错误）
3. **最小化修复** — 使用 Edit 工具应用解决错误所需的最小变更
4. **重新执行 Build** — 确认错误已解决且未产生新错误
5. **继续处理** — 继续处理剩余错误

## 第4步：安全机制

在以下情况下请求用户确认：

- 修复**产生的错误比解决的更多**时
- **相同错误在尝试 3 次后仍然存在**时（可能是更深层的问题）
- 修复**需要架构变更**时（不是简单的 build 修复）
- Build 错误源于**缺失的依赖**时（需要 `npm install`、`cargo add` 等）

## 第5步：总结

显示结果：
- 已修复的错误（包含文件路径）
- 剩余的错误（如果有）
- 新产生的错误（应该为 0）
- 对未解决问题的下一步建议

## 恢复策略

| 情况 | 措施 |
|-----------|--------|
| 模块/import 缺失 | 检查包是否已安装，并建议安装命令 |
| 类型不匹配 | 检查两边的类型定义，并修正更窄的类型 |
| 循环依赖 | 用 import 图识别循环并建议拆分 |
| 版本冲突 | 检查 `package.json` / `Cargo.toml` 中的版本约束 |
| Build 工具配置错误 | 检查配置文件并与正常工作的默认值比较 |

为安全起见，一次只修复一个错误。优先选择最小的 diff 而非重构。
