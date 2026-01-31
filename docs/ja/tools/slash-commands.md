---
title: スラッシュコマンド
summary: "スラッシュコマンド：テキスト形式とネイティブ形式、設定、および利用可能なコマンド"
read_when:
  - チャットコマンドの使用や設定を行うとき
  - コマンドのルーティングや権限をデバッグしているとき
---

<div id="slash-commands">
  # スラッシュコマンド
</div>

コマンドは Gateway によって処理されます。ほとんどのコマンドは、`/` で始まる**単独の**メッセージとして送信する必要があります。
ホスト専用の bash チャットコマンドは `! <cmd>` を使用します（エイリアスとして `/bash <cmd>` も利用できます）。

関連する仕組みが 2 つあります:

* **Commands**: 単独の `/...` メッセージ。
* **Directives**: `/think`, `/verbose`, `/reasoning`, `/elevated`, `/exec`, `/model`, `/queue`。
  * Directives は、モデルが参照する前にメッセージから取り除かれます。
  * 通常のチャットメッセージ（Directive のみではない場合）では、「インラインのヒント」として扱われ、セッション設定は**永続化されません**。
  * Directive のみのメッセージ（メッセージが Directives だけを含む場合）では、それらがセッションに永続化され、確認応答が返されます。
  * Directives は **認可された送信者**（チャンネルの許可リスト/ペアリングおよび `commands.useAccessGroups`）に対してのみ適用されます。
    認可されていない送信者に対しては、Directives はプレーンテキストとして扱われます。

また、いくつかの**インラインショートカット**もあります（許可リストに登録された/認可された送信者のみ）：`/help`, `/commands`, `/status`, `/whoami` (`/id`)。
これらは即座に実行され、モデルがメッセージを見る前に取り除かれ、残りのテキストは通常のフローで処理されます。

<div id="config">
  ## 設定
</div>

```json5
{
  commands: {
    native: "auto",
    nativeSkills: "auto",
    text: true,
    bash: false,
    bashForegroundMs: 2000,
    config: false,
    debug: false,
    restart: false,
    useAccessGroups: true
  }
}
```

* `commands.text`（デフォルト `true`）は、チャットメッセージ内の `/...` の解析を有効にします。
  * ネイティブコマンドのない環境（WhatsApp/WebChat/Signal/iMessage/Google Chat/MS Teams）でも、これを `false` に設定してもテキストコマンドは動作します。
* `commands.native`（デフォルト `"auto"`）はネイティブコマンドを登録します。
  * Auto: Discord/Telegram では有効、Slack では無効（スラッシュコマンドを追加するまで）。ネイティブ対応のないプロバイダーでは無視されます。
  * プロバイダーごとに上書きするには、`channels.discord.commands.native`、`channels.telegram.commands.native`、`channels.slack.commands.native` を設定します（bool または `"auto"`）。
  * `false` にすると、起動時に Discord/Telegram に以前登録されていたコマンドを削除します。Slack のコマンドは Slack アプリ側で管理され、自動では削除されません。
* `commands.nativeSkills`（デフォルト `"auto"`）は、サポートされている場合に **スキル** コマンドをネイティブ登録します。
  * Auto: Discord/Telegram では有効、Slack では無効（Slack ではスキルごとにスラッシュコマンドを作成する必要があります）。
  * プロバイダーごとに上書きするには、`channels.discord.commands.nativeSkills`、`channels.telegram.commands.nativeSkills`、`channels.slack.commands.nativeSkills` を設定します（bool または `"auto"`）。
* `commands.bash`（デフォルト `false`）は、`! <cmd>` でホストのシェルコマンドを実行できるようにします（`/bash <cmd>` はエイリアスです。`tools.elevated` 許可リストの設定が必要です）。
* `commands.bashForegroundMs`（デフォルト `2000`）は、バックグラウンドモードに切り替えるまで bash が待機する時間を制御します（`0` は即座にバックグラウンドに移行します）。
* `commands.config`（デフォルト `false`）は `/config` を有効にします（`openclaw.json` の読み書きを行います）。
* `commands.debug`（デフォルト `false`）は `/debug` を有効にします（実行時のみ有効なオーバーライド）。
* `commands.useAccessGroups`（デフォルト `true`）は、コマンドに対して許可リスト／ポリシーを強制します。

<div id="command-list">
  ## コマンド一覧
</div>

テキスト + ネイティブコマンド（有効な場合）:

