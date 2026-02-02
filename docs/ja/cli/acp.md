---
title: ACP
summary: "IDE 連携用の ACP ブリッジを実行する"
read_when:
  - ACP ベースの IDE 連携を設定するとき
  - ACP セッションの Gateway へのルーティングをデバッグするとき
---

<div id="acp">
  # acp
</div>

OpenClaw Gateway と通信する ACP (Agent Client Protocol) ブリッジを実行します。

このコマンドは IDE 向けに標準入出力 (stdio) 経由で ACP を話し、プロンプトを WebSocket 経由で Gateway に転送します。ACP セッションを Gateway のセッションキーに対応付けた状態で保持します。

<div id="usage">
  ## 使い方
</div>

```bash
openclaw acp

# Remote Gateway
openclaw acp --url wss://gateway-host:18789 --token <token>

# Attach to an existing session key
openclaw acp --session agent:main:main

# Attach by label (must already exist)
openclaw acp --session-label "support inbox"

# 最初のプロンプトの前にセッションキーをリセットする
openclaw acp --session agent:main:main --reset-session
```

<div id="acp-client-debug">
  ## ACP クライアント（デバッグ）
</div>

組み込みの ACP クライアントを使えば、IDE を使わずにブリッジの動作を簡易チェックできます。
これにより ACP ブリッジが起動し、対話的にプロンプトを入力できます。

```bash
openclaw acp client

# Point the spawned bridge at a remote Gateway
openclaw acp client --server-args --url wss://gateway-host:18789 --token <token>

# サーバーコマンドを上書きする（デフォルト: openclaw）
openclaw acp client --server "node" --server-args openclaw.mjs acp --url ws://127.0.0.1:19001
```

<div id="usage">
  ## 使い方
</div>

IDE（または他のクライアント）が Agent Client Protocol をサポートしており、
そのクライアントから OpenClaw Gateway のセッションを制御したい場合に ACP を使用します。

1. Gateway が起動していることを確認します（ローカルまたはリモート）。
2. Gateway のターゲットを設定します（設定ファイルまたはフラグ）。
3. IDE から stdio 経由で `openclaw acp` を実行するように指定します。

永続化される設定例:

```bash
openclaw config set gateway.remote.url wss://gateway-host:18789
openclaw config set gateway.remote.token <token>
```

設定ファイルを書き込まずに直接実行する例:

```bash
openclaw acp --url wss://gateway-host:18789 --token <token>
```

<div id="selecting-agents">
  ## エージェントの選択
</div>

ACP はエージェントを直接選択しません。Gateway のセッションキーに基づいてルーティングします。

特定のエージェントを指定するには、エージェントスコープのセッションキーを使用します。

```bash
openclaw acp --session agent:main:main
openclaw acp --session agent:design:main
openclaw acp --session agent:qa:bug-123
```

各 ACP セッションは 1 つの Gateway セッションキーに対応します。1 つのエージェントは複数のセッションを持つことができます。ACP は、キーまたはラベルを上書きしない限り、分離された `acp:<uuid>` セッションをデフォルトとして使用します。

<div id="zed-editor-setup">
  ## Zed エディタのセットアップ
</div>

`~/.config/zed/settings.json` にカスタム ACP エージェントを追加します（または Zed の Settings UI を使用します）：

```json
{
  "agent_servers": {
    "OpenClaw ACP": {
      "type": "custom",
      "command": "openclaw",
      "args": ["acp"],
      "env": {}
    }
  }
}
```

特定の Gateway またはエージェントを指定するには:

```json
{
  "agent_servers": {
    "OpenClaw ACP": {
      "type": "custom",
      "command": "openclaw",
      "args": [
        "acp",
        "--url", "wss://gateway-host:18789",
        "--token", "<token>",
        "--session", "agent:design:main"
      ],
      "env": {}
    }
  }
}
```

Zed でエージェントパネルを開き、「OpenClaw ACP」を選択してスレッドを開始します。

<div id="session-mapping">
  ## セッションのマッピング
</div>

デフォルトでは、ACP のセッションには、`acp:` プレフィックス付きの分離された Gateway セッションキーが割り当てられます。
既知のセッションを再利用するには、セッションキーまたはラベルを指定します:

* `--session <key>`: 特定の Gateway セッションキーを使用します。
* `--session-label <label>`: 既存のセッションをラベルで特定します。
* `--reset-session`: そのキー用に新しいセッション ID を払い出します（同じキーで新しい対話履歴）。

ACP クライアントがメタデータをサポートしている場合、セッション単位で上書きできます:

```json
{
  "_meta": {
    "sessionKey": "agent:main:main",
    "sessionLabel": "サポート受信トレイ",
    "resetSession": true
  }
}
```

セッションキーについての詳細は [/concepts/session](/ja/concepts/session) を参照してください。

<div id="options">
  ## オプション
</div>

* `--url <url>`: Gateway の WebSocket URL（設定済みの場合は gateway.remote.url がデフォルト）。
* `--token <token>`: Gateway 用認証トークン。
* `--password <password>`: Gateway 用認証パスワード。
* `--session <key>`: デフォルトのセッションキー。
* `--session-label <label>`: 解決に使用するデフォルトのセッションラベル。
* `--require-existing`: セッションキー／ラベルが存在しない場合はエラーにする。
* `--reset-session`: 初回使用前にセッションキーをリセットする。
* `--no-prefix-cwd`: プロンプトの前にカレントディレクトリを自動付与しない。
* `--verbose, -v`: 詳細なログを stderr に出力する。

<div id="acp-client-options">
  ### `acp client` オプション
</div>

* `--cwd <dir>`: ACP セッションの作業ディレクトリ。
* `--server <command>`: ACP サーバーコマンド（デフォルト: `openclaw`）。
* `--server-args <args...>`: ACP サーバーに渡す追加の引数。
* `--server-verbose`: ACP サーバーで詳細ログを有効にする。
* `--verbose, -v`: クライアントの詳細ログを有効にする。