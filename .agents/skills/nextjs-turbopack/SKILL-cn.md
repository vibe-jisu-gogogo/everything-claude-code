---
name: nextjs-turbopack
description: Next.js 16+ 和 Turbopack — 增量打包、文件系统缓存、开发速度优化，以及 Turbopack 与 webpack 的适用场景对比。
---

# Next.js 与 Turbopack

Next.js 16+ 本地开发默认使用 Turbopack：这是一个用 Rust 编写的增量打包工具，能够显著提升开发服务启动速度和热更新效率。

## 适用场景

- **Turbopack（默认开发模式）**：用于日常开发工作。冷启动和 HMR（模块热替换）速度更快，尤其在大型应用中优势明显。
- **Webpack（传统开发模式）**：仅当你遇到 Turbopack 错误，或者开发环境依赖仅支持 webpack 的插件时使用。通过 `--webpack` 参数禁用 Turbopack（部分 Next.js 版本需使用 `--no-turbopack`，请参考对应版本的官方文档）。
- **生产环境**：生产构建行为（`next build`）使用 Turbopack 还是 webpack 取决于 Next.js 版本，请查阅你使用版本的 Next.js 官方文档。

适用场景：开发或调试 Next.js 16+ 应用、排查开发服务启动慢或 HMR 异常问题，或者优化生产构建包体积。

## 工作原理

- **Turbopack**：Next.js 开发环境使用的增量打包工具。基于文件系统缓存实现，重启速度提升显著（例如大型项目中提速 5–14 倍）。
- **开发环境默认启用**：从 Next.js 16 开始，`next dev` 命令默认启用 Turbopack，除非手动禁用。
- **文件系统缓存**：服务重启时会复用之前的构建结果；缓存通常存放在 `.next` 目录下；基础使用无需额外配置。
- **Bundle Analyzer（Next.js 16.1+）**：实验性的包体积分析工具，用于检查构建输出和识别体积过大的依赖；通过配置或实验性标志启用（请参考对应版本的 Next.js 文档）。

## 示例

### 命令

```bash
next dev
next build
next start
```

### 使用方法

执行 `next dev` 即可启动带 Turbopack 的本地开发服务。使用 Bundle Analyzer（参考 Next.js 文档）优化代码拆分并精简大型依赖。尽可能优先使用 App Router 和服务端组件。

## 最佳实践

- 尽量使用最新的 Next.js 16.x 版本，以获得稳定的 Turbopack 和缓存功能。
- 如果开发环境运行缓慢，请确认你正在使用 Turbopack（默认配置），且缓存没有被不必要地清理。
- 如果遇到生产构建包体积问题，使用对应版本的 Next.js 官方包分析工具。