* `/help`
* `/commands`
* `/skill <name> [input]`（指定した名前のスキルを実行）
* `/status`（現在の状態を表示。利用可能な場合は、現在のモデルプロバイダーの利用状況/クオータも含む）
* `/allowlist`（許可リストのエントリを一覧表示/追加/削除）
* `/approve <id> allow-once|allow-always|deny`（実行承認プロンプトを処理）
* `/context [list|detail|json]`（「コンテキスト」の内容を説明。`detail` はファイル単位 + ツール単位 + スキル単位 + システムプロンプトのサイズを表示）
* `/whoami`（自分の送信者 ID を表示。エイリアス: `/id`）
* `/subagents list|stop|log|info|send`（現在のセッションのサブエージェント実行を調査、停止、ログ確認、またはメッセージ送信）
* `/config show|get|set|unset`（設定をディスクに永続化（オーナーのみ）。`commands.config: true` が必要）
* `/debug show|set|unset|reset`（ランタイム設定のオーバーライド（オーナーのみ）。`commands.debug: true` が必要）
* `/usage off|tokens|full|cost`（レスポンスごとの利用状況フッター、またはローカルコストのサマリー）
* `/tts off|always|inbound|tagged|status|provider|limit|summary|audio`（TTS を制御。[/tts](/ja/tts) を参照）
  * Discord: ネイティブコマンドは `/voice`（Discord は `/tts` を予約）。テキストベースの `/tts` も引き続き使用可能。
* `/stop`
* `/restart`
* `/dock-telegram`（エイリアス: `/dock_telegram`）（返信先を Telegram に切り替え）
* `/dock-discord`（エイリアス: `/dock_discord`）（返信先を Discord に切り替え）
* `/dock-slack`（エイリアス: `/dock_slack`）（返信先を Slack に切り替え）
* `/activation mention|always`（グループのみ）
* `/send on|off|inherit`（オーナーのみ）
* `/reset` または `/new [model]`（モデルのヒントは任意。残りはそのまま渡される）
* `/think <off|minimal|low|medium|high|xhigh>`（モデル/プロバイダーにより動的に選択肢が変化。エイリアス: `/thinking`, `/t`）
* `/verbose on|full|off`（エイリアス: `/v`）
* `/reasoning on|off|stream`（エイリアス: `/reason`。on の場合、`Reasoning:` で始まる別メッセージを送信。`stream` は Telegram のドラフトのみ）
* `/elevated on|off|ask|full`（エイリアス: `/elev`。`full` は実行承認をスキップ）
* `/exec host=<sandbox|gateway|node> security=<deny|allowlist|full> ask=<off|on-miss|always> node=<id>`（現在値を表示するには `/exec` を送信）
* `/model <name>`（エイリアス: `/models`。または `agents.defaults.models.*.alias` で定義された `/<alias>`）
* `/queue <mode>`（`debounce:2s cap:25 drop:summarize` のようなオプションを付与可能。現在の設定を確認するには `/queue` を送信）
* `/bash <command>`（ホストのみ。`! <command>` のエイリアス。`commands.bash: true` + `tools.elevated` の許可リストが必要）

テキストのみ:

* `/compact [instructions]`（[/concepts/compaction](/ja/concepts/compaction) を参照）
* `! <command>`（ホストのみ。同時に 1 コマンドのみ実行可能。長時間実行ジョブには `!poll` + `!stop` を使用）
* `!poll`（出力/ステータスを確認。任意の `sessionId` を指定可能。`/bash poll` も有効）
* `!stop`（実行中の bash ジョブを停止。任意の `sessionId` を指定可能。`/bash stop` も有効）

Notes:

* コマンドは、コマンド本体と引数の間に省略可能な `:` を受け付けます（例: `/think: high`, `/send: on`, `/help:`）。
* `/new <model>` はモデルのエイリアス、`provider/model`、またはプロバイダー名（あいまい一致）を受け付けます。一致しない場合、そのテキストはメッセージ本文として扱われます。
* プロバイダーごとの利用状況の詳細を確認するには、`openclaw status --usage` を使用します。
* `/allowlist add|remove` には `commands.config=true` が必要で、チャンネルの `configWrites` を尊重します。
* `/usage` はレスポンスごとの利用状況フッターを制御します。`/usage cost` は OpenClaw セッションログからローカルのコストサマリーを出力します。
* `/restart` はデフォルトで無効です。有効化するには `commands.restart: true` を設定します。
* `/verbose` はデバッグと追加の可視性のためのもので、通常利用時は **オフ** にしておいてください。
* `/reasoning`（および `/verbose`）はグループ環境ではリスクがあります。意図せず内部の推論内容やツール出力を公開してしまう可能性があります。特にグループチャットではオフのままにしておくことを推奨します。
* **高速パス:** 許可リストに登録された送信者からの「コマンドのみ」のメッセージは即時に処理されます（キューとモデルをバイパス）。
* **グループメンションの制御:** 許可リストに登録された送信者からの「コマンドのみ」のメッセージはメンション要件をバイパスします。
* **インラインショートカット（許可リスト送信者のみ）:** 一部のコマンドは通常メッセージ内に埋め込まれていても動作し、モデルが残りのテキストを見る前に取り除かれます。
  * 例: `hey /status` はステータス返信をトリガーし、残りのテキストは通常フローで処理されます。
