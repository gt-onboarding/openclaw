---
title: Typebox
summary: "Gateway プロトコルの唯一のソース・オブ・トゥルースとしての TypeBox スキーマ"
read_when:
  - プロトコルスキーマやコード生成を更新するとき
---

<div id="typebox-as-protocol-source-of-truth">
  # プロトコルのソース・オブ・トゥルースとしての TypeBox
</div>

最終更新日: 2026-01-10

TypeBox は TypeScript ファーストのスキーマライブラリです。これを使用して **Gateway
WebSocket プロトコル**（ハンドシェイク、リクエスト/レスポンス、サーバーイベント）を定義しています。これらのスキーマが
**ランタイム検証**、**JSON Schema へのエクスポート**、および macOS アプリ向けの **Swift コード生成** の基盤になります。
信頼できる定義源は 1 つだけで、それ以外はすべて生成物です。

プロトコルのより高レベルな文脈が知りたい場合は、
[Gateway アーキテクチャ](/ja/concepts/architecture) から読み始めてください。

<div id="mental-model-30-seconds">
  ## メンタルモデル（30秒）
</div>

すべての Gateway WS メッセージは、次の 3 種類のいずれかのフレームです：

* **Request**: `{ type: "req", id, method, params }`
* **Response**: `{ type: "res", id, ok, payload | error }`
* **Event**: `{ type: "event", event, payload, seq?, stateVersion? }`

最初のフレームは、必ず `connect` リクエストである必要があります。その後、クライアントは
メソッド（例: `health`, `send`, `chat.send`）を呼び出し、イベント（例:
`presence`, `tick`, `agent`）を購読できます。

接続フロー（最小構成）：

```
Client                    Gateway
  |---- req:connect -------->|
  |<---- res:hello-ok --------|
  |<---- event:tick ----------|
  |---- req:health ---------->|
  |<---- res:health ----------|
```

一般的なメソッドとイベント:

| Category  | Examples                                                  | Notes                          |
| --------- | --------------------------------------------------------- | ------------------------------ |
| Core      | `connect`, `health`, `status`                             | `connect` は常に先頭である必要がある        |
| Messaging | `send`, `poll`, `agent`, `agent.wait`                     | 副作用を伴うものは `idempotencyKey` が必要 |
| Chat      | `chat.history`, `chat.send`, `chat.abort`, `chat.inject`  | WebChat でこれらが使用される             |
| Sessions  | `sessions.list`, `sessions.patch`, `sessions.delete`      | セッション管理                        |
| Nodes     | `node.list`, `node.invoke`, `node.pair.*`                 | Gateway の WS ＋ ノード操作           |
| Events    | `tick`, `presence`, `agent`, `chat`, `health`, `shutdown` | サーバープッシュ                       |

公式な一覧は `src/gateway/server.ts`（`METHODS`、`EVENTS`）にある。

<div id="where-the-schemas-live">
  ## スキーマの配置場所
</div>

* ソースコード: `src/gateway/protocol/schema.ts`
* ランタイムバリデータ (AJV): `src/gateway/protocol/index.ts`
* サーバーのハンドシェイク + メソッドディスパッチ: `src/gateway/server.ts`
* ノードクライアント: `src/gateway/client.ts`
* 生成された JSON Schema: `dist/protocol.schema.json`
* 生成された Swift モデル: `apps/macos/Sources/OpenClawProtocol/GatewayModels.swift`

<div id="current-pipeline">
  ## 現在のパイプライン
</div>

* `pnpm protocol:gen`
  * JSON Schema（draft‑07）を `dist/protocol.schema.json` に書き出す
* `pnpm protocol:gen:swift`
  * Gateway 用の Swift モデルを生成する
* `pnpm protocol:check`
  * 両方のジェネレーターを実行し、出力がコミットされていることを検証する

<div id="how-the-schemas-are-used-at-runtime">
  ## 実行時におけるスキーマの利用方法
</div>

* **サーバー側**: すべての受信フレームは AJV で検証されます。ハンドシェイクでは、
  パラメータが `ConnectParams` に一致する `connect` リクエストのみを受け付けます。
* **クライアント側**: JS クライアントは、イベントおよびレスポンスフレームを利用する前に検証します。
* **メソッドのインターフェース**: Gateway は、`hello-ok` においてサポートされている `methods` と
  `events` を公開します。

