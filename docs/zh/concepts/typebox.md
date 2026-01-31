---
title: Typebox
summary: "将 TypeBox schema 作为 Gateway 协议的单一事实来源"
read_when:
  - 更新协议 schema 或进行代码生成时
---

<div id="typebox-as-protocol-source-of-truth">
  # 以 TypeBox 作为协议的单一事实来源（source of truth）
</div>

最后更新日期：2026-01-10

TypeBox 是一个专为 TypeScript 设计的 schema 库。我们用它来定义 **Gateway WebSocket 协议**（握手、请求/响应、服务器事件）。这些 schema 支持 **运行时校验**、**JSON Schema 导出**，以及为 macOS 应用进行 **Swift 代码生成**。一个单一事实来源，其余全部由此自动生成。

如果你想先了解更高层次的协议背景，请从
[Gateway 架构](/zh/concepts/architecture) 开始。

<div id="mental-model-30-seconds">
  ## 心智模型（30 秒速览）
</div>

每条 Gateway WS 消息都属于以下三种帧之一：

* **Request**：`{ type: "req", id, method, params }`
* **Response**：`{ type: "res", id, ok, payload | error }`
* **Event**：`{ type: "event", event, payload, seq?, stateVersion? }`

第一帧**必须**是一个 `connect` 请求。之后，客户端可以调用
方法（例如 `health`、`send`、`chat.send`），并订阅事件（例如
`presence`、`tick`、`agent`）。

连接流程（最简版）：

```
Client                    Gateway
  |---- req:connect -------->|
  |<---- res:hello-ok --------|
  |<---- event:tick ----------|
  |---- req:health ---------->|
  |<---- res:health ----------|
```

常用方法与事件：

| 分类 | 示例                                                        | 备注                         |
| -- | --------------------------------------------------------- | -------------------------- |
| 核心 | `connect`, `health`, `status`                             | `connect` 必须排在第一位          |
| 消息 | `send`, `poll`, `agent`, `agent.wait`                     | 有副作用的调用需要 `idempotencyKey` |
| 聊天 | `chat.history`, `chat.send`, `chat.abort`, `chat.inject`  | WebChat 使用这些               |
| 会话 | `sessions.list`, `sessions.patch`, `sessions.delete`      | 会话管理                       |
| 节点 | `node.list`, `node.invoke`, `node.pair.*`                 | Gateway 的 WS + 节点操作        |
| 事件 | `tick`, `presence`, `agent`, `chat`, `health`, `shutdown` | 服务器推送                      |

权威列表位于 `src/gateway/server.ts`（`METHODS`、`EVENTS`）。

<div id="where-the-schemas-live">
  ## Schema 文件所在位置
</div>

* 源码：`src/gateway/protocol/schema.ts`
* 运行时验证器（AJV）：`src/gateway/protocol/index.ts`
* 服务器握手与方法分发：`src/gateway/server.ts`
* 节点客户端：`src/gateway/client.ts`
* 生成的 JSON Schema：`dist/protocol.schema.json`
* 生成的 Swift 模型：`apps/macos/Sources/OpenClawProtocol/GatewayModels.swift`

<div id="current-pipeline">
  ## 当前流程
</div>

* `pnpm protocol:gen`
  * 将 JSON Schema（draft‑07）写入 `dist/protocol.schema.json`
* `pnpm protocol:gen:swift`
  * 生成 Swift Gateway 模型
* `pnpm protocol:check`
  * 运行两个生成器并验证输出已被提交

<div id="how-the-schemas-are-used-at-runtime">
  ## 这些 schema 在运行时如何使用
</div>

* **服务器端**：每个入站帧都会用 AJV 进行校验。握手阶段只接受其参数符合 `ConnectParams` 的 `connect` 请求。
* **客户端**：JS 客户端在使用事件和响应帧之前会先对它们进行校验。
* **方法接口**：Gateway 会在 `hello-ok` 中公开支持的 `methods` 和 `events`。

<div id="example-frames">
  ## 帧示例
</div>

Connect（首个报文）：

```json
{
  "type": "req",
  "id": "c1",
  "method": "connect",
  "params": {
    "minProtocol": 2,
    "maxProtocol": 2,
    "client": {
      "id": "openclaw-macos",
      "displayName": "macos",
      "version": "1.0.0",
      "platform": "macos 15.1",
      "mode": "ui",
      "instanceId": "A1B2"
    }
  }
}
```

Hello-ok 响应：

```json
{
  "type": "res",
  "id": "c1",
  "ok": true,
  "payload": {
    "type": "hello-ok",
    "protocol": 2,
    "server": { "version": "dev", "connId": "ws-1" },
    "features": { "methods": ["health"], "events": ["tick"] },
    "snapshot": { "presence": [], "health": {}, "stateVersion": { "presence": 0, "health": 0 }, "uptimeMs": 0 },
    "policy": { "maxPayload": 1048576, "maxBufferedBytes": 1048576, "tickIntervalMs": 30000 }
  }
}
```

请求与响应：