* 現在の対象: `/help`, `/commands`, `/status`, `/whoami` (`/id`)。
* 許可されていない「コマンドのみ」のメッセージは黙って無視され、インラインの `/...` トークンはプレーンテキストとして扱われます。
* **スキルコマンド:** `user-invocable` スキルはスラッシュコマンドとして公開されます。名前は `a-z0-9_`（最大 32 文字）にサニタイズされ、衝突した場合は数値のサフィックスが付きます（例: `_2`）。
  * `/skill <name> [input]` は名前でスキルを実行します（ネイティブコマンドの制限によりスキルごとのコマンドを作れない場合に有用です）。
  * デフォルトでは、スキルコマンドは通常のリクエストとしてモデルに転送されます。
  * スキルはオプションで `command-dispatch: tool` を宣言し、コマンドをツールに直接ルーティングすることができます（決定的で、モデルは介在しません）。
  * 例: `/prose`（OpenProse プラグイン）— [OpenProse](/ja/prose) を参照してください。
* **ネイティブコマンド引数:** Discord では動的オプションにオートコンプリート機能が使われます（必須引数を省略するとボタンメニューが表示されます）。Telegram と Slack では、コマンドが選択肢をサポートしていて引数を省略した場合にボタンメニューが表示されます。

<div id="usage-surfaces-what-shows-where">
  ## 表示される場所（どこに何が出るか）
</div>

* 使用量トラッキングが有効な場合、**プロバイダーの使用量／クオータ**（例: 「Claude 残り 80%」）は、現在のモデルプロバイダーについて `/status` に表示されます。
* **レスポンスごとのトークン数／コスト**は、`/usage off|tokens|full` で制御されます（通常の返信の末尾に付加されます）。
* `/model status` は使用量ではなく、**モデル／認証／エンドポイント** に関する情報です。

<div id="model-selection-model">
  ## モデル選択 (`/model`)
</div>

`/model` はディレクティブとして実装されています。

使用例:

```
/model
/model list
/model 3
/model openai/gpt-5.2
/model opus@anthropic:default
/model status
```

Notes:

* `/model` と `/model list` は、番号付きのコンパクトなピッカー（モデルファミリー + 利用可能なプロバイダー）を表示します。
* `/model <#>` は、そのピッカーから選択し、可能な場合は現在のプロバイダーを優先します。
* `/model status` は、設定済みのプロバイダーのエンドポイント（`baseUrl`）および、利用可能な場合は API モード（`api`）を含む詳細ビューを表示します。

<div id="debug-overrides">
  ## デバッグ用オーバーライド
</div>

`/debug` を使うと、**ランタイム時のみ有効な** 設定オーバーライドを行えます（メモリ上のみで、ディスクには書き込まれません）。オーナーのみが使用可能です。デフォルトでは無効で、`commands.debug: true` を指定すると有効になります。

例:

```
/debug show
/debug set messages.responsePrefix="[openclaw]"
/debug set channels.whatsapp.allowFrom=["+1555","+4477"]
/debug unset messages.responsePrefix
/debug reset
```

注意事項:

* オーバーライドは新しい設定の読み込みに対して即座に適用されますが、`openclaw.json` には **書き込まれません**。
* すべてのオーバーライドをクリアしてオンディスクの設定に戻すには、`/debug reset` を使用します。

<div id="config-updates">
  ## 設定の更新
</div>

`/config` はディスク上の設定ファイル（`openclaw.json`）に書き込みます。所有者のみ実行可能です。デフォルトでは無効であり、`commands.config: true` を指定すると有効になります。

使用例:

```
/config show
/config show messages.responsePrefix
/config get messages.responsePrefix
/config set messages.responsePrefix="[openclaw]"
/config unset messages.responsePrefix
```

注意:

* 設定は書き込み前に検証され、無効な変更は拒否されます。
* `/config` の更新は再起動しても保持されます。

<div id="surface-notes">
  ## 補足メモ
</div>

* **テキストコマンド** は通常のチャットセッションで実行されます（DM は `main` を共有し、グループはそれぞれ独自のセッションを持ちます）。
* **ネイティブコマンド** は分離されたセッションを使用します:
  * Discord: `agent:<agentId>:discord:slash:<userId>`
  * Slack: `agent:<agentId>:slack:slash:<userId>`（プレフィックスは `channels.slack.slashCommand.sessionPrefix` で設定可能）
  * Telegram: `telegram:slash:<userId>`（`CommandTargetSessionKey` を通じて該当チャットセッションを指定）
* **`/stop`** はアクティブなチャットセッションを対象として、現在の実行を中断できます。
* **Slack:** `channels.slack.slashCommand` は、単一の `/openclaw` 形式のコマンド用として引き続きサポートされています。`commands.native` を有効にする場合は、組み込みコマンドごとに 1 つずつ（`/help` と同じ名前で）Slack のスラッシュコマンドを作成する必要があります。Slack 向けのコマンド引数メニューは、エフェメラルな Block Kit ボタンとして配信されます。