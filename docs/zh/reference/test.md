---
title: 测试
summary: "如何在本地运行测试（使用 vitest）以及何时使用 force/coverage 模式"
read_when:
  - 运行或修复测试时
---

<div id="tests">
  # 测试
</div>

* 完整测试套件（测试集、联机测试、Docker）：[测试](/zh/testing)

* `pnpm test:force`：终止任何占用默认控制端口的残留 Gateway 进程，然后在独立的 Gateway 端口上运行完整 Vitest 测试套件，避免服务器端测试与正在运行的实例发生端口冲突。当之前的 Gateway 运行导致端口 18789 仍被占用时，请使用此命令。

* `pnpm test:coverage`：以 V8 覆盖率模式运行 Vitest。全局阈值为：代码行、分支、函数与语句均需达到 70%。覆盖率统计会排除重度集成的入口点（CLI 绑定、Gateway/Telegram 桥接、webchat 静态服务器），以便将目标聚焦在可进行单元测试的逻辑上。

* `pnpm test:e2e`：运行 Gateway 端到端冒烟测试（多实例 WS/HTTP/节点配对）。

* `pnpm test:live`：运行提供方在线实测（minimax/zai）。需要 API 密钥，并设置 `LIVE=1`（或提供方特定的 `*_LIVE_TEST=1`）以取消跳过。

<div id="model-latency-bench-local-keys">
  ## 模型延迟基准（本地密钥）
</div>

脚本：[`scripts/bench-model.ts`](https://github.com/openclaw/openclaw/blob/main/scripts/bench-model.ts)

使用方式：

* `source ~/.profile && pnpm tsx scripts/bench-model.ts --runs 10`
* 可选环境变量：`MINIMAX_API_KEY`、`MINIMAX_BASE_URL`、`MINIMAX_MODEL`、`ANTHROPIC_API_KEY`
* 默认提示语：“Reply with a single word: ok. No punctuation or extra text.”

最近一次运行（2025-12-31，运行 20 次）：

* minimax 中位延迟 1279 ms（最小 1114，最大 2431）
* opus 中位延迟 2454 ms（最小 1224，最大 3170）

<div id="onboarding-e2e-docker">
  ## Onboarding 端到端流程（Docker）
</div>

Docker 是可选项；只有在需要对容器化的 Onboarding 做冒烟测试时才需要。

在干净的 Linux 容器中运行完整冷启动流程：

```bash
scripts/e2e/onboard-docker.sh
```

此脚本通过伪终端驱动交互式向导，校验配置/工作区/会话文件，然后启动 Gateway 并运行 `openclaw health`。

<div id="qr-import-smoke-docker">
  ## QR 导入冒烟测试（Docker）
</div>

确保在 Docker 环境中，`qrcode-terminal` 能在 Node 22+ 下成功加载：

```bash
pnpm test:docker:qr
```
