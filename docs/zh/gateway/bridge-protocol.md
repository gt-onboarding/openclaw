---
title: Bridge 协议
summary: "Bridge 协议（旧版节点）：TCP JSONL、配对、基于 scope 的 RPC"
read_when:
  - 构建或调试节点客户端（iOS/Android/macOS 节点模式）
  - 排查配对或 Bridge 认证失败
  - 审计 Gateway 对外暴露的节点接口面
---

<div id="bridge-protocol-legacy-node-transport">
  # Bridge 协议（旧版节点传输）
</div>

Bridge 协议是一种**旧版**节点传输方式（TCP JSONL）。新的节点客户端
应改用统一的 Gateway WebSocket 协议。

如果你正在构建 operator 或节点客户端，请使用
[Gateway 协议](/zh/gateway/protocol)。

**注意：** 当前的 OpenClaw 版本已不再附带 TCP bridge 监听器；本文档仅作为历史参考而保留。
旧版的 `bridge.*` 配置键已不再属于配置 schema 的一部分。

<div id="why-we-have-both">
  ## 为什么两者都存在
</div>

* **安全边界**：Bridge 只暴露一个精简的允许列表，而不是完整的 Gateway API 接口范围。
* **配对 + 节点身份**：节点接入由 Gateway 负责，并与每个节点的 token 绑定。
* **发现体验（UX）**：节点可以通过局域网（LAN）中的 Bonjour 发现 Gateway，或通过 tailnet 直接连接。
* **回环 WS**：完整的 WS 控制平面会保持在本地，除非通过 SSH 建立隧道转发。

<div id="transport">
  ## 传输
</div>

* 使用 TCP，每行一个 JSON 对象（JSONL）。
* 可选 TLS（当 `bridge.tls.enabled` 为 true 时启用）。
* 旧版默认监听端口为 `18790`（当前版本不会启动 TCP bridge）。

启用 TLS 时，用于服务发现的 TXT 记录会包含 `bridgeTls=1` 以及
`bridgeTlsSha256`，以便节点可以对证书进行固定（pinning）。

<div id="handshake-pairing">
  ## 握手与配对
</div>

1. 客户端发送 `hello`，其中包含节点元数据和令牌（如果已完成配对）。
2. 如果尚未配对，Gateway 返回 `error`（`NOT_PAIRED` / `UNAUTHORIZED`）。
3. 客户端发送 `pair-request`。
4. Gateway 等待批准，然后发送 `pair-ok` 和 `hello-ok`。

`hello-ok` 返回 `serverName`，并且可能包含 `canvasHostUrl`。

<div id="frames">
  ## 帧（Frames）
</div>

客户端 → Gateway：

* `req` / `res`: 带 scope 的 Gateway RPC 调用（chat, sessions, config, health, voicewake, skills.bins）
* `event`: 节点信号（语音转录、智能体请求、聊天订阅、exec 生命周期）

Gateway → 客户端：

* `invoke` / `invoke-res`: 节点命令（`canvas.*`, `camera.*`, `screen.record`,
  `location.get`, `sms.send`)
* `event`: 已订阅会话的聊天更新
* `ping` / `pong`: 保活

旧版 allowlist 访问控制逻辑位于 `src/gateway/server-bridge.ts` 中（已移除）。

<div id="exec-lifecycle-events">
  ## Exec 生命周期事件
</div>

节点可以触发 `exec.finished` 或 `exec.denied` 事件，用于上报 `system.run` 活动。
这些事件会在 Gateway 中映射为系统事件。（旧版节点可能仍然会触发 `exec.started`。）

负载字段（除特别说明外均为可选）：

* `sessionKey`（必填）：用于接收该系统事件的智能体会话。
* `runId`：用于分组的唯一 exec ID。
* `command`：原始或已格式化的命令字符串。
* `exitCode`、`timedOut`、`success`、`output`：完成详情（仅在 finished 时包含）。
* `reason`：拒绝原因（仅在 denied 时包含）。

<div id="tailnet-usage">
  ## Tailnet 使用
</div>

* 将 bridge 绑定到 tailnet IP：在 `~/.openclaw/openclaw.json` 中设置 `bridge.bind: "tailnet"`。
* 客户端通过 MagicDNS 名称或 tailnet IP 进行连接。
* Bonjour **不会**跨越不同网络；在需要时请使用手动指定主机/端口或广域 DNS‑SD。

<div id="versioning">
  ## 版本
</div>

Bridge 当前处于 **隐式 v1** 状态（无最小/最大版本协商）。预期保持向后兼容；在进行任何破坏性变更之前，应先在 Bridge 协议中添加一个版本字段。