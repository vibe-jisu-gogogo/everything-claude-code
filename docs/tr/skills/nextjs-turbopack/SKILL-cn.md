---
name: nextjs-turbopack
description: Next.js 16+ and Turbopack — incremental bundling, FS caching, dev speed, and when to use Turbopack vs webpack.
origin: ECC
---

# Next.js 和 Turbopack

Next.js 16+ 默认使用 Turbopack 进行本地开发：这是一个用 Rust 编写的增量 bundler，可显著加快开发启动和 hot update 速度。

## 何时使用

- **Turbopack（默认开发模式）**：用于日常开发。特别是在大型应用中提供更快的冷启动和 HMR。
- **Webpack（传统开发模式）**：仅在遇到 Turbopack bug 或依赖开发环境中 webpack 特定插件时使用。使用 `--webpack` 禁用（或根据您的 Next.js 版本使用 `--no-turbopack`；请查看您版本的文档）。
- **生产环境**：生产构建行为（`next build`）可能使用 Turbopack 或 webpack，具体取决于 Next.js 版本；请检查您版本的官方 Next.js 文档。

适用于以下情况：开发或调试 Next.js 16+ 应用，诊断缓慢的开发启动或 HMR，或优化生产 bundle。

## 工作原理

- **Turbopack**：Next.js 开发环境的增量 bundler。使用文件系统缓存，因此重启速度更快（例如，在大型项目中快 5-14 倍）。
- **开发环境默认**：从 Next.js 16 开始，`next dev` 默认使用 Turbopack，除非被禁用。
- **文件系统缓存**：重启会重用之前的工作；缓存通常位于 `.next` 下；基本使用不需要额外配置。
- **Bundle Analyzer（Next.js 16.1+）**：实验性 Bundle Analyzer 用于检查输出并找出重型依赖项；通过配置或实验性标志启用（请查看您版本的 Next.js 文档）。

## 示例

### 命令

```bash
next dev
next build
next start
```

### 使用方法

运行 `next dev` 使用 Turbopack 进行本地开发。使用 Bundle Analyzer 优化 code-splitting 并减少大型依赖项（请参阅 Next.js 文档）。尽可能优先使用 App Router 和 server component。

## 最佳实践

- 保持使用最新的 Next.js 16.x 版本，以获得稳定的 Turbopack 和缓存行为。
- 如果开发环境运行缓慢，请确保您使用的是 Turbopack（默认），并且缓存没有被不必要地清除。
- 对于生产 bundle 大小问题，请使用您版本的官方 Next.js bundle 分析工具。
