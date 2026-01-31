---
title: TUI
summary: "ターミナル UI (TUI): どのマシンからでも Gateway に接続する"
read_when:
  - TUI の初心者向けウォークスルーが必要なとき
  - TUI の機能・コマンド・ショートカットの完全な一覧が必要なとき
---

<div id="tui-terminal-ui">
  # TUI（ターミナルUI）
</div>

<div id="quick-start">
  ## クイックスタート
</div>

1. Gateway を起動する。

```bash
openclaw gateway
```

2. TUI を開きます。

```bash
openclaw tui
```

3. メッセージを入力して Enter キーを押します。

リモート Gateway:

```bash
openclaw tui --url ws://<host>:<port> --token <gateway-token>
```

Gateway がパスワード認証を使用している場合は `--password` を指定してください。

<div id="what-you-see">
  ## 画面構成
</div>

* ヘッダー: 接続URL、現在のエージェント、現在のセッション。
* チャットログ: ユーザーメッセージ、アシスタントの返信、システム通知、ツールカード。
* ステータス行: 接続/実行状態（connecting、running、streaming、idle、error）。
* フッター: 接続状態 + エージェント + セッション + モデル + think/verbose/reasoning の各フラグ + トークン数 + deliver。
* 入力欄: 補完機能付きテキストエディタ。

<div id="mental-model-agents-sessions">
  ## メンタルモデル: エージェント + セッション
</div>

* エージェントは一意のスラッグです（例: `main`, `research`）。Gateway がその一覧を提供します。
* セッションは現在のエージェントに属します。
* セッションキーは `agent:<agentId>:<sessionKey>` の形式で保存されます。
  * `/session main` と入力すると、TUI はそれを `agent:<currentAgent>:main` に展開します。
  * `/session agent:other:main` と入力すると、そのエージェントのそのセッションに明示的に切り替わります。
* セッションのスコープ:
  * `per-sender`（デフォルト）: 各エージェントは複数のセッションを持ちます。
  * `global`: TUI は常に `global` セッションを使用します（ピッカーが空になる場合があります）。
* 現在のエージェントとセッションは常にフッターに表示されます。

<div id="sending-delivery">
  ## 送信と配信
</div>

* メッセージは Gateway に送信されますが、プロバイダーへの配信は既定では無効になっています。
* 配信を有効にするには:
  * `/deliver on`
  * または Settings パネル
  * または `openclaw tui --deliver` で起動する

<div id="pickers-overlays">
  ## ピッカーとオーバーレイ
</div>

* モデルピッカー: 利用可能なモデルを一覧表示し、セッション用のオーバーライドを設定します。
* エージェントピッカー: 別のエージェントを選択します。
* セッションピッカー: 現在のエージェントのセッションのみを表示します。
* 設定: deliver、ツール出力の展開、思考表示の有無を切り替えます。

<div id="keyboard-shortcuts">
  ## キーボードショートカット
</div>

* Enter: メッセージを送信
* Esc: 実行中の処理を中断
* Ctrl+C: 入力をクリア（2回押すと終了）
* Ctrl+D: 終了
* Ctrl+L: モデル選択
* Ctrl+G: エージェント選択
* Ctrl+P: セッション選択
* Ctrl+O: ツール出力の展開を切り替え
* Ctrl+T: 思考表示の切り替え（履歴を再読み込み）

<div id="slash-commands">
  ## スラッシュコマンド
</div>

コア:

* `/help`
* `/status`
* `/agent <id>` (または `/agents`)
* `/session <key>` (または `/sessions`)
* `/model <provider/model>` (または `/models`)

セッション制御:

* `/think <off|minimal|low|medium|high>`
* `/verbose <on|full|off>`
* `/reasoning <on|off|stream>`
* `/usage <off|tokens|full>`
* `/elevated <on|off|ask|full>` (別名: `/elev`)
* `/activation <mention|always>`
* `/deliver <on|off>`

セッションのライフサイクル:

* `/new` または `/reset` (セッションをリセット)
* `/abort` (進行中の実行を中断)
* `/settings`
* `/exit`

その他の Gateway のスラッシュコマンド (例: `/context`) は Gateway に転送され、システム出力として表示されます。詳細は [Slash commands](/ja/tools/slash-commands) を参照してください。

<div id="local-shell-commands">
  ## ローカルシェルコマンド
</div>

* 行の先頭に `!` を付けると、TUI ホスト上でローカルシェルコマンドを実行します。
* TUI はセッションごとに一度だけローカル実行を許可するかどうかを確認します。拒否すると、そのセッションでは `!` は無効のままになります。
* コマンドは、TUI の作業ディレクトリで新しい非対話型シェル内で実行されます（永続的な `cd` や環境変数の変更は行われません）。
* 単独の `!` は通常のメッセージとして送信され、先頭にスペースがある場合はローカル実行をトリガーしません。

<div id="tool-output">
  ## ツール出力
</div>

* ツール呼び出しは、引数と結果を含むカードとして表示されます。
* Ctrl+O でカードの折りたたみ／展開を切り替えます。
* ツール実行中は、同じカード内に途中経過がストリーミング表示されます。

<div id="history-streaming">
  ## 履歴とストリーミング
</div>

* 接続時に、TUI は最新の履歴を読み込みます（デフォルトでは直近 200 件のメッセージ）。
* ストリーミング中のレスポンスは、確定されるまでその場で更新され続けます。
* TUI は、よりリッチなツールカードのためにエージェントのツールイベントも監視します。

<div id="connection-details">
  ## 接続の詳細
</div>

* TUI は Gateway に `mode: "tui"` として登録されます。
* 再接続が行われた場合はシステムメッセージが表示され、イベントのギャップはログに出力されます。

<div id="options">
  ## オプション
</div>

* `--url <url>`: Gateway の WS URL（デフォルトは config の値、または `ws://127.0.0.1:<port>`）
* `--token <token>`: Gateway トークン（必要な場合）
* `--password <password>`: Gateway パスワード（必要な場合）
* `--session <key>`: セッションキー（デフォルト: `main`、scope が global の場合は `global`）
* `--deliver`: アシスタントの返信をプロバイダーへ配信する（デフォルト無効）
* `--thinking <level>`: 送信時の thinking レベルを上書き
* `--timeout-ms <ms>`: エージェントのタイムアウト（ミリ秒単位、デフォルトは `agents.defaults.timeoutSeconds`）

<div id="troubleshooting">
  ## トラブルシューティング
</div>

メッセージを送信しても出力がない場合:

* TUI で `/status` を実行し、Gateway が接続されており、アイドル状態かビジー状態かを確認します。
* Gateway のログを確認します: `openclaw logs --follow`。
* エージェントが実行可能か確認します: `openclaw status` と `openclaw models status`。
* チャットチャネルでメッセージが受信されるはずの場合は、配信を有効にします（`/deliver on` または `--deliver`）。
* `--history-limit <n>`: 読み込む履歴エントリの数（既定値: 200）

<div id="troubleshooting">
  ## トラブルシューティング
</div>

* `disconnected`: Gateway が動作していることと、`--url/--token/--password` が正しいことを確認してください。
* ピッカーにエージェントが表示されない場合: `openclaw agents list` とルーティング設定を確認してください。
* セッションピッカーが空の場合: グローバルスコープにいるか、まだセッションが作成されていない可能性があります。