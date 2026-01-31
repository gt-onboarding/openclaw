---
title: RPC
summary: "用于外部 CLI（signal-cli、imsg）和 Gateway 使用模式的 RPC 适配器"
read_when:
  - 在添加或修改外部 CLI 集成时
  - 在调试 RPC 适配器（signal-cli、imsg）时
---

<div id="rpc-adapters">
  # RPC 适配器
</div>

OpenClaw 通过 JSON-RPC 集成外部 CLI 工具。当前使用两种模式。

<div id="pattern-a-http-daemon-signal-cli">
  ## 模式 A：HTTP 守护进程（signal-cli）
</div>

* `signal-cli` 以守护进程方式运行，通过 HTTP 提供 JSON-RPC。
* 事件流通过 SSE 暴露（`/api/v1/events`）。
* 健康检查接口：`/api/v1/check`。
* 当 `channels.signal.autoStart=true` 时，OpenClaw 负责其生命周期管理。

有关安装配置和端点说明，请参阅 [Signal](/zh/channels/signal)。

<div id="pattern-b-stdio-child-process-imsg">
  ## 模式 B：stdio 子进程（imsg）
</div>

* OpenClaw 以子进程方式启动 `imsg rpc`。
* JSON-RPC 通过 stdin/stdout 以按行分隔的方式传输（每行一个 JSON 对象）。
* 不需要 TCP 端口，也不需要守护进程。

使用的核心方法：

* `watch.subscribe` → 通知（`method: "message"`）
* `watch.unsubscribe`
* `send`
* `chats.list`（用于探测/诊断）

有关设置与地址指定方式（推荐使用 `chat_id`），请参见 [iMessage](/zh/channels/imessage)。

<div id="adapter-guidelines">
  ## 适配器规范
</div>

* 由 Gateway 管理进程（启动/停止与提供方生命周期绑定）。
* 保持 RPC 客户端的健壮性：设置超时、在进程退出时重启。
* 优先使用稳定 ID（例如 `chat_id`），而不是用于展示的字符串。