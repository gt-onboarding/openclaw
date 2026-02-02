---
title: OpenResponses HTTP API
summary: "Gateway から OpenResponses 互換の /v1/responses HTTP エンドポイントを公開する"
read_when:
  - OpenResponses API に対応したクライアントと統合するとき
  - アイテム単位の入力、クライアントによるツール呼び出し、または SSE イベントを利用したいとき
---

<div id="openresponses-api-http">
  # OpenResponses API (HTTP)
</div>

OpenClaw の Gateway では、OpenResponses 互換の `POST /v1/responses` エンドポイントを提供できます。

このエンドポイントは**デフォルトでは無効**です。まずは設定で有効化してください。

- `POST /v1/responses`
- Gateway と同じポート（WS + HTTP 多重化）: `http://<gateway-host>:<port>/v1/responses`

内部的には、リクエストは通常の Gateway エージェント実行（`openclaw agent` と同じコードパス）として処理されるため、ルーティング／権限／設定は Gateway の構成と一致します。

<div id="authentication">
  ## 認証
</div>

Gateway の認証設定を使用します。Bearer トークンを送信してください:

- `Authorization: Bearer <token>`

補足:

- `gateway.auth.mode="token"` の場合、`gateway.auth.token`（または `OPENCLAW_GATEWAY_TOKEN`）を使用します。
- `gateway.auth.mode="password"` の場合、`gateway.auth.password`（または `OPENCLAW_GATEWAY_PASSWORD`）を使用します。

<div id="choosing-an-agent">
  ## エージェントの選択
</div>

カスタムヘッダーは不要です。エージェント ID を OpenResponses の `model` フィールドに指定します:

- `model: "openclaw:<agentId>"`（例: `"openclaw:main"`, `"openclaw:beta"`）
- `model: "agent:<agentId>"`（エイリアス）

または、ヘッダーを使って特定の OpenClaw エージェントを指定します:

- `x-openclaw-agent-id: <agentId>`（デフォルト: `main`）

高度な設定:

- `x-openclaw-session-key: <sessionKey>` を使用して、セッションルーティングを完全に制御します。

<div id="enabling-the-endpoint">
  ## エンドポイントを有効にする
</div>

`gateway.http.endpoints.responses.enabled` を `true` に設定します：

```json5
{
  gateway: {
    http: {
      endpoints: {
        responses: { enabled: true }
      }
    }
  }
}
```


<div id="disabling-the-endpoint">
  ## エンドポイントを無効化する
</div>

`gateway.http.endpoints.responses.enabled` を `false` に設定します。

```json5
{
  gateway: {
    http: {
      endpoints: {
        responses: { enabled: false }
      }
    }
  }
}
```


<div id="session-behavior">
  ## セッションの動作
</div>

デフォルトでは、このエンドポイントは**リクエストごとにステートレス**です（呼び出しのたびに新しいセッションキーが生成されます）。

リクエストに OpenResponses の `user` 文字列が含まれている場合、Gateway はその値から安定したセッションキーを生成し、同じエージェントセッションを複数回の呼び出しで共有できるようにします。

<div id="request-shape-supported">
  ## リクエスト構造（サポート済み）
</div>

リクエストは、アイテムベースの入力を用いる OpenResponses API に従います。現在サポートされている項目は次のとおりです:

- `input`: 文字列、またはアイテムオブジェクトの配列。
- `instructions`: system プロンプトにマージされる指示文。
- `tools`: クライアント側ツールの定義（function tools）。
- `tool_choice`: クライアントツールのフィルタまたは必須指定。
- `stream`: SSE（Server-Sent Events）ストリーミングを有効化。
- `max_output_tokens`: ベストエフォートの出力トークン上限（プロバイダー依存）。
- `user`: 安定したセッションルーティング。

受け付けるが、**現在は無視される**項目:

- `max_tool_calls`
- `reasoning`
- `metadata`
- `store`
- `previous_response_id`
- `truncation`

<div id="items-input">
  ## 項目（入力）
</div>

<div id="message">
  ### `message`
</div>

ロール: `system`、`developer`、`user`、`assistant`。

- `system` と `developer` はシステムプロンプトに追加されます。
- 直近の `user` または `function_call_output` アイテムが「現在のメッセージ」になります。
- それ以前の `user` / `assistant` メッセージは、コンテキスト用の履歴として含まれます。

<div id="function_call_output-turn-based-tools">
  ### `function_call_output` (ターンベースツール)
</div>

ツールの結果をモデルに返します：

```json
{
  "type": "function_call_output",
  "call_id": "call_123",
  "output": "{\"temperature\": \"72F\"}"
}
```


<div id="reasoning-and-item_reference">
  ### `reasoning` と `item_reference`
</div>

スキーマとの互換性のために受け付けますが、プロンプトを構築する際には使用されません。

<div id="tools-client-side-function-tools">
  ## ツール（クライアント側の function ツール）
</div>

