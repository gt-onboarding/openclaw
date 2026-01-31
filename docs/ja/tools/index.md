---
title: ツール
summary: "OpenClaw のエージェント用ツール群（ブラウザ、キャンバス、ノード、メッセージ、cron）で、レガシーな `openclaw-*` スキルを置き換えるものです"
read_when:
  - エージェントのツールを追加または変更するとき
  - '`openclaw-*` スキルを廃止または変更するとき'
---

<div id="tools-openclaw">
  # ツール (OpenClaw)
</div>

OpenClaw は、ブラウザ、キャンバス、ノード、cron 用の**ファーストクラスのエージェントツール**を提供します。
これらは従来の `openclaw-*` スキルを置き換えるものであり、ツールは型定義されていてシェル経由の呼び出しは行わず、
エージェントはそれらを直接利用することを前提としています。

<div id="disabling-tools">
  ## ツールの無効化
</div>

`openclaw.json` の `tools.allow` / `tools.deny` を使って、ツールをグローバルに許可／拒否できます
（`deny` が優先されます）。これにより、禁止されたツールがモデルプロバイダーに送信されるのを防げます。

```json5
{
  tools: { deny: ["browser"] }
}
```

Notes:

* 照合は大文字・小文字を区別しません。
* `*` ワイルドカードがサポートされています（`"*"` はすべてのツールを意味します）。
* `tools.allow` が未知または未ロードのプラグインツール名のみを参照している場合、OpenClaw は警告をログに記録し、コアツールが引き続き利用可能であるように許可リストを無視します。

<div id="tool-profiles-base-allowlist">
  ## ツールプロファイル（基本許可リスト）
</div>

`tools.profile` は、`tools.allow` / `tools.deny` より前段として **基本ツール許可リスト** を設定します。
エージェント単位での上書き設定: `agents.list[].tools.profile`。

プロファイル:

* `minimal`: `session_status` のみ
* `coding`: `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`
* `messaging`: `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status`
* `full`: 制限なし（未設定と同じ）

例（デフォルトではメッセージング専用だが、Slack と Discord 向けツールも許可する）:

```json5
{
  tools: {
    profile: "messaging",
    allow: ["slack", "discord"]
  }
}
```

例（コーディング用プロファイルだが、どのコンテキストでも exec/process を禁止）:

```json5
{
  tools: {
    profile: "coding",
    deny: ["group:runtime"]
  }
}
```

例（グローバルなコーディングプロファイル、メッセージング専用のサポートエージェント）:

```json5
{
  tools: { profile: "coding" },
  agents: {
    list: [
      {
        id: "support",
        tools: { profile: "messaging", allow: ["slack"] }
      }
    ]
  }
}
```

<div id="provider-specific-tool-policy">
  ## プロバイダー固有のツールポリシー
</div>

`tools.byProvider` を使用すると、グローバルデフォルトを変更せずに、
特定のプロバイダー（または単一の `provider/model`）向けにツールを**さらに制限**できます。
エージェントごとのオーバーライド: `agents.list[].tools.byProvider`。

これは基本のツールプロファイルの**後**かつ allow/deny リストの**前**に適用されるため、
ツールセットを狭める方向にしか作用しません。
プロバイダーキーには `provider`（例: `google-antigravity`）または
`provider/model`（例: `openai/gpt-5.2`）のいずれかを指定できます。

例（グローバルのコーディングプロファイルは維持しつつ、Google Antigravity 用の利用可能なツールを最小限に抑える）:

```json5
{
  tools: {
    profile: "coding",
    byProvider: {
      "google-antigravity": { profile: "minimal" }
    }
  }
}
```

例（不安定なエンドポイント向けのプロバイダー／モデル別許可リスト）:

```json5
{
  tools: {
    allow: ["group:fs", "group:runtime", "sessions_list"],
    byProvider: {
      "openai/gpt-5.2": { allow: ["group:fs", "sessions_list"] }
    }
  }
}
```

例（単一プロバイダー向けのエージェント固有オーバーライド）：

```json5
{
  agents: {
    list: [
      {
        id: "support",
        tools: {
          byProvider: {
            "google-antigravity": { allow: ["message", "sessions_list"] }
          }
        }
      }
    ]
  }
}
```

<div id="tool-groups-shorthands">
  ## ツールグループ（省略表記）
</div>

ツールポリシー（グローバル、エージェント、サンドボックス）は、複数のツールに展開される `group:*` エントリをサポートします。
これらは `tools.allow` / `tools.deny` で指定します。

