---
title: OpenAI HTTP API
summary: "Gateway から OpenAI 互換の /v1/chat/completions HTTP エンドポイントを公開する"
read_when:
  - OpenAI Chat Completions に対応したツールを統合するとき
---

<div id="openai-chat-completions-http">
  # OpenAI Chat Completions (HTTP)
</div>

OpenClaw の Gateway は、簡易的な OpenAI 互換 Chat Completions エンドポイントを提供できます。

このエンドポイントは **デフォルトでは無効** です。まず設定で有効化してください。

- `POST /v1/chat/completions`
- Gateway と同じポート（WS + HTTP 多重化）: `http://<gateway-host>:<port>/v1/chat/completions`

内部的には、このエンドポイントへのリクエストは通常の Gateway エージェント実行として処理されます（`openclaw agent` と同じコードパス）。そのため、ルーティング／権限／設定は Gateway 側の構成と同一になります。

<div id="authentication">
  ## 認証
</div>

Gateway の認証設定を使用します。以下のように Bearer トークンを送信します:

- `Authorization: Bearer <token>`

注意事項:

- `gateway.auth.mode="token"` の場合は、`gateway.auth.token`（または `OPENCLAW_GATEWAY_TOKEN`）を使用します。
- `gateway.auth.mode="password"` の場合は、`gateway.auth.password`（または `OPENCLAW_GATEWAY_PASSWORD`）を使用します。

<div id="choosing-an-agent">
  ## エージェントの選択
</div>

カスタムヘッダーは不要です。エージェント ID は OpenAI の `model` フィールドに指定します:

- `model: "openclaw:<agentId>"`（例: `"openclaw:main"`、`"openclaw:beta"`）
- `model: "agent:<agentId>"`（エイリアス）

または、特定の OpenClaw エージェントをヘッダーで指定します:

- `x-openclaw-agent-id: <agentId>`（デフォルト: `main`）

高度な制御:

- `x-openclaw-session-key: <sessionKey>` を使用して、セッションのルーティングを完全に制御できます。

<div id="enabling-the-endpoint">
  ## エンドポイントを有効にする
</div>

`gateway.http.endpoints.chatCompletions.enabled` を `true` に設定します:

```json5
{
  gateway: {
    http: {
      endpoints: {
        chatCompletions: { enabled: true }
      }
    }
  }
}
```


<div id="disabling-the-endpoint">
  ## エンドポイントを無効化する
</div>

`gateway.http.endpoints.chatCompletions.enabled` を `false` に設定します。

```json5
{
  gateway: {
    http: {
      endpoints: {
        chatCompletions: { enabled: false }
      }
    }
  }
}
```


<div id="session-behavior">
  ## セッションの動作
</div>

デフォルトでは、このエンドポイントは **リクエストごとにステートレス** です（呼び出しごとに新しいセッションキーが生成されます）。

リクエストに OpenAI の `user` 文字列が含まれている場合、Gateway はそれをもとに安定したセッションキーを導出するため、複数回の呼び出し間でエージェントのセッションを共有できます。

<div id="streaming-sse">
  ## ストリーミング（SSE）
</div>

Server-Sent Events（SSE）を受信するには、`stream: true` を設定してください：

- `Content-Type: text/event-stream`
- 各イベント行は `data: <json>` 形式です
- ストリームは `data: [DONE]` で終了します

<div id="examples">
  ## 例
</div>

非ストリーミング:

```bash
curl -sS http://127.0.0.1:18789/v1/chat/completions \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "messages": [{"role":"user","content":"hi"}]
  }'
```

ストリーミング:

```bash
curl -N http://127.0.0.1:18789/v1/chat/completions \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "stream": true,
    "messages": [{"role":"user","content":"hi"}]
  }'
```
