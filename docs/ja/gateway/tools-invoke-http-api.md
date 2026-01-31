---
title: Tools Invoke Http Api
summary: "Gateway の HTTP エンドポイント経由で単一のツールを直接呼び出す"
read_when:
  - 完全なエージェントのターンを実行せずにツールだけを呼び出したいとき
  - ツールポリシーの強制適用が必要な自動化フローを構築するとき
---

<div id="tools-invoke-http">
  # Tools Invoke (HTTP)
</div>

OpenClaw の Gateway は、単一のツールを直接呼び出すためのシンプルな HTTP エンドポイントを提供します。これは常に有効ですが、Gateway の認証とツールポリシーによって保護されています。

- `POST /tools/invoke`
- Gateway と同じポート (WS + HTTP 多重化): `http://<gateway-host>:<port>/tools/invoke`

デフォルトの最大ペイロードサイズは 2 MB です。

<div id="authentication">
  ## 認証
</div>

Gateway の認証設定を使用します。Bearer トークンを送信してください:

- `Authorization: Bearer <token>`

注意:

- `gateway.auth.mode="token"` の場合、`gateway.auth.token`（または `OPENCLAW_GATEWAY_TOKEN`）を使用します。
- `gateway.auth.mode="password"` の場合、`gateway.auth.password`（または `OPENCLAW_GATEWAY_PASSWORD`）を使用します。

<div id="request-body">
  ## リクエストボディ
</div>

```json
{
  "tool": "sessions_list",
  "action": "json",
  "args": {},
  "sessionKey": "main",
  "dryRun": false
}
```

Fields:

* `tool` (string, required): 呼び出すツール名。
* `action` (string, optional): ツールのスキーマが `action` をサポートしていて、かつ `args` ペイロードで省略された場合に、`args` にマッピングされます。
* `args` (object, optional): ツール固有の引数。
* `sessionKey` (string, optional): 対象となるセッションキー。省略された場合、または `"main"` の場合、Gateway は設定済みのメインセッションキーを使用します（`session.mainKey` とデフォルトのエージェント設定、またはグローバルスコープでの `global` を優先）。
* `dryRun` (boolean, optional): 将来の利用のために予約されています。現在は無視されます。


<div id="policy-routing-behavior">
  ## ポリシーとルーティングの動作
</div>

ツールの利用可否は、Gateway エージェントで使用されるものと同じポリシーチェーンを通じてフィルタされます:

- `tools.profile` / `tools.byProvider.profile`
- `tools.allow` / `tools.byProvider.allow`
- `agents.<id>.tools.allow` / `agents.<id>.tools.byProvider.allow`
- グループポリシー（セッションキーがグループまたはチャネルに対応している場合）
- サブエージェントポリシー（サブエージェントのセッションキーで呼び出す場合）

ツールがポリシーによって許可されていない場合、エンドポイントは **404** を返します。

グループポリシーがコンテキストを解決できるようにするため、次を任意で設定できます:

- `x-openclaw-message-channel: <channel>`（例: `slack`, `telegram`）
- `x-openclaw-account-id: <accountId>`（複数アカウントが存在する場合）

<div id="responses">
  ## レスポンス
</div>

- `200` → `{ ok: true, result }`
- `400` → `{ ok: false, error: { type, message } }`（不正なリクエストまたはツールエラー）
- `401` → 認証されていない
- `404` → ツールが利用できない（見つからない、または許可リストに含まれていない）
- `405` → 許可されていないメソッド

<div id="example">
  ## 使用例
</div>

```bash
curl -sS http://127.0.0.1:18789/tools/invoke \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -d '{
    "tool": "sessions_list",
    "action": "json",
    "args": {}
  }'
```
