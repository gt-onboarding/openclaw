---
title: Peekaboo
summary: "macOS UI 自動化のための PeekabooBridge 連携"
read_when:
  - OpenClaw.app 内で PeekabooBridge をホストする場合
  - Swift Package Manager 経由で Peekaboo を連携する場合
  - PeekabooBridge のプロトコルやパスを変更する場合
---

<div id="peekaboo-bridge-macos-ui-automation">
  # Peekaboo Bridge (macOS UI 自動化)
</div>

OpenClaw は、ローカル環境で権限対応の UI 自動化ブローカーとして **PeekabooBridge** をホストできます。これにより、`peekaboo` CLI が macOS アプリの TCC 権限を再利用しながら UI 自動化を実行できるようになります。

<div id="what-this-is-and-isnt">
  ## これは何か（そして何ではないか）
</div>

- **Host**: OpenClaw.app は PeekabooBridge のホストとして動作できます。
- **Client**: `peekaboo` CLI を使用します（専用の `openclaw ui ...` UI 画面はありません）。
- **UI**: 画面オーバーレイは Peekaboo.app 側に残り、OpenClaw はシンプルなブローカー・ホストとして動作します。

<div id="enable-the-bridge">
  ## ブリッジを有効化する
</div>

macOS アプリで次を行います:

- Settings → **Enable Peekaboo Bridge**

有効にすると、OpenClaw はローカルの UNIX ソケットサーバーを起動します。無効にするとホストが停止され、`peekaboo` は利用可能なほかのホストにフォールバックします。

<div id="client-discovery-order">
  ## クライアント検出順序
</div>

Peekaboo クライアントは通常、次の順序でホストへの接続を試みます：

1. Peekaboo.app（フル UX）
2. Claude.app（インストールされている場合）
3. OpenClaw.app（軽量ブローカー）

どのホストがアクティブで、どのソケットパスが使用されているかを確認するには、
`peekaboo bridge status --verbose` を実行します。次のようにして上書きできます：

```bash
export PEEKABOO_BRIDGE_SOCKET=/path/to/bridge.sock
```


<div id="security-permissions">
  ## セキュリティと権限
</div>

- ブリッジは **呼び出し元のコード署名** を検証し、TeamID の許可リスト
  （Peekaboo ホストの TeamID と OpenClaw アプリの TeamID）を適用します。
- リクエストは約 10 秒でタイムアウトします。
- 必要な権限が不足している場合、ブリッジは System Settings を起動するのではなく、
  分かりやすいエラーメッセージを返します。

<div id="snapshot-behavior-automation">
  ## スナップショットの動作（自動化）
</div>

スナップショットはメモリ上に保存され、短時間のうちに自動的に破棄されます。
より長期間保持する必要がある場合は、クライアント側からスナップショットを取り直してください。

<div id="troubleshooting">
  ## トラブルシューティング
</div>

- `peekaboo` が「bridge client is not authorized」と報告する場合は、クライアントが正しく署名されていることを確認するか、**debug** モード時に限り、`PEEKABOO_ALLOW_UNSIGNED_SOCKET_CLIENTS=1` を指定してホストを実行してください。
- ホストが見つからない場合は、ホストアプリ（Peekaboo.app または OpenClaw.app）のいずれかを開き、必要な権限が付与されていることを確認してください。