<div id="example-frames">
  ## フレーム例
</div>

Connect（最初のメッセージ）:

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

Hello-ok 応答:

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

リクエストとレスポンス：

```json
{ "type": "req", "id": "r1", "method": "health" }
```

```json
{ "type": "res", "id": "r1", "ok": true, "payload": { "ok": true } }
```

イベント：

```json
{ "type": "event", "event": "tick", "payload": { "ts": 1730000000 }, "seq": 12 }
```

<div id="minimal-client-nodejs">
  ## 最小構成のクライアント（Node.js）
</div>

最小限かつ実用的なフロー: 接続 + ヘルスチェック。

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
  ## 作業例：メソッドをエンドツーエンドで追加する
</div>

例：`{ ok: true, text }` を返す新しい `system.echo` リクエストを追加する。

1. **スキーマ（単一のソース・オブ・トゥルース）**

`src/gateway/protocol/schema.ts` に追加します:

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

両方を `ProtocolSchemas` に追加し、型定義をエクスポートします：

```ts
  SystemEchoParams: SystemEchoParamsSchema,
  SystemEchoResult: SystemEchoResultSchema,
```

```ts
export type SystemEchoParams = Static<typeof SystemEchoParamsSchema>;
export type SystemEchoResult = Static<typeof SystemEchoResultSchema>;
```

2. **検証**

`src/gateway/protocol/index.ts` で、AJV のバリデーターを export します：

```ts
export const validateSystemEchoParams =
  ajv.compile<SystemEchoParams>(SystemEchoParamsSchema);
```

3. **サーバーの動作**

`src/gateway/server-methods/system.ts` にハンドラを追加します:

```ts
export const systemHandlers: GatewayRequestHandlers = {
  "system.echo": ({ params, respond }) => {
    const text = String(params.text ?? "");
    respond(true, { ok: true, text });
  },
};
```

`src/gateway/server-methods.ts`（このファイルはすでに `systemHandlers` をマージ済みです）に登録し、
その後 `src/gateway/server.ts` の `METHODS` に `"system.echo"` を追加します。

4. **再生成**

```bash
pnpm protocol:check
```

5. **テストとドキュメント**

`src/gateway/server.*.test.ts` にサーバー用テストを追加し、そのメソッドをドキュメントに明記してください。

<div id="swift-codegen-behavior">
  ## Swift コード生成の挙動
</div>

Swift のコードジェネレーターは次のものを出力します:

* `req`、`res`、`event`、`unknown` ケースを持つ `GatewayFrame` enum
* 厳密に型付けされたペイロード用の struct / enum
* `ErrorCode` 値と `GATEWAY_PROTOCOL_VERSION`

未知のフレームタイプは、前方互換性のために生のペイロードとして保持されます。

<div id="versioning-compatibility">
  ## バージョニングと互換性
</div>

* `PROTOCOL_VERSION` は `src/gateway/protocol/schema.ts` に定義されています。
* クライアントは `minProtocol` と `maxProtocol` を送信し、不一致がある場合サーバーは拒否します。
* Swift のモデルは、古いクライアントとの互換性が損なわれないように、未知のフレーム型も保持したままにします。

<div id="schema-patterns-and-conventions">
  ## スキーマのパターンと規約
</div>

* ほとんどのオブジェクトは、ペイロードを厳密に制約するために `additionalProperties: false` を使用します。
* `NonEmptyString` は ID やメソッド／イベント名のデフォルト型です。
* トップレベルの `GatewayFrame` は、`type` に対する **判別用フィールド（discriminator）** を使用します。
* 副作用を持つメソッドは通常、パラメータ（params）に `idempotencyKey` が必要です
  （例: `send`, `poll`, `agent`, `chat.send`）。

<div id="live-schema-json">
  ## ライブな JSON Schema
</div>

生成された JSON Schema はリポジトリ内の `dist/protocol.schema.json` にあります。
公開されている raw 形式のファイルは通常、次の場所で参照できます:

* https://raw.githubusercontent.com/openclaw/openclaw/main/dist/protocol.schema.json

<div id="when-you-change-schemas">
  ## スキーマを変更する場合
</div>

1. TypeBox のスキーマを更新します。
2. `pnpm protocol:check` を実行します。
3. 再生成されたスキーマと Swift モデルをコミットします。