利用可能なグループは次のとおりです:

* `group:runtime`: `exec`, `bash`, `process`
* `group:fs`: `read`, `write`, `edit`, `apply_patch`
* `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
* `group:memory`: `memory_search`, `memory_get`
* `group:web`: `web_search`, `web_fetch`
* `group:ui`: `browser`, `canvas`
* `group:automation`: `cron`, `gateway`
* `group:messaging`: `message`
* `group:nodes`: `nodes`
* `group:openclaw`: すべての組み込み OpenClaw ツール（プロバイダープラグインは除く）

例（ファイル系ツールと `browser` のみを許可する場合）:

```json5
{
  tools: {
    allow: ["group:fs", "browser"]
  }
}
```

<div id="plugins-tools">
  ## プラグイン + ツール
</div>

プラグインは、コアセットに加えて **追加のツール**（および CLI コマンド）を登録できます。
インストールと設定については [Plugins](/ja/plugin) を、ツールの使用ガイダンスがプロンプトにどのように挿入されるかについては [Skills](/ja/tools/skills) を参照してください。一部のプラグインは、ツールとあわせて独自のスキル（例: voice-call プラグイン）を提供します。

オプションのプラグインツール:

* [Lobster](/ja/tools/lobster): 再開可能な承認機能を備えた型付きワークフロー実行環境（Gateway ホスト上に Lobster CLI が必要です）。
* [LLM Task](/ja/tools/llm-task): 構造化ワークフロー出力のための JSON 専用 LLM ステップ（スキーマ検証はオプション）。

<div id="tool-inventory">
  ## ツール一覧
</div>

<div id="apply_patch">
  ### `apply_patch`
</div>

1つ以上のファイルに対して構造化パッチを適用します。複数ハンクからなる編集に使用します。
試験的機能: `tools.exec.applyPatch.enabled` を有効化すると利用できます（OpenAI モデルのみ）。

<div id="exec">
  ### `exec`
</div>

ワークスペース内でシェルコマンドを実行します。

主なパラメータ:

* `command` (必須)
* `yieldMs` (指定時間経過後に自動でバックグラウンド実行へ移行、デフォルト 10000)
* `background` (即時バックグラウンド実行)
* `timeout` (秒数; 超過した場合はプロセスを強制終了、デフォルト 1800)
* `elevated` (bool 値; 昇格モードが有効かつ許可されている場合にホスト上で実行; エージェントがサンドボックス化されている場合にのみ挙動が変化)
* `host` (`sandbox | gateway | node`)
* `security` (`deny | allowlist | full`)
* `ask` (`off | on-miss | always`)
* `node` (`host=node` のときのノード ID/名前)
* 実際の TTY が必要な場合は、`pty: true` を設定してください。

注意事項:

* バックグラウンド実行された場合、`status: "running"` と `sessionId` を返します。
* バックグラウンドセッションのポーリング/ログ取得/書き込み/kill/クリアには `process` を使用します。
* `process` が禁止されている場合、`exec` は同期的に実行され、`yieldMs` / `background` は無視されます。
* `elevated` は `tools.elevated` と、任意の `agents.list[].tools.elevated` オーバーライド (両方が許可している必要あり) によって制御され、`host=gateway` + `security=full` のエイリアスです。
* `elevated` はエージェントがサンドボックス化されている場合にのみ挙動を変えます (それ以外では何もしません)。
* `host=node` は、macOS コンパニオンアプリまたはヘッドレスなノードホスト (`openclaw node run`) をターゲットにできます。
* Gateway/ノードの承認と許可リストについては、[Exec approvals](/ja/tools/exec-approvals) を参照してください。

<div id="process">
  ### `process`
</div>

バックグラウンドの実行セッションを管理します。

主なアクション:

* `list`, `poll`, `log`, `write`, `kill`, `clear`, `remove`

補足:

* `poll` は、完了時に新しい出力と終了ステータスを返します。
* `log` は行単位の `offset` / `limit` をサポートします（最後の N 行を取得する場合は `offset` を省略します）。
* `process` はエージェント単位でスコープされており、他のエージェントのセッションは表示されません。

<div id="web_search">
  ### `web_search`
</div>

Brave Search API を使用してウェブを検索します。

主要なパラメータ:

* `query` (必須)
* `count` (1〜10。デフォルトは `tools.web.search.maxResults` の値)

注記:

* Brave API キーが必要です (推奨: `openclaw configure --section web` を実行するか、`BRAVE_API_KEY` を設定)。
* `tools.web.search.enabled` で有効にします。
* レスポンスはキャッシュされます (デフォルトでは 15 分間)。
* セットアップ方法は [Web tools](/ja/tools/web) を参照してください。

<div id="web_fetch">
  ### `web_fetch`
</div>

URL から可読なコンテンツを取得して抽出します (HTML → markdown/text)。

主要なパラメータ:

* `url` (必須)
* `extractMode` (`markdown` | `text`)
* `maxChars` (長いページを切り詰め)

注意事項:

* `tools.web.fetch.enabled` で有効化します。
* レスポンスはキャッシュされます (デフォルト 15 分)。
* JS を多用するサイトでは、`browser` ツールを優先してください。
* セットアップについては [Web tools](/ja/tools/web) を参照してください。
* オプションのアンチボット用フォールバックについては [Firecrawl](/ja/tools/firecrawl) を参照してください。

<div id="browser">
  ### `browser`
</div>

OpenClaw が管理する専用ブラウザを制御します。

主なアクション:

* `status`, `start`, `stop`, `tabs`, `open`, `focus`, `close`
* `snapshot` (aria/ai)
* `screenshot` (画像ブロック + `MEDIA:<path>` を返す)
* `act` (UI アクション: click/type/press/hover/drag/select/fill/resize/wait/evaluate)
* `navigate`, `console`, `pdf`, `upload`, `dialog`

プロファイル管理:

* `profiles` — すべてのブラウザプロファイルとそのステータスを一覧表示
* `create-profile` — 自動割り当てされたポート (または `cdpUrl`) 付きで新しいプロファイルを作成
* `delete-profile` — ブラウザを停止し、ユーザーデータを削除して、設定から削除 (ローカルのみ)
* `reset-profile` — プロファイルのポート上で孤立したプロセスを強制終了 (ローカルのみ)

共通パラメータ:

* `profile` (省略可能; 省略時は `browser.defaultProfile`)
* `target` (`sandbox` | `host` | `node`)
* `node` (省略可能; 特定のノード ID/名前を指定)

注意事項:

* `browser.enabled=true` が必要です (デフォルトは `true`。無効化するには `false` を設定)。
* すべてのアクションは、マルチインスタンス対応のため省略可能な `profile` パラメータを受け取ります。
* `profile` が省略された場合は `browser.defaultProfile` を使用します (デフォルトは &quot;chrome&quot;)。
* プロファイル名: 英小文字英数字 + ハイフンのみ (最大 64 文字)。
* ポート範囲: 18800-18899 (プロファイル最大約 100 個)。
* リモートプロファイルは接続 (attach) のみ可能です (start/stop/reset は不可)。
* ブラウザ機能を持つノードが接続されている場合、`target` を固定しない限り、そのノードへ自動ルーティングされる場合があります。
* `snapshot` は Playwright がインストールされている場合、デフォルトで `ai` を使用します。アクセシビリティツリーには `aria` を使用します。
* `snapshot` は role-snapshot オプション (`interactive`, `compact`, `depth`, `selector`) もサポートし、`e12` のような参照を返します。
* `act` には `snapshot` からの `ref` が必要です (AI snapshot からの数値 `12`、または role snapshot からの `e12`)。まれに CSS セレクタが必要な場合のみ `evaluate` を使用してください。
* デフォルトでは `act` → `wait` の使用は避けてください。信頼できる UI 状態に待機できない例外的なケースでのみ使用してください。
* `upload` はオプションで `ref` を受け取り、準備完了後に自動クリックを行うことができます。
* `upload` は `<input type="file">` を直接設定するために、`inputRef` (aria ref) または `element` (CSS セレクタ) もサポートします。

<div id="canvas">
  ### `canvas`
</div>

ノードの Canvas を操作します（present、eval、snapshot、A2UI）。

主なアクション:

* `present`, `hide`, `navigate`, `eval`
* `snapshot`（画像ブロック + `MEDIA:<path>` を返す）
* `a2ui_push`, `a2ui_reset`

注意:

* 内部的には Gateway の `node.invoke` を使用します。
* `node` が指定されていない場合、ツールはデフォルト（接続中のノードが 1 つだけの場合はそのノード、またはローカルの mac ノード）を選択します。
* A2UI は v0.8 のみ対応です（`createSurface` なし）。CLI は v0.9 JSONL を行ごとのエラーとして拒否します。
* 簡易動作確認: `openclaw nodes canvas a2ui push --node <id> --text "Hello from A2UI"`。

<div id="nodes">
  ### `nodes`
</div>

ペアリング済みノードを検出して対象として指定し、通知を送信し、カメラ／画面をキャプチャします。

主なアクション:

* `status`, `describe`
* `pending`, `approve`, `reject` (ペアリング)
* `notify` (macOS `system.notify`)
* `run` (macOS `system.run`)
* `camera_snap`, `camera_clip`, `screen_record`
* `location_get`

補足:

* カメラ／画面コマンドを実行するには、ノードアプリが前面で動作している必要があります。
* 画像は画像ブロック + `MEDIA:<path>` を返します。
* 動画は `FILE:<path>` (mp4) を返します。
* 位置情報は JSON ペイロード (lat/lon/accuracy/timestamp) を返します。
* `run` のパラメータ: `command` は argv 配列。任意で `cwd`, `env` (`KEY=VAL`), `commandTimeoutMs`, `invokeTimeoutMs`, `needsScreenRecording` を指定できます。

例 (`run`):

```json
{
  "action": "run",
  "node": "office-mac",
  "command": ["echo", "Hello"],
  "env": ["FOO=bar"],
  "commandTimeoutMs": 12000,
  "invokeTimeoutMs": 45000,
  "needsScreenRecording": false
}
```

<div id="image">
  ### `image`
</div>

設定済みの画像モデルを使って画像を解析します。

コアパラメータ:

* `image`（必須。パスまたはURL）
* `prompt`（任意。デフォルトは &quot;Describe the image.&quot;）
* `model`（任意の上書き指定）
* `maxBytesMb`（任意のサイズ上限）

注意事項:

* `agents.defaults.imageModel` が（プライマリまたはフォールバックとして）設定されている場合、またはデフォルトモデルと設定済みの認証情報から暗黙的な画像モデルを推測できる場合（ベストエフォートのペアリング）のみ利用可能です。
* 画像モデルを直接使用します（メインのチャットモデルとは独立）。

<div id="message">
  ### `message`
</div>

Discord/Google Chat/Slack/Telegram/WhatsApp/Signal/iMessage/MS Teams 間でメッセージおよびチャンネルアクションを送信します。

主要アクション:

* `send`（テキスト + オプションのメディア。MS Teams では Adaptive Cards 用の `card` もサポート）
* `poll`（WhatsApp/Discord/MS Teams の投票）
* `react` / `reactions` / `read` / `edit` / `delete`
* `pin` / `unpin` / `list-pins`
* `permissions`
* `thread-create` / `thread-list` / `thread-reply`
* `search`
* `sticker`
* `member-info` / `role-info`
* `emoji-list` / `emoji-upload` / `sticker-upload`
* `role-add` / `role-remove`
* `channel-info` / `channel-list`
* `voice-status`
* `event-list` / `event-create`
* `timeout` / `kick` / `ban`

注意事項:

* `send` は WhatsApp を Gateway 経由でルーティングし、その他のチャンネルは直接送信します。
* `poll` は WhatsApp および MS Teams に対しては Gateway を使用し、Discord の投票は直接送信します。
* `message` ツール呼び出しがアクティブなチャットセッションに結び付けられている場合、コンテキストをまたいだ情報リークを防ぐため、送信先はそのセッションのターゲットに制約されます。

<div id="cron">
  ### `cron`
</div>

Gateway の cron ジョブとウェイクアップを管理します。

主なアクション:

* `status`, `list`
* `add`, `update`, `remove`, `run`, `runs`
* `wake` (システムイベントをキューに投入し、必要に応じて即時のハートビートを実行)

注意:

* `add` は完全な cron ジョブオブジェクトを受け取ります (スキーマは `cron.add` RPC と同一)。
* `update` は `{ id, patch }` を使用します。

<div id="gateway">
  ### `gateway`
</div>

実行中の Gateway プロセスを再起動、または更新を適用します（インプレース）。

主なアクション:

* `restart`（権限を確認したうえで `SIGUSR1` を送信し、プロセス内で再起動します。`openclaw gateway` をインプレースで再起動）
* `config.get` / `config.schema`
* `config.apply`（設定の検証 + 書き込み + 再起動 + 起動）
* `config.patch`（部分的な更新をマージ + 再起動 + 起動）
* `update.run`（更新を実行 + 再起動 + 起動）

補足:

* 進行中の応答を中断しないように、`delayMs`（デフォルトは 2000ms）を使用してください。
* `restart` はデフォルトで無効です。`commands.restart: true` を指定して有効化します。

<div id="sessions_list-sessions_history-sessions_send-sessions_spawn-session_status">
  ### `sessions_list` / `sessions_history` / `sessions_send` / `sessions_spawn` / `session_status`
</div>

セッションを一覧表示し、トランスクリプト履歴を確認したり、別のセッションに送信したりします。

コアパラメータ:

* `sessions_list`: `kinds?`, `limit?`, `activeMinutes?`, `messageLimit?` (0 = 制限なし)
* `sessions_history`: `sessionKey` (または `sessionId`), `limit?`, `includeTools?`
* `sessions_send`: `sessionKey` (または `sessionId`), `message`, `timeoutSeconds?` (0 = fire-and-forget)
* `sessions_spawn`: `task`, `label?`, `agentId?`, `model?`, `runTimeoutSeconds?`, `cleanup?`
* `session_status`: `sessionKey?` (デフォルトは現在のもの。`sessionId` を受け付ける), `model?` (`default` はオーバーライドをクリア)

補足:

* `main` は正準のダイレクトチャット用キーです。global/unknown は非表示です。
* `messageLimit > 0` の場合、セッションごとに直近 N 件のメッセージを取得します (ツールメッセージはフィルタされます)。
* `sessions_send` は `timeoutSeconds > 0` の場合、最終的な完了まで待機します。
* 配信/announce は完了後に実行され、best-effort で行われます。`status: "ok"` は、announce が配信されたことではなく、エージェントの実行が完了したことを示します。
* `sessions_spawn` はサブエージェントの実行を開始し、リクエスト元のチャットに対して announce の返信を投稿します。
* `sessions_spawn` はノンブロッキングで、即座に `status: "accepted"` を返します。
* `sessions_send` は返信のピンポン往復を実行します (終了するには `REPLY_SKIP` を返信。最大ターン数は `session.agentToAgent.maxPingPongTurns` で制御、0～5)。
* ピンポン完了後、対象エージェントは **announce ステップ** を実行します。アナウンスを抑止するには `ANNOUNCE_SKIP` を返信します。

<div id="agents_list">
  ### `agents_list`
</div>

現在のセッションが `sessions_spawn` でターゲットにできるエージェント ID の一覧。

注記:

* 結果はエージェントごとの許可リスト（`agents.list[].subagents.allowAgents`）によって制限されます。
* `["*"]` が設定されている場合、このツールは設定済みのすべてのエージェントを含め、その際 `allowAny: true` を設定します。

<div id="parameters-common">
  ## パラメータ（共通）
</div>

Gateway バックエンドのツール（`canvas`、`nodes`、`cron`）:

* `gatewayUrl`（デフォルト: `ws://127.0.0.1:18789`）
* `gatewayToken`（認証が有効な場合に使用）
* `timeoutMs`

