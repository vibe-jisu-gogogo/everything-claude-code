---
name: bun-runtime
description: 作为 runtime、package manager、bundler 和 test runner 的 Bun。何时选择 Bun 还是 Node、迁移注意事项，以及 Vercel 支持说明。
---

# Bun Runtime

Bun 是一款快速的一体化 JavaScript runtime 及工具集，包含 runtime、package manager、bundler 和 test runner。

## 适用场景

- **优先选择 Bun** 适用于：新的 JS/TS 项目、对安装/运行速度要求较高的脚本、使用 Bun runtime 的 Vercel 部署，以及需要单一工具链（运行+安装+测试+构建）的场景。
- **优先选择 Node** 适用于：需要最大化生态兼容性的场景、依赖假设使用 Node 的老旧工具，或者依赖存在已知 Bun 兼容问题的情况。

适用时机：采用 Bun 技术栈、从 Node 迁移、编写或调试 Bun 脚本/测试，或者在 Vercel 或其他平台上配置 Bun 时。

## 工作原理

- **Runtime**：可无缝替代 Node 的兼容 runtime（基于 JavaScriptCore 构建，使用 Zig 语言实现）。
- **Package manager**：`bun install` 速度远快于 npm/yarn。当前版本的 Bun 默认使用文本格式的锁文件 `bun.lock`；旧版本使用二进制格式的 `bun.lockb`。
- **Bundler**：内置面向应用和库的 bundler 与转译器。
- **Test runner**：内置类 Jest API 的 `bun test` 功能。

**从 Node 迁移**：将 `node script.js` 替换为 `bun run script.js` 或 `bun script.js`。使用 `bun install` 替代 `npm install`，绝大多数 package 都可正常工作。对于 npm scripts 使用 `bun run` 执行；使用 `bun x` 实现类似 npx 的一次性运行。支持 Node 内置模块；存在对应 Bun API 时优先使用以获得更好性能。

**Vercel**：在项目设置中将 runtime 设置为 Bun。构建命令：`bun run build` 或 `bun build ./src/index.ts --outdir=dist`。安装依赖使用 `bun install --frozen-lockfile` 以实现可复现的部署。

## 示例

### 运行与安装

```bash
# 安装依赖（创建/更新 bun.lock 或 bun.lockb）
bun install

# 运行脚本或文件
bun run dev
bun run src/index.ts
bun src/index.ts
```

### 脚本与环境变量

```bash
bun run --env-file=.env dev
FOO=bar bun run script.ts
```

### 测试

```bash
bun test
bun test --watch
```

```typescript
// test/example.test.ts
import { expect, test } from "bun:test";

test("add", () => {
  expect(1 + 2).toBe(3);
});
```

### Runtime API

```typescript
const file = Bun.file("package.json");
const json = await file.json();

Bun.serve({
  port: 3000,
  fetch(req) {
    return new Response("Hello");
  },
});
```

## 最佳实践

- 提交锁文件（`bun.lock` 或 `bun.lockb`）以实现可复现的安装。
- 脚本优先使用 `bun run` 执行。对于 TypeScript，Bun 原生支持运行 `.ts` 文件。
- 保持依赖最新；Bun 及其生态迭代速度很快。