```json
{ "type": "req", "id": "r1", "method": "health" }
```

```json
{ "type": "res", "id": "r1", "ok": true, "payload": { "ok": true } }
```

事件：

```json
{ "type": "event", "event": "tick", "payload": { "ts": 1730000000 }, "seq": 12 }
```

<div id="minimal-client-nodejs">
  ## 最小客户端（Node.js）
</div>

最小可用流程：连接 + 健康检查。

```ts
import { WebSocket } from "ws";

const ws = new WebSocket("ws://127.0.0.1:18789");

ws.on("open", () => {
  ws.send(JSON.stringify({
    type: "req",
    id: "c1",
    method: "connect",
    params: {
      minProtocol: 3,
      maxProtocol: 3,
      client: {
        id: "cli",
        displayName: "example",
        version: "dev",
        platform: "node",
        mode: "cli"
      }
    }
  }));
});

ws.on("message", (data) => {
  const msg = JSON.parse(String(data));
  if (msg.type === "res" && msg.id === "c1" && msg.ok) {
    ws.send(JSON.stringify({ type: "req", id: "h1", method: "health" }));
  }
  if (msg.type === "res" && msg.id === "h1") {
    console.log("health:", msg.payload);
    ws.close();
  }
});
```

<div id="worked-example-add-a-method-endtoend">
  ## 实践示例：端到端新增一个方法
</div>

示例：新增一个 `system.echo` 请求，返回 `{ ok: true, text }`。

1. **Schema（权威来源）**

在 `src/gateway/protocol/schema.ts` 中新增：

```ts
export const SystemEchoParamsSchema = Type.Object(
  { text: NonEmptyString },
  { additionalProperties: false },
);

export const SystemEchoResultSchema = Type.Object(
  { ok: Type.Boolean(), text: NonEmptyString },
  { additionalProperties: false },
);
```

将二者都添加到 `ProtocolSchemas`，并导出相应的类型：

```ts
  SystemEchoParams: SystemEchoParamsSchema,
  SystemEchoResult: SystemEchoResultSchema,
```

```ts
export type SystemEchoParams = Static<typeof SystemEchoParamsSchema>;
export type SystemEchoResult = Static<typeof SystemEchoResultSchema>;
```

2. **验证**

在 `src/gateway/protocol/index.ts` 中导出一个 AJV 验证器：

```ts
export const validateSystemEchoParams =
  ajv.compile<SystemEchoParams>(SystemEchoParamsSchema);
```

3. **服务器端行为**

在 `src/gateway/server-methods/system.ts` 中添加一个处理函数：

```ts
export const systemHandlers: GatewayRequestHandlers = {
  "system.echo": ({ params, respond }) => {
    const text = String(params.text ?? "");
    respond(true, { ok: true, text });
  },
};
```

在 `src/gateway/server-methods.ts` 中注册它（已合并 `systemHandlers`），
然后在 `src/gateway/server.ts` 中将 `"system.echo"` 添加到 `METHODS` 中。

4. **重新生成**

```bash
pnpm protocol:check
```

5. **测试与文档**

在 `src/gateway/server.*.test.ts` 中添加一个服务器测试用例，并在文档中注明该方法。

<div id="swift-codegen-behavior">
  ## Swift 代码生成行为
</div>

Swift 代码生成器会生成：

* 带有 `req`、`res`、`event` 和 `unknown` 成员的 `GatewayFrame` 枚举
* 强类型的负载结构体和枚举
* `ErrorCode` 值和 `GATEWAY_PROTOCOL_VERSION`

未知的帧类型会被保留为原始负载，以实现前向兼容性。

<div id="versioning-compatibility">
  ## 版本管理与兼容性
</div>

* `PROTOCOL_VERSION` 定义在 `src/gateway/protocol/schema.ts` 中。
* 客户端会发送 `minProtocol` 和 `maxProtocol`；服务器在版本不匹配时会拒绝请求。
* Swift 模型会保留未知的帧类型，以避免导致旧版客户端失效。

<div id="schema-patterns-and-conventions">
  ## Schema 模式和约定
</div>

* 大多数对象使用 `additionalProperties: false` 来实现严格的载荷约束。
* `NonEmptyString` 是 ID 以及方法/事件名称的默认类型。
* 顶层的 `GatewayFrame` 在 `type` 上使用 **判别字段（discriminator）**。
* 具有副作用的方法通常要求在参数中提供 `idempotencyKey`
  （示例：`send`、`poll`、`agent`、`chat.send`）。

<div id="live-schema-json">
  ## 实时 Schema JSON
</div>

生成的 JSON Schema 位于代码仓库中的 `dist/protocol.schema.json`。已发布的原始文件通常可在以下地址获取：

* https://raw.githubusercontent.com/openclaw/openclaw/main/dist/protocol.schema.json

<div id="when-you-change-schemas">
  ## 当你修改 Schema 时
</div>

1. 更新 TypeBox Schema。
2. 运行 `pnpm protocol:check`。
3. 提交重新生成的 Schema 和 Swift 模型。