ブラウザツール:

* `profile`（任意。省略時は `browser.defaultProfile`）
* `target`（`sandbox` | `host` | `node`）
* `node`（任意。特定のノード ID/名前に固定する場合に指定）

<div id="recommended-agent-flows">
  ## 推奨エージェントフロー
</div>

ブラウザ自動化フロー:

1. `browser` → `status` / `start`
2. `snapshot` (ai または aria)
3. `act` (click/type/press)
4. 画面での確認が必要な場合は `screenshot`

キャンバス描画フロー:

1. `canvas` → `present`
2. `a2ui_push` (任意)
3. `snapshot`

ノードのターゲット指定:

1. `nodes` → `status`
2. 選択したノードに対して `describe`
3. `notify` / `run` / `camera_snap` / `screen_record`

<div id="safety">
  ## 安全性
</div>

* 直接の `system.run` は避けてください。`nodes` → `run` は、ユーザーから明示的な同意を得た場合にのみ使用してください。
* カメラ／画面キャプチャについては、ユーザーの同意を尊重してください。
* メディア関連のコマンドを呼び出す前に、権限を確認するために `status/describe` を使用してください。

<div id="how-tools-are-presented-to-the-agent">
  ## ツールがエージェントにどのように提示されるか
</div>

ツールは、2つの並行するチャネルを通じてエージェントに提供されます。

1. **システムプロンプトテキスト**: 人間が読めるリストとガイダンス。
2. **ツールスキーマ**: モデル API に送信される構造化された関数定義。

つまり、エージェントは「どのツールが存在するか」と「どのように呼び出すか」の両方を認識します。ツールがシステムプロンプトテキストまたはスキーマのどちらにも含まれていない場合、モデルはそのツールを呼び出すことができません。