---
title: プロトコル
summary: "Gateway WebSocket プロトコル: ハンドシェイク、フレーム、バージョン管理"
read_when:
  - Gateway の WS クライアントを実装または更新しているとき
  - プロトコルの不整合や接続失敗をデバッグしているとき
  - プロトコルのスキーマ／モデルを再生成しているとき
---

<div id="gateway-protocol-websocket">
  # Gateway protocol (WebSocket)
</div>

Gateway WS プロトコルは、OpenClaw における**唯一のコントロールプレーン兼ノード用トランスポート**です。すべてのクライアント（CLI、web UI、macOS アプリ、iOS/Android ノード、ヘッドレス ノード）は WebSocket 経由で接続し、ハンドシェイク時に自分の **role** と **スコープ** を宣言します。

<div id="transport">
  ## トランスポート
</div>

- WebSocket、JSON ペイロードを含むテキストフレームを使用。
- 最初のフレームは必ず `connect` リクエストとします。

<div id="handshake-connect">
  ## ハンドシェイク（接続）
</div>

Gateway → クライアント（接続前のチャレンジ）:

```json
{
  "type": "event",
  "event": "connect.challenge",
  "payload": { "nonce": "…", "ts": 1737264000000 }
}
```

クライアント → Gateway：

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

Gateway → クライアント：

```json
{
  "type": "res",
  "id": "…",
  "ok": true,
  "payload": { "type": "hello-ok", "protocol": 3, "policy": { "tickIntervalMs": 15000 } }
}
```

デバイストークンが発行されると、`hello-ok` には次の情報も含まれます：

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
  ### ノードの例
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
  ## フレーミング
</div>

- **Request**: `{type:"req", id, method, params}`  
- **Response**: `{type:"res", id, ok, payload|error}`  
- **Event**: `{type:"event", event, payload, seq?, stateVersion?}`

副作用を伴うメソッドには、**冪等性キー (idempotency keys)** が必要です（スキーマを参照してください）。

<div id="roles-scopes">
  ## ロールとスコープ
</div>

<div id="roles">
  ### ロール
</div>

- `operator` = コントロールプレーンのクライアント（CLI/UI/自動化）。
- `node` = 機能を提供するホスト（camera/screen/canvas/system.run）。

<div id="scopes-operator">
  ### スコープ（operator）
</div>

代表的なスコープ:

- `operator.read`
- `operator.write`
- `operator.admin`
- `operator.approvals`
- `operator.pairing`

<div id="capscommandspermissions-node">
  ### caps / commands / permissions（ノード）
</div>

ノードは接続時に、自身の機能に関するクレームを宣言します:

- `caps`: 機能の高レベルなカテゴリ。
- `commands`: 呼び出しに使用するコマンドの許可リスト。
- `permissions`: `screen.record` や `camera.capture` などの細かなオン/オフ権限。

Gateway はこれらを**クレーム**として扱い、サーバー側の許可リストに基づいて制御を強制します。

<div id="presence">
  ## プレゼンス
</div>

- `system-presence` はデバイスの識別子をキーとしたエントリを返します。
- プレゼンスエントリには `deviceId`、`roles`、`scopes` が含まれており、デバイスが **operator** と **ノード** の両方として
  接続している場合でも、UI でデバイスごとに 1 行だけ表示できるようになっています。

<div id="node-helper-methods">
  ### ノードのヘルパーメソッド
</div>

- ノードは自動許可チェックのために、現在のスキル実行ファイルの一覧を取得する目的で `skills.bins` を呼び出すことができます。

<div id="exec-approvals">
  ## Exec 承認
</div>

- Exec リクエストに承認が必要な場合、Gateway は `exec.approval.requested` をブロードキャストします。
- オペレータークライアントは `exec.approval.resolve` を呼び出して承認を確定します（`operator.approvals` スコープが必要）。

<div id="versioning">
  ## バージョニング
</div>

- `PROTOCOL_VERSION` は `src/gateway/protocol/schema.ts` に定義されています。
- クライアントは `minProtocol` と `maxProtocol` を送信し、サーバーは不一致の場合にリクエストを拒否します。
- スキーマとモデルは TypeBox の定義から生成されます:
  - `pnpm protocol:gen`
  - `pnpm protocol:gen:swift`
  - `pnpm protocol:check`

<div id="auth">
  ## 認証
</div>

- `OPENCLAW_GATEWAY_TOKEN`（または `--token`）が設定されている場合、`connect.params.auth.token`
  がそれと一致していないとソケットは閉じられます。
- ペアリング後、Gateway は接続ロールとスコープに対応する **デバイストークン** を発行します。
  これは `hello-ok.auth.deviceToken` として返され、クライアントは今後の接続のために永続化しておく必要があります。
- デバイストークンは `device.token.rotate` および
  `device.token.revoke`（`operator.pairing` スコープが必要）でローテーション／失効させることができます。

<div id="device-identity-pairing">
  ## デバイス ID とペアリング
</div>

- ノードは、キーペアのフィンガープリントから導出される安定したデバイス ID（`device.id`）を含める必要があります。
- Gateway は、デバイスごと・ロールごとにトークンを発行します。
- ローカルの自動承認が有効になっていない限り、新しいデバイス ID にはペアリング承認が必要です。
- **ローカル** 接続にはループバックと、Gateway ホスト自身の tailnet アドレスが含まれます
  （同一ホストの tailnet バインドであれば引き続き自動承認が可能です）。
- すべての WS クライアントは `connect` 時に `device` ID を含めなければなりません（オペレーター + ノード）。
  Control UI は、`gateway.controlUi.allowInsecureAuth` が有効な場合に **のみ**
  （もしくはブレークグラス用途で `gateway.controlUi.dangerouslyDisableDeviceAuth` が有効な場合）
  `device` ID を省略できます。
- 非ローカル接続は、サーバーから提供される `connect.challenge` ノンスに署名する必要があります。

<div id="tls-pinning">
  ## TLS + ピンニング
</div>

- WS 接続で TLS を利用できます。
- クライアントはオプションとして Gateway 証明書のフィンガープリントをピン留めできます（`gateway.tls`
  設定および `gateway.remote.tlsFingerprint`、もしくは CLI の `--tls-fingerprint` を参照）。

<div id="scope">
  ## スコープ
</div>

このプロトコルは、**Gateway の完全な API**（status、channels、models、chat、
agent、sessions、nodes、approvals など）を提供します。具体的な API サーフェスは、
`src/gateway/protocol/schema.ts` 内の TypeBox スキーマで定義されています。