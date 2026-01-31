---
title: 协议
summary: "Gateway WebSocket 协议：握手、帧、版本管理"
read_when:
  - 实现或更新 Gateway WS 客户端时
  - 调试协议不匹配或连接失败问题时
  - 重新生成协议 schema/模型时
---

<div id="gateway-protocol-websocket">
  # Gateway 协议（WebSocket）
</div>

Gateway WS 协议是 OpenClaw 的**唯一控制平面与节点传输通道**。
所有客户端（CLI、web UI、macOS 应用、iOS/Android 节点、无头
节点）都通过 WebSocket 连接，并在握手阶段声明自己的**角色（role）**和**scope**。

<div id="transport">
  ## 传输
</div>

- WebSocket，文本帧，负载为 JSON。
- 第一帧**必须**是一个 `connect` 请求。

<div id="handshake-connect">
  ## 握手（连接）
</div>

Gateway → 客户端（预连接质询）：

```json
{
  "type": "event",
  "event": "connect.challenge",
  "payload": { "nonce": "…", "ts": 1737264000000 }
}
```

客户端 → Gateway：

```json
{
  "type": "req",
  "id": "…",
  "method": "connect",
  "params": {
    "minProtocol": 3,
    "maxProtocol": 3,
    "client": {
      "id": "cli",
      "version": "1.2.3",
      "platform": "macos",
      "mode": "operator"
    },
    "role": "operator",
    "scopes": ["operator.read", "operator.write"],
    "caps": [],
    "commands": [],
    "permissions": {},
    "auth": { "token": "…" },
    "locale": "en-US",
    "userAgent": "openclaw-cli/1.2.3",
    "device": {
      "id": "device_fingerprint",
      "publicKey": "…",
      "signature": "…",
      "signedAt": 1737264000000,
      "nonce": "…"
    }
  }
}
```

Gateway → 客户端：

```json
{
  "type": "res",
  "id": "…",
  "ok": true,
  "payload": { "type": "hello-ok", "protocol": 3, "policy": { "tickIntervalMs": 15000 } }
}
```

设备令牌签发后，`hello-ok` 还会包含：

```json
{
  "auth": {
    "deviceToken": "…",
    "role": "operator",
    "scopes": ["operator.read", "operator.write"]
  }
}
```


<div id="node-example">
  ### 节点示例
</div>

```json
{
  "type": "req",
  "id": "…",
  "method": "connect",
  "params": {
    "minProtocol": 3,
    "maxProtocol": 3,
    "client": {
      "id": "ios-node",
      "version": "1.2.3",
      "platform": "ios",
      "mode": "node"
    },
    "role": "node",
    "scopes": [],
    "caps": ["camera", "canvas", "screen", "location", "voice"],
    "commands": ["camera.snap", "canvas.navigate", "screen.record", "location.get"],
    "permissions": { "camera.capture": true, "screen.record": false },
    "auth": { "token": "…" },
    "locale": "en-US",
    "userAgent": "openclaw-ios/1.2.3",
    "device": {
      "id": "device_fingerprint",
      "publicKey": "…",
      "signature": "…",
      "signedAt": 1737264000000,
      "nonce": "…"
    }
  }
}
```


<div id="framing">
  ## 帧结构
</div>

- **请求**：`{type:"req", id, method, params}`  
- **响应**：`{type:"res", id, ok, payload|error}`  
- **事件**：`{type:"event", event, payload, seq?, stateVersion?}`

会产生副作用的方法必须提供**幂等性键**（参见 schema）。

<div id="roles-scopes">
  ## 角色和 scope
</div>

<div id="roles">
  ### 角色
</div>

- `operator` = 控制平面客户端（CLI/UI/自动化）。
- `node` = 能力宿主节点（camera/screen/canvas/system.run）。

<div id="scopes-operator">
  ### 作用域（operator）
</div>

常见作用域：

