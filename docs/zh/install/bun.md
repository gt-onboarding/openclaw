---
title: Bun
summary: "Bun 工作流（实验性）：安装与 pnpm 相比的注意事项与坑点"
read_when:
  - 你想要最快的本地开发迭代速度（bun + watch）
  - 你遇到了 Bun 在 install/patch/lifecycle 脚本上的问题
---

<div id="bun-experimental">
  # Bun（实验性）
</div>

目标：在不偏离 pnpm 工作流的前提下，使用 **Bun** 运行本仓库（可选，但不建议用于 WhatsApp/Telegram）。

⚠️ **不推荐用于 Gateway 运行时环境**（在 WhatsApp/Telegram 上存在 bug）。生产环境请使用 Node。

<div id="status">
  ## 状态
</div>

- Bun 是一个可选的本地运行时，可用于直接运行 TypeScript（`bun run …`、`bun --watch …`）。
- `pnpm` 目前是默认的构建工具，并且仍然得到完整支持（也被部分文档工具使用）。
- Bun 无法使用 `pnpm-lock.yaml`，并会忽略该文件。

<div id="install">
  ## 安装
</div>

默认：

```sh
bun install
```

注意：`bun.lock`/`bun.lockb` 已被列入 `.gitignore`，因此无论怎样都不会给仓库带来额外的变更。如果你希望*完全不写入锁文件*：

```sh
bun install --no-save
```


<div id="build-test-bun">
  ## 构建/测试（Bun）
</div>

```sh
bun run build
bun run vitest run
```


<div id="bun-lifecycle-scripts-blocked-by-default">
  ## Bun 生命周期脚本（默认被阻止）
</div>

除非显式信任（`bun pm untrusted` / `bun pm trust`），Bun 可能会阻止依赖项的生命周期脚本。
对于这个仓库，常见被阻止的脚本并不是必需的：

* `@whiskeysockets/baileys` `preinstall`：检查 Node 主版本号 &gt;= 20（我们运行的是 Node 22+）。
* `protobufjs` `postinstall`：输出关于不兼容版本方案的警告（不会生成任何构建产物）。

如果你遇到了确实需要这些脚本才能解决的真实运行时问题，请显式信任它们：

```sh
bun pm trust @whiskeysockets/baileys protobufjs
```


<div id="caveats">
  ## 注意事项
</div>

- 一些脚本目前仍然将 pnpm 写死在代码里（例如 `docs:build`、`ui:*`、`protocol:check`）。目前请使用 pnpm 来运行这些脚本。