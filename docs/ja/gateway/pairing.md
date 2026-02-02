---
title: ペアリング
summary: "iOS およびその他のリモートノード向けの Gateway 管理ノードとのペアリング（オプション B）"
read_when:
  - macOS の UI を使わずにノードペアリング承認を実装するとき
  - リモートノードを承認するための CLI フローを追加するとき
  - ノード管理機能によって Gateway プロトコルを拡張するとき
---

<div id="gateway-owned-pairing-option-b">
  # Gateway 管理のペアリング (オプション B)
</div>

Gateway 管理のペアリングでは、どのノードが参加を許可されるかについての
信頼できる情報源は **Gateway** です。UI（macOS アプリや今後のクライアント）は、
保留中のリクエストを承認または却下するだけのフロントエンドに過ぎません。

**重要:** WS ノードは、`connect` 中に **デバイスペアリング**（ロール `node`）を使用します。
`node.pair.*` は別個のペアリングストアであり、WS ハンドシェイクの可否を決めるものでは **ありません**。
明示的に `node.pair.*` を呼び出すクライアントだけが、このフローを使用します。

<div id="concepts">
  ## コンセプト
</div>

- **保留中のリクエスト**: 参加を要求しているノード。承認が必要。
- **ペアリング済みノード**: 承認され、認証トークンが発行されたノード。
- **トランスポート**: Gateway の WS エンドポイントはリクエストを転送するが、
  どのノードを参加させるかは決定しない。（レガシーの TCP ブリッジのサポートは非推奨となり、削除済み。）

<div id="how-pairing-works">
  ## ペアリングの仕組み
</div>

1. ノードが Gateway の WS に接続してペアリングを要求します。
2. Gateway は **保留中のリクエスト** を保存し、`node.pair.requested` を発行します。
3. オペレーターがそのリクエストを承認または拒否します（CLI または UI）。
4. 承認されると、Gateway は **新しいトークン** を発行します（再ペアリング時にはトークンがローテーションされます）。
5. ノードはそのトークンを使って再接続し、「ペアリング済み」となります。

保留中のリクエストは**5 分**経過すると自動的に失効します。

<div id="cli-workflow-headless-friendly">
  ## CLI ワークフロー（ヘッドレス環境に適した）
</div>

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
openclaw nodes reject <requestId>
openclaw nodes status
openclaw nodes rename --node <id|name|ip> --name "Living Room iPad"
```

`nodes status` は、ペアリング／接続されているノードと、それぞれのケイパビリティを表示します。


<div id="api-surface-gateway-protocol">
  ## API サーフェス（Gateway プロトコル）
</div>

Events:

- `node.pair.requested` — 新しい保留中リクエストが作成されたときにイベントが送出されます。
- `node.pair.resolved` — リクエストが承認／拒否／期限切れになったときにイベントが送出されます。

Methods:

- `node.pair.request` — 保留中リクエストを作成するか、既存のものを再利用します。
- `node.pair.list` — 保留中およびペア済みのノードを一覧表示します。
- `node.pair.approve` — 保留中リクエストを承認します（トークンを発行します）。
- `node.pair.reject` — 保留中リクエストを拒否します。
- `node.pair.verify` — `{ nodeId, token }` を検証します。

Notes:

- `node.pair.request` はノードごとに冪等です：繰り返し呼び出しても同じ保留中リクエストが返されます。
- 承認時には **常に** 新しいトークンが生成されます。`node.pair.request` からトークンが返されることはありません。
- リクエストには、自動承認フロー向けのヒントとして `silent: true` を含めることができます。

<div id="auto-approval-macos-app">
  ## 自動承認（macOS アプリ）
</div>

macOS アプリは、以下の条件を満たす場合にオプションとして**サイレント承認**を試行できます:

- リクエストが `silent` としてマークされている
- 同一ユーザーを使って Gateway ホストへの SSH 接続を検証できる

サイレント承認に失敗した場合は、通常の「Approve/Reject」プロンプトにフォールバックします。

<div id="storage-local-private">
  ## ストレージ（ローカル・非公開）
</div>

ペアリング状態は Gateway の状態ディレクトリ（デフォルト `~/.openclaw`）配下に保存されます:

- `~/.openclaw/nodes/paired.json`
- `~/.openclaw/nodes/pending.json`

`OPENCLAW_STATE_DIR` を変更した場合、`nodes/` フォルダもそれに合わせて移動します。

セキュリティに関する注意:

- トークンはシークレットです。`paired.json` は機微情報として扱ってください。
- トークンをローテーションすると、再承認（またはノードエントリの削除）が必要になります。

<div id="transport-behavior">
  ## トランスポートの動作
</div>

- トランスポートは**ステートレス**であり、メンバーシップ情報を保持しません。
- Gateway がオフラインであるか、ペアリングが無効になっている場合、ノードはペアリングできません。
- Gateway がリモートモードで動作している場合でも、ペアリングはリモート Gateway のストアに対して行われます。