`tools: [{ type: "function", function: { name, description?, parameters? } }]` の形でツールを指定します。

エージェントがツールを呼び出すと判断した場合、レスポンスには `function_call` の出力アイテムが含まれます。
その後、ターンを継続するために `function_call_output` を使って後続リクエストを送信します。

<div id="images-input_image">
  ## 画像 (`input_image`)
</div>

base64 または URL のソースをサポートします:

```json
{
  "type": "input_image",
  "source": { "type": "url", "url": "https://example.com/image.png" }
}
```

許可されている MIME タイプ（現時点）：`image/jpeg`、`image/png`、`image/gif`、`image/webp`。
最大サイズ（現時点）：10MB。


<div id="files-input_file">
  ## ファイル (`input_file`)
</div>

Base64 または URL ソースをサポートします：

```json
{
  "type": "input_file",
  "source": {
    "type": "base64",
    "media_type": "text/plain",
    "data": "SGVsbG8gV29ybGQh",
    "filename": "hello.txt"
  }
}
```

許可されている MIME タイプ（現在）: `text/plain`, `text/markdown`, `text/html`, `text/csv`,
`application/json`, `application/pdf`.

最大サイズ（現在）: 5MB。

現在の動作:

* ファイル内容はデコードされて **system prompt** に追加され、ユーザーメッセージには追加されません。
  そのため内容は一時的なものとなり（セッション履歴には永続化されません）。
* PDF はテキストとしてパースされます。テキストがほとんど見つからない場合、最初の数ページが画像にラスタライズされ、
  モデルに渡されます。

PDF の解析には Node 向けの `pdfjs-dist` legacy ビルド（worker なし）を使用します。モダンな PDF.js ビルドは
ブラウザの worker/DOM グローバルを前提としているため、Gateway では使用しません。

URL フェッチのデフォルト設定:

* `files.allowUrl`: `true`
* `images.allowUrl`: `true`
* リクエストには保護がかけられています（DNS 解決、プライベート IP ブロッキング、リダイレクト上限、タイムアウト）。


<div id="file-image-limits-config">
  ## ファイルと画像の上限（設定）
</div>

既定値は `gateway.http.endpoints.responses` 配下で調整できます：

```json5
{
  gateway: {
    http: {
      endpoints: {
        responses: {
          enabled: true,
          maxBodyBytes: 20000000,
          files: {
            allowUrl: true,
            allowedMimes: ["text/plain", "text/markdown", "text/html", "text/csv", "application/json", "application/pdf"],
            maxBytes: 5242880,
            maxChars: 200000,
            maxRedirects: 3,
            timeoutMs: 10000,
            pdf: {
              maxPages: 4,
              maxPixels: 4000000,
              minTextChars: 200
            }
          },
          images: {
            allowUrl: true,
            allowedMimes: ["image/jpeg", "image/png", "image/gif", "image/webp"],
            maxBytes: 10485760,
            maxRedirects: 3,
            timeoutMs: 10000
          }
        }
      }
    }
  }
}
```

省略時のデフォルト値:

* `maxBodyBytes`: 20MB
* `files.maxBytes`: 5MB
* `files.maxChars`: 200k
* `files.maxRedirects`: 3
* `files.timeoutMs`: 10s
* `files.pdf.maxPages`: 4
* `files.pdf.maxPixels`: 4,000,000
* `files.pdf.minTextChars`: 200
* `images.maxBytes`: 10MB
* `images.maxRedirects`: 3
* `images.timeoutMs`: 10s


<div id="streaming-sse">
  ## ストリーミング（SSE）
</div>

`stream: true` を設定すると、Server-Sent Events（SSE）としてストリームを受信できます:

- `Content-Type: text/event-stream`
- 各イベント行は `event: <type>` および `data: <json>` です
- ストリームは `data: [DONE]` で終了します

現在送出されるイベント種別は次のとおりです:

- `response.created`
- `response.in_progress`
- `response.output_item.added`
- `response.content_part.added`
- `response.output_text.delta`
- `response.output_text.done`
- `response.content_part.done`
- `response.output_item.done`
- `response.completed`
- `response.failed`（エラー時）

<div id="usage">
  ## 利用状況
</div>

`usage` は、基盤となるプロバイダーがトークン数を報告した場合に値が設定されます。

<div id="errors">
  ## エラー
</div>

エラーは次のような JSON オブジェクト形式になります:

```json
{ "error": { "message": "...", "type": "invalid_request_error" } }
```

よくあるケース:

* `401` 認証情報がない／無効
* `400` 不正なリクエストボディ
* `405` 許可されていない HTTP メソッド


<div id="examples">
  ## 例
</div>

ストリーミングしない場合:

```bash
curl -sS http://127.0.0.1:18789/v1/responses \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "input": "hi"
  }'
```

ストリーミング：

```bash
curl -N http://127.0.0.1:18789/v1/responses \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "stream": true,
    "input": "hi"
  }'
```
