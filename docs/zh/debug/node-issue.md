---
title: Node 问题
summary: Node + tsx "__name is not a function" 崩溃说明与解决方法
read_when:
  - 调试仅使用 Node 的开发脚本或 watch 模式故障
  - 调查 OpenClaw 中 tsx/esbuild 加载器崩溃问题
---

<div id="node-tsx-__name-is-not-a-function-crash">
  # Node + tsx 导致 "__name is not a function" 崩溃
</div>

<div id="summary">
  ## 摘要
</div>

使用 Node 配合 `tsx` 运行 OpenClaw 时，在启动阶段会崩溃，并输出如下错误：

```
[openclaw] Failed to start CLI: TypeError: __name is not a function
    at createSubsystemLogger (.../src/logging/subsystem.ts:203:25)
    at .../src/agents/auth-profiles/constants.ts:25:20
```

这是在将开发脚本从 Bun 切换到 `tsx`（提交 `2871657e`，2026-01-06）之后开始发生的。同样的运行时路径在 Bun 下可以正常工作。


<div id="environment">
  ## 环境
</div>

- Node：v25.x（在 v25.3.0 上复现）
- tsx：4.21.0
- 操作系统：macOS（很可能在其他运行 Node 25 的平台上也可以复现）

<div id="repro-node-only">
  ## 复现（仅限 Node.js）
</div>

```bash
# 在仓库根目录下
node --version
pnpm install
node --import tsx src/entry.ts status
```


<div id="minimal-repro-in-repo">
  ## 仓库中的最小可复现示例
</div>

```bash
node --import tsx scripts/repro/tsx-name-repro.ts
```


<div id="node-version-check">
  ## Node 版本检查
</div>

- Node 25.3.0：未通过
- Node 22.22.0（Homebrew `node@22`）：未通过
- Node 24：此环境尚未安装；需要验证

<div id="notes-hypothesis">
  ## 备注 / 假设
</div>

- `tsx` 使用 esbuild 来转换 TS/ESM。esbuild 的 `keepNames` 选项会生成一个 `__name` 辅助函数，并用 `__name(...)` 来包装函数定义。
- 崩溃表明在运行时 `__name` 存在但不是一个函数，这意味着在 Node 25 的 loader 路径中，该模块的这个辅助函数要么缺失，要么被覆盖。
- 在其他 esbuild 使用场景中，也曾在该辅助函数缺失或被重写时报告过类似的 `__name` 问题。

<div id="regression-history">
  ## 回归历史
</div>

- `2871657e`（2026-01-06）：将脚本从 Bun 切换为 tsx，使 Bun 变为可选项。
- 在此之前（使用 Bun 路径时），`openclaw status` 和 `gateway:watch` 是可以正常工作的。

<div id="workarounds">
  ## 解决方案
</div>

- 在开发脚本中使用 Bun（目前的临时回退方案）。
- 使用 Node + tsc 的 watch，然后运行编译后的输出：
  ```bash
  pnpm exec tsc --watch --preserveWatchOutput
  node --watch openclaw.mjs status
  ```
- 本地已确认：`pnpm exec tsc -p tsconfig.json` + `node openclaw.mjs status` 在 Node 25 上可以正常工作。
- 如有可能，在 TS loader 中禁用 esbuild 的 keepNames（可防止插入 `__name` 辅助函数）；`tsx` 目前尚未暴露该选项。
- 在 Node LTS（22/24）上使用 `tsx` 进行测试，以确认该问题是否为 Node 25 特有。

<div id="references">
  ## 参考资料
</div>

- https://opennext.js.org/cloudflare/howtos/keep_names
- https://esbuild.github.io/api/#keep-names
- https://github.com/evanw/esbuild/issues/1031

<div id="next-steps">
  ## 后续步骤
</div>

- 在 Node 22/24 上复现，以确认是否为 Node 25 的回归。
- 测试 `tsx` nightly 版本，或者如果已知存在回归，则固定到较早版本。
- 如果在 Node LTS 上也能复现，向上游提交一个最小复现示例，并附上带有 `__name` 的堆栈跟踪。