- `operator.read`
- `operator.write`
- `operator.admin`
- `operator.approvals`
- `operator.pairing`

<div id="capscommandspermissions-node">
  ### 功能 / 命令 / 权限（节点）
</div>

节点在连接时声明其能力：

- `caps`：高层能力类别。
- `commands`：用于调用的命令允许列表。
- `permissions`：细粒度开关（例如 `screen.record`、`camera.capture`）。

Gateway 将这些视为**声明**，并在服务端执行允许列表策略。

<div id="presence">
  ## 在线状态（Presence）
</div>

- `system-presence` 返回以设备标识为键的条目。
- 在线状态条目包含 `deviceId`、`roles` 和 `scopes`，因此 UI 即使在设备同时以 **operator** 和 **节点** 身份连接时，也只需为每台设备显示一行。

<div id="node-helper-methods">
  ### 节点辅助方法
</div>

- 节点可以调用 `skills.bins` 来获取当前的技能可执行文件列表，
  用于自动允许（auto-allow）检查。

<div id="exec-approvals">
  ## Exec 审批
</div>

- 当某个 exec 请求需要审批时，Gateway 会广播 `exec.approval.requested`。
- Operator 客户端通过调用 `exec.approval.resolve` 来完成审批（需要 `operator.approvals` scope）。

<div id="versioning">
  ## 版本管理
</div>

- `PROTOCOL_VERSION` 位于 `src/gateway/protocol/schema.ts`。
- 客户端会发送 `minProtocol` 和 `maxProtocol`；服务器会拒绝不在兼容范围内的版本。
- Schema 和 model 均从 TypeBox 定义生成：
  - `pnpm protocol:gen`
  - `pnpm protocol:gen:swift`
  - `pnpm protocol:check`

<div id="auth">
  ## 认证
</div>

- 如果设置了 `OPENCLAW_GATEWAY_TOKEN`（或 `--token`），则 `connect.params.auth.token`
  必须与其匹配，否则套接字将被关闭。
- 配对完成后，Gateway 会签发一个**设备令牌（device token）**，其作用范围限定为连接角色和相关 scopes。该令牌会在
  `hello-ok.auth.deviceToken` 中返回，客户端应将其持久化保存，以便后续连接使用。
- 可以通过 `device.token.rotate` 和 `device.token.revoke` 轮换/吊销设备令牌（需要 `operator.pairing` scope）。

<div id="device-identity-pairing">
  ## 设备身份与配对
</div>

- 节点应包含由密钥对指纹派生出的稳定设备身份（`device.id`）。
- Gateway 会按设备 + 角色发放令牌。
- 除非启用了本地自动批准，否则新设备 ID 需要配对审批。
- **本地** 连接包括回环地址以及 Gateway 主机自身的 tailnet 地址
  （因此同一主机上的 tailnet 绑定仍然可以自动批准）。
- 所有 WS 客户端在 `connect` 时必须携带 `device` 身份信息（无论是操作员还是节点）。
  只有在启用 `gateway.controlUi.allowInsecureAuth` 时，Control UI 才能省略该信息
  （或在紧急 “break-glass” 用途中启用 `gateway.controlUi.dangerouslyDisableDeviceAuth`）。
- 非本地连接必须对服务器提供的 `connect.challenge` 随机数（nonce）进行签名。

<div id="tls-pinning">
  ## TLS 与证书固定（pinning）
</div>

- WS 连接支持通过 TLS 建立。
- 客户端可以选择对 Gateway 的证书指纹进行固定（参见 `gateway.tls`
  配置以及 `gateway.remote.tlsFingerprint` 或 CLI `--tls-fingerprint`）。

<div id="scope">
  ## 作用范围
</div>

该协议公开 **完整的 Gateway api**（状态、通道、模型、聊天、智能体、会话、节点、审批等）。其具体接口范围由 `src/gateway/protocol/schema.ts` 中的 TypeBox 模式定义。