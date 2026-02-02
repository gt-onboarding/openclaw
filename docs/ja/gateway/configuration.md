---
title: 設定
summary: "~/.openclaw/openclaw.json のすべての設定オプションとその使用例"
read_when:
  - 設定フィールドを追加または変更する場合
---

<div id="configuration">
  # 設定 🔧
</div>

OpenClaw はオプションの **JSON5** 設定ファイルを `~/.openclaw/openclaw.json` から読み込みます（コメントや末尾カンマを含められます）。

ファイルが存在しない場合、OpenClaw は比較的安全なデフォルト設定を使用します（組み込み Pi エージェント + 送信者ごとのセッション + ワークスペース `~/.openclaw/workspace`）。通常、設定ファイルが必要になるのは次のような場合です。

* ボットをトリガーできる相手を制限する（`channels.whatsapp.allowFrom`、`channels.telegram.allowFrom` など）
* グループの許可リストとメンション動作を制御する（`channels.whatsapp.groups`、`channels.telegram.groups`、`channels.discord.guilds`、`agents.list[].groupChat`）
* メッセージのプレフィックスをカスタマイズする（`messages`）
* エージェントのワークスペースを設定する（`agents.defaults.workspace` または `agents.list[].workspace`）
* 組み込みエージェントのデフォルト（`agents.defaults`）とセッション動作（`session`）を調整する
* エージェントごとのアイデンティティを設定する（`agents.list[].identity`）

> **設定は初めてですか？** 詳細な説明付きの完全なサンプルについては、[Configuration Examples](/ja/gateway/configuration-examples) ガイドを参照してください。

<div id="strict-config-validation">
  ## 厳密な設定検証
</div>

OpenClaw は、スキーマに完全に一致する設定のみを受け付けます。
未知のキー、不正な型、または無効な値がある場合、安全性のため Gateway は**起動を拒否**します。

バリデーションに失敗した場合:

* Gateway は起動しません。
* 診断用コマンドのみが実行可能です（例: `openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`, `openclaw service`, `openclaw help`）。
* `openclaw doctor` を実行して、問題点の詳細を確認します。
* `openclaw doctor --fix`（または `--yes`）を実行して、マイグレーションや修復を適用します。

`--fix` / `--yes` を明示的に指定しない限り、`openclaw doctor` が変更を書き込むことはありません。

<div id="schema-ui-hints">
  ## スキーマと UI ヒント
</div>

Gateway は、UI エディタ向けに設定の JSON Schema 表現を `config.schema` 経由で公開します。
Control UI はこのスキーマからフォームを生成し、最終手段として **Raw JSON** エディタも提供します。

チャネル用プラグインや拡張機能は、自身の設定用にスキーマと UI ヒントを登録できるため、
アプリ間でチャネル設定をハードコードされたフォームに依存せず、スキーマドリブンな形で一貫性を保てます。

ヒント（ラベル、グルーピング、機密性の高いフィールド）はスキーマと一緒に配布されるため、クライアントは
設定の知識をハードコードすることなく、より適切なフォームをレンダリングできます。

<div id="apply-restart-rpc">
  ## 適用＋再起動（RPC）
</div>

`config.apply` を使うと、1 回の操作で設定全体の検証＋書き込みと Gateway の再起動を行えます。
再起動センチネルを書き込み、Gateway が復帰した後に最後のアクティブなセッションへ ping を送ります。

警告: `config.apply` は **設定全体** を置き換えます。いくつかのキーだけを変更したい場合は、
`config.patch` か `openclaw config set` を使ってください。`~/.openclaw/openclaw.json` のバックアップを必ず取っておいてください。

パラメータ:

* `raw` (string) — 設定全体を表す JSON5 ペイロード
* `baseHash` (optional) — `config.get` から取得した設定ハッシュ（既存の設定がある場合は必須）
* `sessionKey` (optional) — 起動用 ping を送る、最後のアクティブなセッションキー
* `note` (optional) — 再起動センチネルに含めるメモ
* `restartDelayMs` (optional) — 再起動までの遅延（デフォルト 2000）

例（`gateway call` 経由）:

```bash
openclaw gateway call config.get --params '{}' # payload.hash をキャプチャ
openclaw gateway call config.apply --params '{
  "raw": "{\\n  agents: { defaults: { workspace: \\"~/.openclaw/workspace\\" } }\\n}\\n",
  "baseHash": "<hash-from-config.get>",
  "sessionKey": "agent:main:whatsapp:dm:+15555550123",
  "restartDelayMs": 1000
}'
```

<div id="partial-updates-rpc">
  ## 部分更新 (RPC)
</div>

`config.patch` を使うと、関係のないキーを上書きせずに、既存の設定に部分的な更新をマージできます。これは JSON merge patch のセマンティクスに従います:

* オブジェクトは再帰的にマージされる
* `null` はキーを削除する
* 配列は置き換えられる
  `config.apply` と同様に、バリデーションを行い、設定を書き出し、再起動センチネルを保存し、Gateway の再起動をスケジュールします（`sessionKey` が指定されている場合はオプションでウェイクアップを行います）。

パラメータ:

* `raw` (string) — 変更したいキーだけを含む JSON5 ペイロード
* `baseHash` (required) — `config.get` から取得した設定ハッシュ
* `sessionKey` (optional) — ウェイクアップ ping 用の直近のアクティブなセッションキー
* `note` (optional) — 再起動センチネルに含めるメモ
* `restartDelayMs` (optional) — 再起動までの遅延（デフォルト 2000）

例:

```bash
openclaw gateway call config.get --params '{}' # payload.hash をキャプチャ
openclaw gateway call config.patch --params '{
  "raw": "{\\n  channels: { telegram: { groups: { \\"*\\": { requireMention: false } } } }\\n}\\n",
  "baseHash": "<hash-from-config.get>",
  "sessionKey": "agent:main:whatsapp:dm:+15555550123",
  "restartDelayMs": 1000
}'
```

<div id="minimal-config-recommended-starting-point">
  ## 最小限の設定（推奨される出発点）
</div>

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } }
}
```

次のコマンドでデフォルトイメージを一度ビルドします:

```bash
scripts/sandbox-setup.sh
```

<div id="self-chat-mode-recommended-for-group-control">
  ## セルフチャットモード（グループ制御に推奨）
</div>

ボットが WhatsApp のグループで @メンションに反応せず、特定のテキストトリガーにのみ反応するようにするには:

```json5
{
  agents: {
    defaults: { workspace: "~/.openclaw/workspace" },
    list: [
      {
        id: "main",
        groupChat: { mentionPatterns: ["@openclaw", "reisponde"] }
      }
    ]
  },
  channels: {
    whatsapp: {
      // 許可リストはDMのみ。自分の番号を含めるとセルフチャットモードが有効になります。
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } }
    }
  }
}
```

<div id="config-includes-include">
  ## Config Includes (`$include`)
</div>

`$include` ディレクティブを使って、設定を複数のファイルに分割できます。これは次のような用途に有用です。

* 大規模な設定を整理する（例: クライアントごとのエージェント定義）
* 環境間で共通設定を共有する
* 機密性の高い設定を別ファイルとして分離して管理する

<div id="basic-usage">
  ### 基本的な使い方
</div>

```json5
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789 },
  
  // 単一ファイルをインクルード（キーの値を置き換え）
  agents: { "$include": "./agents.json5" },
  
  // Include multiple files (deep-merged in order)
  broadcast: { 
    "$include": [
      "./clients/mueller.json5",
      "./clients/schmidt.json5"
    ]
  }
}
```

```json5
// ~/.openclaw/agents.json5
{
  defaults: { sandbox: { mode: "all", scope: "session" } },
  list: [
    { id: "main", workspace: "~/.openclaw/workspace" }
  ]
}
```

<div id="merge-behavior">
  ### マージ時の挙動
</div>

* **単一ファイル**: `$include` を含むオブジェクト全体を置き換える
* **複数ファイルの配列**: 配列の順序でディープマージ（後ろのファイルが前のファイルを上書き）
* **同階層のキーあり**: `$include` の処理後に同階層のキーをマージ（インクルードされた値を上書き）
* **同階層のキー + 配列/プリミティブ**: サポートされない（インクルードされる内容はオブジェクトでなければならない）

```json5
// 同階層のキーはインクルードされた値を上書きする
{
  "$include": "./base.json5",   // { a: 1, b: 2 }
  b: 99                          // 結果: { a: 1, b: 99 }
}
```

<div id="nested-includes">
  ### ネストされたインクルード
</div>

インクルードされたファイル自身も `$include` ディレクティブを含めることができ、最大 10 階層まで入れ子にできます。

```json5
// clients/mueller.json5
{
  agents: { "$include": "./mueller/agents.json5" },
  broadcast: { "$include": "./mueller/broadcast.json5" }
}
```

<div id="path-resolution">
  ### パス解決
</div>

* **相対パス**: インクルード元ファイルからの相対パスとして解決されます
* **絶対パス**: そのまま使用されます
* **親ディレクトリ**: `../` 参照は期待どおりに動作します

```json5
{ "$include": "./sub/config.json5" }      // relative
{ "$include": "/etc/openclaw/base.json5" } // absolute
{ "$include": "../shared/common.json5" }   // 親ディレクトリ
```

<div id="error-handling">
  ### エラー処理
</div>

* **ファイル未検出**: 解決済みパスを含む明確なエラー
* **パースエラー**: どのインクルードファイルで失敗したかを表示
* **循環インクルード**: インクルードチェーンとともに検出・報告

<div id="example-multi-client-legal-setup">
  ### 例: 複数クライアント向けの法務設定
</div>

```json5
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789, auth: { token: "secret" } },
  
  // Common agent defaults
  agents: {
    defaults: {
      sandbox: { mode: "all", scope: "session" }
    },
    // 全クライアントのエージェントリストをマージ
    list: { "$include": [
      "./clients/mueller/agents.json5",
      "./clients/schmidt/agents.json5"
    ]}
  },
  
  // Merge broadcast configs
  broadcast: { "$include": [
    "./clients/mueller/broadcast.json5",
    "./clients/schmidt/broadcast.json5"
  ]},
  
  channels: { whatsapp: { groupPolicy: "allowlist" } }
}
```

```json5
// ~/.openclaw/clients/mueller/agents.json5
[
  { id: "mueller-transcribe", workspace: "~/clients/mueller/transcribe" },
  { id: "mueller-docs", workspace: "~/clients/mueller/docs" }
]
```

```json5
// ~/.openclaw/clients/mueller/broadcast.json5
{
  "120363403215116621@g.us": ["mueller-transcribe", "mueller-docs"]
}
```

<div id="common-options">
  ## 共通オプション
</div>

<div id="env-vars-env">
  ### 環境変数 + `.env`
</div>

OpenClaw は、親プロセス（シェル、launchd/systemd、CI など）の環境変数を読み込みます。

加えて、次も読み込みます:

* 現在の作業ディレクトリにある `.env`（存在する場合）
* グローバルなフォールバックとしての `~/.openclaw/.env`（別名 `$OPENCLAW_STATE_DIR/.env`）

どちらの `.env` ファイルも、既存の環境変数を上書きしません。

設定ファイル内でインラインの環境変数を指定することもできます。これらは、
プロセス環境にそのキーが存在しない場合にのみ適用されます（同じ「非上書き」ルールが適用されます）:

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-..."
    }
  }
}
```

優先順位と参照元の全体像については、[/environment](/ja/environment) を参照してください。

<div id="envshellenv-optional">
  ### `env.shellEnv` (任意)
</div>

オプトインの便利機能です。有効化されていて、想定されているキーがまだ一つも設定されていない場合、OpenClaw はログインシェルを実行し、不足している想定キーだけをインポートします（既存の値は決して上書きしません）。
これは実質的に、シェルのプロファイルを `source` するのと同等の動作になります。

```json5
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000
    }
  }
}
```

環境変数で指定する場合:

* `OPENCLAW_LOAD_SHELL_ENV=1`
* `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`

<div id="env-var-substitution-in-config">
  ### 設定における環境変数の置換
</div>

任意の設定項目の文字列値で、環境変数を `${VAR_NAME}` 構文を使って直接参照できます。変数は、検証が行われる前の、設定の読み込み時に置換されます。

```json5
{
  models: {
    providers: {
      "vercel-gateway": {
        apiKey: "${VERCEL_GATEWAY_API_KEY}"
      }
    }
  },
  gateway: {
    auth: {
      token: "${OPENCLAW_GATEWAY_TOKEN}"
    }
  }
}
```

**ルール:**

* マッチ対象は大文字の環境変数名のみ: `[A-Z_][A-Z0-9_]*`
* 環境変数が存在しない、または空の場合は、設定読み込み時にエラーが発生する
* `$${VAR}` と書くと、リテラルの `${VAR}` を出力する
* `$include` と併用可能（インクルードされたファイルにも置換が適用される）

**インライン置換:**

```json5
{
  models: {
    providers: {
      custom: {
        baseUrl: "${CUSTOM_API_BASE}/v1"  // → "https://api.example.com/v1"
      }
    }
  }
}
```

<div id="auth-storage-oauth-api-keys">
  ### 認証ストレージ（OAuth + API キー）
</div>

OpenClaw は **各エージェントごと** の認証プロファイル（OAuth + API キー）を次の場所に保存します:

* `<agentDir>/auth-profiles.json`（デフォルト: `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`）

関連項目: [/concepts/oauth](/ja/concepts/oauth)

レガシーな OAuth インポート:

* `~/.openclaw/credentials/oauth.json`（または `$OPENCLAW_STATE_DIR/credentials/oauth.json`）

組み込み Pi エージェントは、実行時キャッシュを次の場所に保持します:

* `<agentDir>/auth.json`（自動管理されます。手動で編集しないでください）

レガシーなエージェントディレクトリ（マルチエージェント対応以前）:

* `~/.openclaw/agent/*`（`openclaw doctor` によって `~/.openclaw/agents/<defaultAgentId>/agent/*` に移行されます）

オーバーライド:

* OAuth ディレクトリ（レガシーインポート専用）：`OPENCLAW_OAUTH_DIR`
* エージェントディレクトリ（デフォルトエージェントのルートディレクトリのオーバーライド先）：`OPENCLAW_AGENT_DIR`（推奨）、`PI_CODING_AGENT_DIR`（レガシー）

初回利用時には、OpenClaw は `oauth.json` のエントリを `auth-profiles.json` にインポートします。

<div id="auth">
  ### `auth`
</div>

認証プロファイル用の任意のメタデータです。ここにはシークレットは**保存されません**。
プロファイル ID をプロバイダーとモード（および任意の email）に対応付け、フェイルオーバー時に使用されるプロバイダーの切り替え順序を定義します。

```json5
{
  auth: {
    profiles: {
      "anthropic:me@example.com": { provider: "anthropic", mode: "oauth", email: "me@example.com" },
      "anthropic:work": { provider: "anthropic", mode: "api_key" }
    },
    order: {
      anthropic: ["anthropic:me@example.com", "anthropic:work"]
    }
  }
}
```

<div id="agentslistidentity">
  ### `agents.list[].identity`
</div>

デフォルト値および UX に使用される、エージェントごとの任意のアイデンティティ設定です。これは macOS のオンボーディングアシスタントによって書き込まれます。

設定されている場合、OpenClaw は（明示的に設定していない項目に限り）次のデフォルト値を決定します:

* `messages.ackReaction`: **アクティブなエージェント** の `identity.emoji` から設定されます（未設定の場合は 👀 にフォールバック）
* `agents.list[].groupChat.mentionPatterns`: エージェントの `identity.name` / `identity.emoji` から設定されます（これにより、Telegram/Slack/Discord/Google Chat/iMessage/WhatsApp のグループ全体で “@Samantha” のようなメンションが機能します）
* `identity.avatar` は、ワークスペースからの相対画像パス、またはリモート URL / data URL を受け付けます。ローカルファイルはエージェントのワークスペース内に存在している必要があります。

`identity.avatar` は以下を受け付けます:

* ワークスペースからの相対パス（エージェントのワークスペース内にとどまっている必要があります）
* `http(s)` URL
* `data:` URI

```json5
{
  agents: {
    list: [
      {
        id: "main",
        identity: {
          name: "Samantha",
          theme: "helpful sloth",
          emoji: "🦥",
          avatar: "avatars/samantha.png"
        }
      }
    ]
  }
}
```

<div id="wizard">
  ### `wizard`
</div>

CLI wizard（`onboard`、`configure`、`doctor`）によって生成されるメタデータ。

```json5
{
  wizard: {
    lastRunAt: "2026-01-01T00:00:00.000Z",
    lastRunVersion: "2026.1.4",
    lastRunCommit: "abc1234",
    lastRunCommand: "configure",
    lastRunMode: "local"
  }
}
```

<div id="logging">
  ### `logging`
</div>

* デフォルトのログファイル：`/tmp/openclaw/openclaw-YYYY-MM-DD.log`
* 固定されたパスを使いたい場合は、`logging.file` を `/tmp/openclaw/openclaw.log` に設定します。
* コンソール出力は次の設定で個別に調整できます：
  * `logging.consoleLevel`（デフォルトは `info`、`--verbose` 指定時は `debug` に引き上げ）
  * `logging.consoleStyle`（`pretty` | `compact` | `json`）
* 秘密情報の漏えいを防ぐため、ツールのサマリー出力をマスクできます：
  * `logging.redactSensitive`（`off` | `tools`、デフォルト：`tools`）
  * `logging.redactPatterns`（正規表現文字列の配列。デフォルト値を上書き）

```json5
{
  logging: {
    level: "info",
    file: "/tmp/openclaw/openclaw.log",
    consoleLevel: "info",
    consoleStyle: "pretty",
    redactSensitive: "tools",
    redactPatterns: [
      // 例: デフォルト設定を独自のルールで上書きする
      "\\bTOKEN\\b\\s*[=:]\\s*([\"']?)([^\\s\"']+)\\1",
      "/\\bsk-[A-Za-z0-9_-]{8,}\\b/gi"
    ]
  }
}
```

<div id="channelswhatsappdmpolicy">
  ### `channels.whatsapp.dmPolicy`
</div>

WhatsApp のダイレクトチャット（DM）をどのように扱うかを制御します:

* `"pairing"` (デフォルト): 未知の送信者にはペアリングコードを発行し、オーナーによる承認が必要です
* `"allowlist"`: `channels.whatsapp.allowFrom`（またはペアリング済みの allow ストア）に含まれる送信者のみ許可します
* `"open"`: すべての受信 DM を許可します（`channels.whatsapp.allowFrom` に `"*"` を含める **必要があります**）
* `"disabled"`: すべての受信 DM を無視します

ペアリングコードは 1 時間で失効します。ボットがペアリングコードを送信するのは、新しいリクエストが作成されたときのみです。保留中の DM ペアリングリクエストは、デフォルトで **チャネルごとに 3 件** に制限されます。

ペアリングの承認:

* `openclaw pairing list whatsapp`
* `openclaw pairing approve whatsapp <code>`

<div id="channelswhatsappallowfrom">
  ### `channels.whatsapp.allowFrom`
</div>

WhatsApp の自動返信をトリガーできる E.164 形式の電話番号の許可リスト（**DM のみ**）。
空で、かつ `channels.whatsapp.dmPolicy="pairing"` の場合、未知の送信者にはペアリングコードが送信されます。
グループ向けには、`channels.whatsapp.groupPolicy` と `channels.whatsapp.groupAllowFrom` を使用します。

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "+447700900123"],
      textChunkLimit: 4000, // optional outbound chunk size (chars)
      chunkMode: "length", // オプション: チャンク分割モード (length | newline)
      mediaMaxMb: 50 // optional inbound media cap (MB)
    }
  }
}
```

<div id="channelswhatsappsendreadreceipts">
  ### `channels.whatsapp.sendReadReceipts`
</div>

受信した WhatsApp メッセージを既読（青いチェックマーク）としてマークするかどうかを制御します。デフォルト: `true`。

セルフチャットモードでは、有効になっていても常に既読通知の送信をスキップします。

アカウントごとの上書き設定: `channels.whatsapp.accounts.<id>.sendReadReceipts`。

```json5
{
  channels: {
    whatsapp: { sendReadReceipts: false }
  }
}
```

<div id="channelswhatsappaccounts-multi-account">
  ### `channels.whatsapp.accounts` (複数アカウント)
</div>

1つの Gateway で複数の WhatsApp アカウントを扱えます:

```json5
{
  channels: {
    whatsapp: {
      accounts: {
        default: {}, // optional; keeps the default id stable
        personal: {},
        biz: {
          // オプションで上書き可能。デフォルト: ~/.openclaw/credentials/whatsapp/biz
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        }
      }
    }
  }
}
```

Notes:

* 送信コマンドは、アカウント `default` が存在する場合はそれを既定として使用します。存在しない場合は、設定済みアカウント ID をソートした際の先頭のものが使用されます。
* 旧来の単一アカウント用 Baileys 認証ディレクトリは、`openclaw doctor` によって `whatsapp/default` に移行されます。

<div id="channelstelegramaccounts-channelsdiscordaccounts-channelsgooglechataccounts-channelsslackaccounts-channelsmattermostaccounts-channelssignalaccounts-channelsimessageaccounts">
  ### `channels.telegram.accounts` / `channels.discord.accounts` / `channels.googlechat.accounts` / `channels.slack.accounts` / `channels.mattermost.accounts` / `channels.signal.accounts` / `channels.imessage.accounts`
</div>

チャネルごとに複数のアカウントを設定できます（各アカウントは固有の `accountId` を持ち、任意で `name` を指定できます）：

```json5
{
  channels: {
    telegram: {
      accounts: {
        default: {
          name: "Primary bot",
          botToken: "123456:ABC..."
        },
        alerts: {
          name: "Alerts bot",
          botToken: "987654:XYZ..."
        }
      }
    }
  }
}
```

Notes:

* `accountId` が省略された場合は `default` が使用されます（CLI とルーティング）。
* 環境変数由来のトークンは **default** アカウントにのみ適用されます。
* 基本チャネル設定（グループポリシー、メンション制御など）は、アカウントごとの上書きがない限り、すべてのアカウントに適用されます。
* 各アカウントをそれぞれ異なる agents.defaults にルーティングするには、`bindings[].match.accountId` を使用してください。

<div id="group-chat-mention-gating-agentslistgroupchat-messagesgroupchat">
  ### グループチャットでのメンション制御（`agents.list[].groupChat` + `messages.groupChat`）
</div>

グループチャットでのメッセージは、デフォルトで **メンション必須**（メタデータによるメンション、または正規表現パターンのいずれかが必要）になります。WhatsApp、Telegram、Discord、Google Chat、iMessage のグループチャットに適用されます。

**メンションの種類:**

* **メタデータメンション**: プラットフォームネイティブの @メンション（例: WhatsApp の「タップしてメンション」）。WhatsApp のセルフチャットモードでは無視されます（`channels.whatsapp.allowFrom` を参照）。
* **テキストパターン**: `agents.list[].groupChat.mentionPatterns` で定義される正規表現パターン。セルフチャットモードかどうかに関係なく常にチェックされます。
* メンション検出が可能な場合（ネイティブメンション、または少なくとも 1 つの `mentionPattern` が存在する場合）にのみ、メンション制御が適用されます。

```json5
{
  messages: {
    groupChat: { historyLimit: 50 }
  },
  agents: {
    list: [
      { id: "main", groupChat: { mentionPatterns: ["@openclaw", "openclaw"] } }
    ]
  }
}
```

`messages.groupChat.historyLimit` はグループチャット履歴コンテキストのグローバルデフォルト値を設定します。チャンネルごとに `channels.<channel>.historyLimit`（マルチアカウントの場合は `channels.<channel>.accounts.*.historyLimit`）で上書きできます。`0` を設定すると履歴ラッピングが無効になります。

<div id="dm-history-limits">
  #### DM の履歴上限
</div>

DM の会話は、エージェントによって管理されるセッションベースの履歴を使用します。DM セッションごとに保持するユーザーの発話（ターン）の数を制限できます。

```json5
{
  channels: {
    telegram: {
      dmHistoryLimit: 30,  // limit DM sessions to 30 user turns
      dms: {
        "123456789": { historyLimit: 50 }  // ユーザー単位のオーバーライド（ユーザーID）
      }
    }
  }
}
```

適用順序:

1. DM単位のオーバーライド: `channels.<provider>.dms[userId].historyLimit`
2. プロバイダーのデフォルト: `channels.<provider>.dmHistoryLimit`
3. 制限なし（すべての履歴を保持）

対応プロバイダー: `telegram`, `whatsapp`, `discord`, `slack`, `signal`, `imessage`, `msteams`.

エージェント単位のオーバーライド（設定されている場合は、`[]` であってもこちらが優先されます）:

```json5
{
  agents: {
    list: [
      { id: "work", groupChat: { mentionPatterns: ["@workbot", "\\+15555550123"] } },
      { id: "personal", groupChat: { mentionPatterns: ["@homebot", "\\+15555550999"] } }
    ]
  }
}
```

メンション制御のデフォルト設定はチャンネルごとに存在します（`channels.whatsapp.groups`、`channels.telegram.groups`、`channels.imessage.groups`、`channels.discord.guilds`）。`*.groups` が設定されている場合、それはグループの許可リストとしても機能します。すべてのグループを許可するには `"*"` を含めてください。

ネイティブの @ メンションを無視して、特定のテキストトリガーに対して **のみ** 応答するには:

```json5
{
  channels: {
    whatsapp: {
      // セルフチャットモードを有効にするには自分の番号を含める(ネイティブ@メンションは無視される)
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } }
    }
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          // Only these text patterns will trigger responses
          mentionPatterns: ["reisponde", "@openclaw"]
        }
      }
    ]
  }
}
```

<div id="group-policy-per-channel">
  ### グループポリシー（チャネルごと）
</div>

`channels.*.groupPolicy` を使用して、グループ／ルームでのメッセージを受信するかどうかを制御します。

```json5
{
  channels: {
    whatsapp: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"]
    },
    telegram: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["tg:123456789", "@alice"]
    },
    signal: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"]
    },
    imessage: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["chat_id:123"]
    },
    msteams: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["user@org.com"]
    },
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        "GUILD_ID": {
          channels: { help: { allow: true } }
        }
      }
    },
    slack: {
      groupPolicy: "allowlist",
      channels: { "#general": { allow: true } }
    }
  }
}
```

補足:

* `"open"`: グループは許可リストをバイパスしますが、メンションによる制御（mention-gating）は引き続き適用されます。
* `"disabled"`: すべてのグループ/ルームメッセージをブロックします。
* `"allowlist"`: 設定された許可リストに一致するグループ/ルームのみを許可します。
* `channels.defaults.groupPolicy` は、プロバイダーの `groupPolicy` が未設定の場合のデフォルト値を設定します。
* WhatsApp/Telegram/Signal/iMessage/Microsoft Teams は `groupAllowFrom` を使用します（フォールバックとして明示的な `allowFrom` を使用）。
* Discord/Slack はチャネルの許可リスト（`channels.discord.guilds.*.channels`, `channels.slack.channels`）を使用します。
* グループ DM（Discord/Slack）は引き続き `dm.groupEnabled` と `dm.groupChannels` によって制御されます。
* デフォルトは `groupPolicy: "allowlist"` です（`channels.defaults.groupPolicy` で上書きされない限り）。許可リストが設定されていない場合、グループメッセージはブロックされます。

<div id="multi-agent-routing-agentslist-bindings">
  ### Multi-agent routing (`agents.list` + `bindings`)
</div>

複数の分離されたエージェント（個別のワークスペース、`agentDir`、セッション）を 1 つの Gateway 内で実行します。
受信メッセージは `bindings` 経由でエージェントにルーティングされます。

* `agents.list[]`: エージェントごとのオーバーライド。
  * `id`: 安定したエージェント ID（必須）。
  * `default`: 任意。複数が設定されている場合は、最初のものが優先され、警告がログに記録されます。
    どれも設定されていない場合、リストの**最初のエントリ**がデフォルトエージェントになります。
  * `name`: エージェントの表示名。
  * `workspace`: デフォルトは `~/.openclaw/workspace-<agentId>`（`main` の場合は `agents.defaults.workspace` をフォールバックとして使用）。
  * `agentDir`: デフォルトは `~/.openclaw/agents/<agentId>/agent`。
  * `model`: エージェントごとのデフォルトモデル。このエージェントに対して `agents.defaults.model` をオーバーライドします。
    * 文字列形式: `"provider/model"`。`agents.defaults.model.primary` のみをオーバーライド
    * オブジェクト形式: `{ primary, fallbacks }`（`fallbacks` は `agents.defaults.model.fallbacks` をオーバーライドします。`[]` を指定すると、そのエージェントに対するグローバル fallback を無効化）
  * `identity`: エージェントごとの名前 / テーマ / 絵文字（メンションパターンおよび ACK リアクションに使用）。
  * `groupChat`: エージェントごとのメンション・ゲーティング（`mentionPatterns`）。
  * `sandbox`: エージェントごとのサンドボックス設定（`agents.defaults.sandbox` をオーバーライド）。
    * `mode`: `"off"` | `"non-main"` | `"all"`
    * `workspaceAccess`: `"none"` | `"ro"` | `"rw"`
    * `scope`: `"session"` | `"agent"` | `"shared"`
    * `workspaceRoot`: カスタムサンドボックスワークスペースのルート
    * `docker`: エージェントごとの Docker オーバーライド（例: `image`、`network`、`env`、`setupCommand`、各種リソース制限。`scope: "shared"` の場合は無視）
    * `browser`: エージェントごとのサンドボックス化されたブラウザ設定のオーバーライド（`scope: "shared"` の場合は無視）
    * `prune`: エージェントごとのサンドボックス掃除（pruning）設定のオーバーライド（`scope: "shared"` の場合は無視）
  * `subagents`: エージェントごとのサブエージェントのデフォルト。
    * `allowAgents`: このエージェントからの `sessions_spawn` に許可するエージェント ID の許可リスト（`["*"]` = すべて許可。デフォルト: 同一エージェントのみ）
  * `tools`: エージェントごとのツール制限（サンドボックスのツールポリシーより前に適用）。
    * `profile`: ベースとなるツールプロファイル（allow/deny の前に適用）
    * `allow`: 許可するツール名の配列
    * `deny`: 拒否するツール名の配列（deny が優先）
* `agents.defaults`: 共有エージェントデフォルト（model、workspace、sandbox など）。
* `bindings[]`: 受信メッセージを `agentId` にルーティングします。
  * `match.channel`（必須）
  * `match.accountId`（任意。`*` = 任意のアカウント、省略時 = デフォルトアカウント）
  * `match.peer`（任意。`{ kind: dm|group|channel, id }`）
  * `match.guildId` / `match.teamId`（任意。チャネル固有）

決定的なマッチ順序:

1. `match.peer`
2. `match.guildId`
3. `match.teamId`
4. `match.accountId`（peer/guild/team なしでの厳密一致）
5. `match.accountId: "*"`（peer/guild/team なしでのチャネル全体）
6. デフォルトエージェント（`agents.list[].default`、なければリストの先頭エントリ、それもなければ `"main"`）

各マッチ階層内では、`bindings` のうち最初にマッチしたエントリが優先されます。

<div id="per-agent-access-profiles-multi-agent">
  #### エージェントごとのアクセスプロファイル（マルチエージェント）
</div>

各エージェントは、それぞれ独自のサンドボックスとツールポリシーを持つことができます。これを使って、1つのGateway内で
異なるアクセスレベルを組み合わせて構成できます。

* **フルアクセス**（パーソナルエージェント）
* **読み取り専用**ツール + ワークスペース
* **ファイルシステムへのアクセスなし**（メッセージング／セッション用ツールのみ）

優先順位や追加の例については、[マルチエージェント サンドボックスとツール](/ja/multi-agent-sandbox-tools) を参照してください。

フルアクセス（サンドボックスなし）:

```json5
{
  agents: {
    list: [
      {
        id: "personal",
        workspace: "~/.openclaw/workspace-personal",
        sandbox: { mode: "off" }
      }
    ]
  }
}
```

読み取り専用のツール + 読み取り専用のワークスペース：

```json5
{
  agents: {
    list: [
      {
        id: "family",
        workspace: "~/.openclaw/workspace-family",
        sandbox: {
          mode: "all",
          scope: "agent",
          workspaceAccess: "ro"
        },
        tools: {
          allow: ["read", "sessions_list", "sessions_history", "sessions_send", "sessions_spawn", "session_status"],
          deny: ["write", "edit", "apply_patch", "exec", "process", "browser"]
        }
      }
    ]
  }
}
```

ファイルシステムにはアクセスしない（メッセージング／セッション向けツールは有効）:

```json5
{
  agents: {
    list: [
      {
        id: "public",
        workspace: "~/.openclaw/workspace-public",
        sandbox: {
          mode: "all",
          scope: "agent",
          workspaceAccess: "none"
        },
        tools: {
          allow: ["sessions_list", "sessions_history", "sessions_send", "sessions_spawn", "session_status", "whatsapp", "telegram", "slack", "discord", "gateway"],
          deny: ["read", "write", "edit", "apply_patch", "exec", "process", "browser", "canvas", "nodes", "cron", "gateway", "image"]
        }
      }
    ]
  }
}
```

例：2つのWhatsAppアカウント → 2つのエージェント

```json5
{
  agents: {
    list: [
      { id: "home", default: true, workspace: "~/.openclaw/workspace-home" },
      { id: "work", workspace: "~/.openclaw/workspace-work" }
    ]
  },
  bindings: [
    { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } }
  ],
  channels: {
    whatsapp: {
      accounts: {
        personal: {},
        biz: {},
      }
    }
  }
}
```

<div id="toolsagenttoagent-optional">
  ### `tools.agentToAgent` (任意)
</div>

エージェント間メッセージングはオプトイン（明示的な有効化）が必要です：

```json5
{
  tools: {
    agentToAgent: {
      enabled: false,
      allow: ["home", "work"]
    }
  }
}
```

<div id="messagesqueue">
  ### `messages.queue`
</div>

エージェントの実行がすでに進行中の場合に、受信メッセージの挙動をどのようにするかを制御します。

```json5
{
  messages: {
    queue: {
      mode: "collect", // steer | followup | collect | steer-backlog (steer+backlog ok) | interrupt (queue=steer レガシー)
      debounceMs: 1000,
      cap: 20,
      drop: "summarize", // old | new | summarize
      byChannel: {
        whatsapp: "collect",
        telegram: "collect",
        discord: "collect",
        imessage: "collect",
        webchat: "collect"
      }
    }
  }
}
```

<div id="messagesinbound">
  ### `messages.inbound`
</div>

**同一の送信者**から短時間に連続して届く受信メッセージをデバウンスし、複数の
メッセージを 1 回のエージェントのターンとして扱います。デバウンスはチャンネル＋会話単位で行われ、
返信時のスレッド／ID には最新のメッセージが使用されます。

```json5
{
  messages: {
    inbound: {
      debounceMs: 2000, // 0 で無効化
      byChannel: {
        whatsapp: 5000,
        slack: 1500,
        discord: 1500
      }
    }
  }
}
```

Notes:

* デバウンスは **テキストのみ** のメッセージをバッチ処理し、メディア／添付ファイルは即座に送信します。
* 制御コマンド（例: `/queue`, `/new`）はデバウンスの対象外となり、常に単独のメッセージとして扱われます。

<div id="commands-chat-command-handling">
  ### `commands` (チャットコマンド処理)
</div>

各コネクタでチャットコマンドをどのように有効化するかを制御します。

```json5
{
  commands: {
    native: "auto",         // register native commands when supported (auto)
    text: true,             // parse slash commands in chat messages
    bash: false,            // ! を許可 (エイリアス: /bash) (ホストのみ; tools.elevated 許可リストが必要)
    bashForegroundMs: 2000, // bash foreground window (0 backgrounds immediately)
    config: false,          // allow /config (writes to disk)
    debug: false,           // allow /debug (runtime-only overrides)
    restart: false,         // allow /restart + gateway restart tool
    useAccessGroups: true   // enforce access-group allowlists/policies for commands
  }
}
```

注意:

* テキストコマンドは **単独の** メッセージとして送信し、先頭に `/` を付ける必要があります（プレーンテキストのエイリアスは使用できません）。
* `commands.text: false` は、チャットメッセージからのコマンド解釈を無効化します。
* `commands.native: "auto"`（デフォルト）は Discord/Telegram でネイティブコマンドを有効にし、Slack は無効のままにします。未対応のチャネルはテキストコマンドのみになります。
* すべてに強制するには `commands.native: true|false` を設定し、チャネルごとに上書きするには `channels.discord.commands.native`, `channels.telegram.commands.native`, `channels.slack.commands.native`（bool または `"auto"`）を使用します。`false` の場合、Discord/Telegram では起動時に以前登録されたコマンドが削除されます。Slack のコマンドは Slack アプリ側で管理されます。
* `channels.telegram.customCommands` は Telegram ボットメニューの追加項目を定義します。名前は正規化され、ネイティブコマンドとの競合は無視されます。
* `commands.bash: true` で、ホストシェルコマンドを実行する `! <cmd>` を有効にします（エイリアスとして `/bash <cmd>` も使用可能）。`tools.elevated.enabled` が有効であり、かつ送信者が `tools.elevated.allowFrom.<channel>` の許可リストに入っている必要があります。
* `commands.bashForegroundMs` は、バックグラウンドに移行する前に bash が待機する時間を制御します。bash ジョブ実行中は、新しい `! <cmd>` 要求は拒否されます（同時に 1 ジョブのみ）。
* `commands.config: true` で `/config` を有効にします（`openclaw.json` の読み書きを行います）。
* `channels.<provider>.configWrites` は、そのチャネル経由で開始される設定変更に対するゲートとして機能します（デフォルト: true）。これは `/config set|unset` と、プロバイダー固有の自動マイグレーション（Telegram スーパーグループ ID の変更、Slack チャネル ID の変更）に適用されます。
* `commands.debug: true` で `/debug` を有効にします（実行時のみのオーバーライド）。
* `commands.restart: true` で `/restart` および Gateway ツールの再起動アクションを有効にします。
* `commands.useAccessGroups: false` の場合、コマンドはアクセスグループの許可リスト／ポリシーをバイパスできます。
* スラッシュコマンドおよびディレクティブは、**認可された送信者** に対してのみ有効です。認可は、チャネルの許可リスト／ペアリングおよび `commands.useAccessGroups` に基づいて決定されます。

<div id="web-whatsapp-web-channel-runtime">
  ### `web` (WhatsApp Web チャネルのランタイム)
</div>

WhatsApp は Gateway の web チャネル (Baileys Web) 経由で動作します。リンク済みのセッションが存在する場合は自動的に起動します。
標準で無効にしておくには、`web.enabled: false` を設定します。

```json5
{
  web: {
    enabled: true,
    heartbeatSeconds: 60,
    reconnect: {
      initialMs: 2000,
      maxMs: 120000,
      factor: 1.4,
      jitter: 0.2,
      maxAttempts: 0
    }
  }
}
```

<div id="channelstelegram-bot-transport">
  ### `channels.telegram` (bot transport)
</div>

OpenClaw は、`channels.telegram` 設定セクションが存在する場合にのみ Telegram を起動します。Bot トークンは `channels.telegram.botToken`（または `channels.telegram.tokenFile`）から取得され、デフォルトアカウント向けのフォールバックとして `TELEGRAM_BOT_TOKEN` が使用されます。
自動起動を無効にするには、`channels.telegram.enabled: false` を設定します。
マルチアカウントのサポートは `channels.telegram.accounts` 配下にあります（上記のマルチアカウントセクションを参照してください）。環境変数によるトークンはデフォルトアカウントにのみ適用されます。
Telegram からの設定書き込み（スーパグループ ID マイグレーションや `/config set|unset` を含む）をブロックするには、`channels.telegram.configWrites: false` を設定します。

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "your-bot-token",
      dmPolicy: "pairing",                 // pairing | allowlist | open | disabled
      allowFrom: ["tg:123456789"],         // optional; "open" requires ["*"]
      groups: {
        "*": { requireMention: true },
        "-1001234567890": {
          allowFrom: ["@admin"],
          systemPrompt: "Keep answers brief.",
          topics: {
            "99": {
              requireMention: false,
              skills: ["search"],
              systemPrompt: "Stay on topic."
            }
          }
        }
      },
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" }
      ],
      historyLimit: 50,                     // include last N group messages as context (0 disables)
      replyToMode: "first",                 // off | first | all
      linkPreview: true,                   // toggle outbound link previews
      streamMode: "partial",               // off | partial | block (下書きストリーミング; ブロックストリーミングとは別)
      draftChunk: {                        // optional; only for streamMode=block
        minChars: 200,
        maxChars: 800,
        breakPreference: "paragraph"       // paragraph | newline | sentence
      },
      actions: { reactions: true, sendMessage: true }, // tool action gates (false disables)
      reactionNotifications: "own",   // off | own | all
      mediaMaxMb: 5,
      retry: {                             // outbound retry policy
        attempts: 3,
        minDelayMs: 400,
        maxDelayMs: 30000,
        jitter: 0.1
      },
      network: {                           // transport overrides
        autoSelectFamily: false
      },
      proxy: "socks5://localhost:9050",
      webhookUrl: "https://example.com/telegram-webhook",
      webhookSecret: "secret",
      webhookPath: "/telegram-webhook"
    }
  }
}
```

Draft streaming に関するメモ:

* Telegram の `sendMessageDraft` を使用します（実際のメッセージではなく、下書きバブル）。
* **private chat topics** が必要です（DM 内の message&#95;thread&#95;id；bot 側でトピックが有効になっている必要があります）。
* `/reasoning stream` は推論内容を下書きにストリーミングし、その後、最終的な回答を送信します。
  リトライポリシーの既定値と動作については、[Retry policy](/ja/concepts/retry) を参照してください。

<div id="channelsdiscord-bot-transport">
  ### `channels.discord` (bot transport)
</div>

Discord ボットは、ボットトークンと任意のアクセス制御（gating）を設定して構成します。
マルチアカウントのサポートは `channels.discord.accounts` で行います（上記のマルチアカウントセクションを参照してください）。環境変数のトークンはデフォルトアカウントにのみ適用されます。

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "your-bot-token",
      mediaMaxMb: 8,                          // clamp inbound media size
      allowBots: false,                       // allow bot-authored messages
      actions: {                              // tool action gates (false disables)
        reactions: true,
        stickers: true,
        polls: true,
        permissions: true,
        messages: true,
        threads: true,
        pins: true,
        search: true,
        memberInfo: true,
        roleInfo: true,
        roles: false,
        channelInfo: true,
        voiceStatus: true,
        events: true,
        moderation: false
      },
      replyToMode: "off",                     // off | first | all
      dm: {
        enabled: true,                        // disable all DMs when false
        policy: "pairing",                    // pairing | allowlist | open | disabled
        allowFrom: ["1234567890", "steipete"], // optional DM allowlist ("open" requires ["*"])
        groupEnabled: false,                 // enable group DMs
        groupChannels: ["openclaw-dm"]          // optional group DM allowlist
      },
      guilds: {
        "123456789012345678": {               // guild id (preferred) or slug
          slug: "friends-of-openclaw",
          requireMention: false,              // per-guild default
          reactionNotifications: "own",       // off | own | all | allowlist
          users: ["987654321098765432"],      // optional per-guild user allowlist
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["docs"],
              systemPrompt: "Short answers only."
            }
          }
        }
      },
      historyLimit: 20,                       // include last N guild messages as context
      textChunkLimit: 2000,                   // optional outbound text chunk size (chars)
      chunkMode: "length",                    // optional chunking mode (length | newline)
      maxLinesPerMessage: 17,                 // メッセージあたりの最大行数の目安(Discord UIのクリッピング対策)
      retry: {                                // outbound retry policy
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1
      }
    }
  }
}
```

`channels.discord` 設定セクションが存在する場合にのみ、OpenClaw は Discord を起動します。トークンはまず `channels.discord.token` から取得され、デフォルトアカウント用のフォールバックとして `DISCORD_BOT_TOKEN` が使用されます（ただし `channels.discord.enabled` が `false` の場合を除きます）。cron/CLI コマンドの配信ターゲットを指定する際は、`user:<id>`（DM）または `channel:<id>`（ギルドチャンネル）を使用してください。数値のみの ID はあいまいなため拒否されます。
ギルドのスラッグは小文字で、スペースは `-` に置き換えます。チャンネルキーにはスラッグ化したチャンネル名（先頭に `#` は付けない）を使用します。リネームによるあいまいさを避けるため、キーとしてはギルド ID を優先してください。
Bot が投稿したメッセージはデフォルトで無視されます。`channels.discord.allowBots` を有効化すると処理されます（自己返信ループを防ぐため、自分自身のメッセージは引き続きフィルタされます）。
リアクション通知モード:

* `off`: リアクションイベントなし。
* `own`: Bot 自身のメッセージへのリアクション（デフォルト）。
* `all`: すべてのメッセージへのすべてのリアクション。
* `allowlist`: すべてのメッセージに対する `guilds.<id>.users` からのリアクション（リストが空の場合は無効）。
  送信テキストは `channels.discord.textChunkLimit`（デフォルト 2000）で分割されます。`channels.discord.chunkMode="newline"` を設定すると、長さによる分割の前に空行（段落境界）で分割します。Discord クライアントは非常に縦長のメッセージを切り詰める場合があるため、`channels.discord.maxLinesPerMessage`（デフォルト 17）は 2000 文字未満であっても行数の多い返信を分割します。
  リトライポリシーのデフォルトと動作については、[Retry policy](/ja/concepts/retry) を参照してください。

<div id="channelsgooglechat-chat-api-webhook">
  ### `channels.googlechat` (Chat API webhook)
</div>

Google Chat は、アプリレベル認証（サービスアカウント）付きの HTTP webhook で動作します。
マルチアカウント対応は `channels.googlechat.accounts` 配下で行います（上記のマルチアカウントセクションを参照してください）。環境変数はデフォルトアカウントにのみ有効です。

```json5
{
  channels: {
    "googlechat": {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      audienceType: "app-url",             // app-url | project-number
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890",        // オプション; メンション検出を改善します
      dm: {
        enabled: true,
        policy: "pairing",                // pairing | allowlist | open | disabled
        allowFrom: ["users/1234567890"]   // オプション; "open" には ["*"] が必要です
      },
      groupPolicy: "allowlist",
      groups: {
        "spaces/AAAA": { allow: true, requireMention: true }
      },
      actions: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20
    }
  }
}
```

注意:

* サービスアカウントの JSON は、インライン（`serviceAccount`）またはファイル指定（`serviceAccountFile`）のどちらでも利用できます。
* デフォルトアカウント向けの環境変数フォールバック: `GOOGLE_CHAT_SERVICE_ACCOUNT` または `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE`。
* `audienceType` と `audience` は、Chat アプリの Webhook 認証設定と一致している必要があります。
* 送信先を設定する際は、`spaces/<spaceId>` または `users/<userId|email>` を使用します。

<div id="channelsslack-socket-mode">
  ### `channels.slack` (socket mode)
</div>

Slack は Socket Mode で動作し、Bot トークンおよび App トークンの両方が必要です。

```json5
{
  channels: {
    slack: {
      enabled: true,
      botToken: "xoxb-...",
      appToken: "xapp-...",
      dm: {
        enabled: true,
        policy: "pairing", // pairing | allowlist | open | disabled
        allowFrom: ["U123", "U456", "*"], // optional; "open" requires ["*"]
        groupEnabled: false,
        groupChannels: ["G123"]
      },
      channels: {
        C123: { allow: true, requireMention: true, allowBots: false },
        "#general": {
          allow: true,
          requireMention: true,
          allowBots: false,
          users: ["U123"],
          skills: ["docs"],
          systemPrompt: "Short answers only."
        }
      },
      historyLimit: 50,          // コンテキストとして最後のN件のチャンネル/グループメッセージを含める (0で無効化)
      allowBots: false,
      reactionNotifications: "own", // off | own | all | allowlist
      reactionAllowlist: ["U123"],
      replyToMode: "off",           // off | first | all
      thread: {
        historyScope: "thread",     // thread | channel
        inheritParent: false
      },
      actions: {
        reactions: true,
        messages: true,
        pins: true,
        memberInfo: true,
        emojiList: true
      },
      slashCommand: {
        enabled: true,
        name: "openclaw",
        sessionPrefix: "slack:slash",
        ephemeral: true
      },
      textChunkLimit: 4000,
      chunkMode: "length",
      mediaMaxMb: 20
    }
  }
}
```

複数アカウントのサポートは `channels.slack.accounts` の下にあります（上記のマルチアカウントのセクションを参照）。環境変数トークンはデフォルトアカウントにのみ適用されます。

OpenClaw は、プロバイダーが有効になっていて、両方のトークン（設定または `SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN`）が設定されている場合に Slack を起動します。cron/CLI コマンドの配信先を指定する際は、`user:<id>`（DM）または `channel:<id>` を使用してください。
Slack 発の設定書き込み（チャネル ID の移行や `/config set|unset` を含む）をブロックするには、`channels.slack.configWrites: false` を設定します。

Bot が投稿したメッセージはデフォルトで無視されます。有効化するには、`channels.slack.allowBots` または `channels.slack.channels.<id>.allowBots` を使用します。

リアクション通知モード:

* `off`: リアクションイベントなし。
* `own`: Bot 自身のメッセージへのリアクション（デフォルト）。
* `all`: すべてのメッセージへのすべてのリアクション。
* `allowlist`: すべてのメッセージに対して、`channels.slack.reactionAllowlist` に含まれる送信元からのリアクション（空リストの場合は無効）。

スレッドセッションの分離:

* `channels.slack.thread.historyScope` は、スレッド履歴をスレッド単位（`thread`, デフォルト）にするか、チャネル全体で共有（`channel`）にするかを制御します。
* `channels.slack.thread.inheritParent` は、新しいスレッドセッションが親チャネルのトランスクリプトを継承するかどうかを制御します（デフォルト: false）。

Slack アクショングループ（`slack` ツールアクションの制御）:

| Action group | デフォルト   | 説明            |
| ------------ | ------- | ------------- |
| reactions    | enabled | リアクション＋一覧取得   |
| messages     | enabled | 読み取り／送信／編集／削除 |
| pins         | enabled | ピン留め／解除／一覧    |
| memberInfo   | enabled | メンバー情報        |
| emojiList    | enabled | カスタム絵文字一覧     |

<div id="channelsmattermost-bot-token">
  ### `channels.mattermost` (bot トークン)
</div>

Mattermost はプラグインとして提供されており、コアインストールには含まれていません。
まず次を実行してインストールしてください: `openclaw plugins install @openclaw/mattermost`（または git チェックアウト環境では `./extensions/mattermost`）。

Mattermost では、bot トークンとサーバーのベース URL が必要です:

```json5
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing",
      chatmode: "oncall", // oncall | onmessage | onchar
      oncharPrefixes: [">", "!"],
      textChunkLimit: 4000,
      chunkMode: "length"
    }
  }
}
```

OpenClaw は、Mattermost アカウントが（bot トークン + ベース URL）で設定され有効化されている場合に、Mattermost を起動します。トークンとベース URL は、デフォルトアカウントについては `channels.mattermost.botToken` と `channels.mattermost.baseUrl`、または `MATTERMOST_BOT_TOKEN` と `MATTERMOST_URL` から取得されます（`channels.mattermost.enabled` が `false` でない限り）。

チャットモード:

* `oncall`（デフォルト）: @メンションされたときのみチャンネルメッセージに応答します。
* `onmessage`: すべてのチャンネルメッセージに応答します。
* `onchar`: トリガープレフィックス（`channels.mattermost.oncharPrefixes`、デフォルトは `[">", "!"]`）でメッセージが始まるときに応答します。

アクセス制御:

* デフォルトの DM: `channels.mattermost.dmPolicy="pairing"`（不明な送信者にはペアリングコードが渡されます）。
* 公開 DM: `channels.mattermost.dmPolicy="open"` と `channels.mattermost.allowFrom=["*"]` の組み合わせ（設定による送信者制限がなく、任意のユーザーからのメッセージ受信を許可するモード）。
* グループ: デフォルトでは `channels.mattermost.groupPolicy="allowlist"`（メンション必須）です。送信者を制限するには `channels.mattermost.groupAllowFrom` を使用します。

マルチアカウント対応は `channels.mattermost.accounts` の配下にあります（上記のマルチアカウントセクションを参照してください）。環境変数はデフォルトアカウントにのみ適用されます。
配信先を指定する際は `channel:<id>` または `user:<id>`（もしくは `@username`）を使用します。プレーンな id はチャンネル id として扱われます。

<div id="channelssignal-signal-cli">
  ### `channels.signal` (signal-cli)
</div>

Signal のリアクションは、システムイベント（共有リアクション用ツール群）を送出できます。

```json5
{
  channels: {
    signal: {
      reactionNotifications: "own", // off | own | all | allowlist
      reactionAllowlist: ["+15551234567", "uuid:123e4567-e89b-12d3-a456-426614174000"],
      historyLimit: 50 // 最後のN件のグループメッセージをコンテキストとして含める (0で無効化)
    }
  }
}
```

リアクション通知モード:

* `off`: リアクションイベントは発生しない。
* `own`: ボット自身が送信したメッセージへのリアクション（デフォルト）。
* `all`: すべてのメッセージに対するすべてのリアクション。
* `allowlist`: すべてのメッセージに対する、`channels.signal.reactionAllowlist` に含まれる相手からのリアクション（リストが空の場合は無効）。

<div id="channelsimessage-imsg-cli">
  ### `channels.imessage` (imsg CLI)
</div>

OpenClaw は `imsg rpc`（標準入出力経由の JSON-RPC）を起動します。デーモンやポートは不要です。

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "imsg",
      dbPath: "~/Library/Messages/chat.db",
      remoteHost: "user@gateway-host", // SSHラッパー使用時のリモート添付ファイル用SCP
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "user@example.com", "chat_id:123"],
      historyLimit: 50,    // include last N group messages as context (0 disables)
      includeAttachments: false,
      mediaMaxMb: 16,
      service: "auto",
      region: "US"
    }
  }
}
```

マルチアカウント対応は `channels.imessage.accounts` 配下で設定します（上記のマルチアカウントセクションを参照してください）。

注意事項:

* Messages のデータベースへの Full Disk Access が必要です。
* 最初の送信時に、Messages の自動化アクセス許可を求めるダイアログが表示されます。
* `chat_id:<id>` をターゲットとして優先的に使用してください。チャット一覧を表示するには `imsg chats --limit 20` を使用します。
* `channels.imessage.cliPath` はラッパースクリプト（例: `imsg rpc` を実行する別の Mac への `ssh`）を指すことができます。パスワード入力を避けるために SSH キーを使用してください。
* リモート SSH ラッパーの場合、`includeAttachments` が有効なときに SCP 経由で添付ファイルを取得できるように、`channels.imessage.remoteHost` を設定してください。

ラッパーの例:

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

<div id="agentsdefaultsworkspace">
  ### `agents.defaults.workspace`
</div>

エージェントがファイル操作を行う際に使用する**単一のグローバルワークスペースディレクトリ**を設定します。

デフォルト: `~/.openclaw/workspace`。

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } }
}
```

`agents.defaults.sandbox` が有効な場合、メイン以外のセッションは、
`agents.defaults.sandbox.workspaceRoot` 配下のスコープ単位のワークスペースで
この設定を上書きできます。

<div id="agentsdefaultsreporoot">
  ### `agents.defaults.repoRoot`
</div>

システムプロンプトの Runtime 行に表示するための任意のリポジトリルートです。未設定の場合、OpenClaw は
ワークスペース（およびカレント作業ディレクトリ）から上位ディレクトリへ向かってたどり、`.git` ディレクトリを検出しようとします。
指定されたパスが実在する場合にのみ利用されます。

```json5
{
  agents: { defaults: { repoRoot: "~/Projects/openclaw" } }
}
```

<div id="agentsdefaultsskipbootstrap">
  ### `agents.defaults.skipBootstrap`
</div>

ワークスペースのブートストラップファイル（`AGENTS.md`、`SOUL.md`、`TOOLS.md`、`IDENTITY.md`、`USER.md`、`BOOTSTRAP.md`）の自動作成を無効にします。

ワークスペースのファイルをリポジトリから事前に投入しているデプロイメントで使用してください。

```json5
{
  agents: { defaults: { skipBootstrap: true } }
}
```

<div id="agentsdefaultsbootstrapmaxchars">
  ### `agents.defaults.bootstrapMaxChars`
</div>

切り捨てが行われる前に、システムプロンプトへ注入される各ワークスペース用ブートストラップファイルの最大文字数です。デフォルト: `20000`。

ファイルがこの上限を超えた場合、OpenClaw は警告をログに記録し、先頭部分と末尾部分を切り詰めて、マーカー付きで注入します。

```json5
{
  agents: { defaults: { bootstrapMaxChars: 20000 } }
}
```

<div id="agentsdefaultsusertimezone">
  ### `agents.defaults.userTimezone`
</div>

**システムプロンプトのコンテキスト**用にユーザーのタイムゾーンを設定します（メッセージのエンベロープ内のタイムスタンプには使用されません）。未設定の場合、OpenClaw は実行時にホストのタイムゾーンを使用します。

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } }
}
```

<div id="agentsdefaultstimeformat">
  ### `agents.defaults.timeFormat`
</div>

システムプロンプトの「Current Date &amp; Time」セクションに表示される**時刻形式**を制御します。
デフォルト: `auto`（OS の設定に従う）。

```json5
{
  agents: { defaults: { timeFormat: "auto" } } // auto | 12 | 24
}
```

<div id="messages">
  ### `messages`
</div>

受信/送信メッセージのプレフィックスと、任意の ACK リアクションを制御します。
キューイング、セッション、ストリーミングのコンテキストについては、[Messages](/ja/concepts/messages) を参照してください。

```json5
{
  messages: {
    responsePrefix: "🦞", // または "auto"
    ackReaction: "👀",
    ackReactionScope: "group-mentions",
    removeAckAfterReply: false
  }
}
```

`responsePrefix` は、すでに付与されている場合を除き、チャネルを問わず **すべての送信側の返信**（ツール要約、ブロックストリーミング、最終返信）に適用されます。

`messages.responsePrefix` が未設定の場合、デフォルトではプレフィックスは適用されません。WhatsApp のセルフチャット（自分とのチャット）への返信のみ例外で、`[{identity.name}]` が設定されていればそれが、設定されていなければ `[openclaw]` がデフォルトとなり、同一端末内での会話の可読性を保ちます。
`"auto"` を指定すると、ルーティングされたエージェントに対して（設定されている場合） `[{identity.name}]` を自動的に導出して適用します。

<div id="template-variables">
  #### テンプレート変数
</div>

`responsePrefix` 文字列には、動的に解決されるテンプレート変数を含めることができます：

| Variable          | Description      | Example                     |
| ----------------- | ---------------- | --------------------------- |
| `{model}`         | 短いモデル名           | `claude-opus-4-5`, `gpt-4o` |
| `{modelFull}`     | フルモデル識別子         | `anthropic/claude-opus-4-5` |
| `{provider}`      | プロバイダー名          | `anthropic`, `openai`       |
| `{thinkingLevel}` | 現在の思考レベル         | `high`, `low`, `off`        |
| `{identity.name}` | エージェントのアイデンティティ名 | ( `"auto"` モードと同じ)          |

変数名は大文字小文字を区別しません（`{MODEL}` = `{model}`）。`{think}` は `{thinkingLevel}` のエイリアスです。
解決できなかった変数は、そのまま文字列として残ります。

```json5
{
  messages: {
    responsePrefix: "[{model} | think:{thinkingLevel}]"
  }
}
```

出力例: `[claude-opus-4-5 | think:high] Here's my response...`

WhatsApp の受信メッセージ用プレフィックスは `channels.whatsapp.messagePrefix`（非推奨:
`messages.messagePrefix`）で設定します。デフォルトは **変更されません**。`channels.whatsapp.allowFrom` が空の場合は `"[openclaw]"`、それ以外の場合は `""`（プレフィックスなし）です。`"[openclaw]"` を使用している場合、ルーティングされたエージェントに `identity.name` が設定されていれば、OpenClaw は代わりに `[{identity.name}]` を使用します。

`ackReaction` は、リアクションをサポートするチャネル（Slack/Discord/Telegram/Google Chat）上で、受信メッセージを確認するための絵文字リアクションをベストエフォートで送信します。デフォルトは、アクティブなエージェントの `identity.emoji` が設定されている場合はその値、未設定の場合は `"👀"` です。無効化するには `""` を設定します。

`ackReactionScope` は、リアクションを送信する条件を制御します:

* `group-mentions`（デフォルト）: グループ/ルームでメンションが必須 **かつ** ボットがメンションされた場合のみ
* `group-all`: すべてのグループ/ルームメッセージ
* `direct`: ダイレクトメッセージのみ
* `all`: すべてのメッセージ

`removeAckAfterReply` は、返信が送信された後にボットの確認リアクションを削除します
（Slack/Discord/Telegram/Google Chat のみ）。デフォルト: `false`。

<div id="messagestts">
  #### `messages.tts`
</div>

送信する返信メッセージのテキスト読み上げ（text-to-speech）を有効にします。有効にすると、OpenClaw は ElevenLabs または OpenAI を使用して音声を生成し、応答に添付します。Telegram では Opus のボイスメッセージが使用され、それ以外のチャネルでは MP3 音声が送信されます。

```json5
{
  messages: {
    tts: {
      auto: "always", // off | always | inbound | tagged
      mode: "final", // final | all (ツール/ブロック応答を含む)
      provider: "elevenlabs",
      summaryModel: "openai/gpt-4.1-mini",
      modelOverrides: {
        enabled: true
      },
      maxTextLength: 4000,
      timeoutMs: 30000,
      prefsPath: "~/.openclaw/settings/tts.json",
      elevenlabs: {
        apiKey: "elevenlabs_api_key",
        baseUrl: "https://api.elevenlabs.io",
        voiceId: "voice_id",
        modelId: "eleven_multilingual_v2",
        seed: 42,
        applyTextNormalization: "auto",
        languageCode: "en",
        voiceSettings: {
          stability: 0.5,
          similarityBoost: 0.75,
          style: 0.0,
          useSpeakerBoost: true,
          speed: 1.0
        }
      },
      openai: {
        apiKey: "openai_api_key",
        model: "gpt-4o-mini-tts",
        voice: "alloy"
      }
    }
  }
}
```

Notes:

* `messages.tts.auto` は自動 TTS を制御します（`off`、`always`、`inbound`、`tagged`）。
* `/tts off|always|inbound|tagged` はセッション単位の自動モードを設定します（設定を上書き）。
* `messages.tts.enabled` はレガシーです。`doctor` が `messages.tts.auto` へ移行します。
* `prefsPath` にはローカルのオーバーライド（プロバイダー／上限／要約設定）が保存されます。
* `maxTextLength` は TTS 入力に対するハード上限で、要約はこの長さに収まるよう切り詰められます。
* `summaryModel` は自動要約用に `agents.defaults.model.primary` を上書きします。
  * `provider/model` か、`agents.defaults.models` のエイリアスを受け付けます。
* `modelOverrides` は `[[tts:...]]` タグのようなモデル駆動オーバーライドを有効にします（デフォルトで有効）。
* `/tts limit` と `/tts summary` はユーザーごとの要約設定を制御します。
* `apiKey` の値は `ELEVENLABS_API_KEY`/`XI_API_KEY` および `OPENAI_API_KEY` へのフォールバックとして機能します。
* `elevenlabs.baseUrl` は ElevenLabs API のベース URL を上書きします。
* `elevenlabs.voiceSettings` は `stability`/`similarityBoost`/`style`（0..1）、
  `useSpeakerBoost`、および `speed`（0.5..2.0）をサポートします。

<div id="talk">
  ### `talk`
</div>

Talk モード（macOS/iOS/Android）向けのデフォルト値です。Voice ID が未設定の場合は、`ELEVENLABS_VOICE_ID` または `SAG_VOICE_ID` にフォールバックします。
`apiKey` が未設定の場合は、`ELEVENLABS_API_KEY`（または Gateway のシェルプロファイルに設定された値）にフォールバックします。
`voiceAliases` を使うと、Talk ディレクティブでわかりやすい名前（例: `"voice":"Clawd"`）を利用できます。

```json5
{
  talk: {
    voiceId: "elevenlabs_voice_id",
    voiceAliases: {
      Clawd: "EXAVITQu4vr4xnSDxMaL",
      Roger: "CwhRBWXzGAHq8TQ4Fs17"
    },
    modelId: "eleven_v3",
    outputFormat: "mp3_44100_128",
    apiKey: "elevenlabs_api_key",
    interruptOnSpeech: true
  }
}
```

<div id="agentsdefaults">
  ### `agents.defaults`
</div>

組み込みエージェントのランタイム（model/thinking/verbose/timeouts）の挙動を制御します。
`agents.defaults.models` は設定済みのモデルカタログを定義し、`/model` の許可リストとしても機能します。
`agents.defaults.model.primary` はデフォルトモデルを設定し、`agents.defaults.model.fallbacks` はグローバルなフェイルオーバー先を定義します。
`agents.defaults.imageModel` は任意指定で、**プライマリモデルに画像入力機能がない場合にのみ使用されます**。
各 `agents.defaults.models` エントリには、次の内容を含めることができます:

* `alias`（任意のモデルショートカット。例: `/opus`）。
* `params`（モデルリクエストにそのまま渡される、プロバイダー固有の任意の API パラメーター）。

`params` はストリーミング実行（組み込みエージェント + コンパクション）にも適用されます。現在サポートされているキーは `temperature`、`maxTokens` です。これらは呼び出し時オプションとマージされ、呼び出し元で指定された値が優先されます。`temperature` は高度な調整用パラメーターです。モデルのデフォルト値を把握していて、明確に変更が必要な場合以外は設定しないでください。

例:

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-sonnet-4-5-20250929": {
          params: { temperature: 0.6 }
        },
        "openai/gpt-5.2": {
          params: { maxTokens: 8192 }
        }
      }
    }
  }
}
```

Z.AI GLM-4.x モデルは、次のいずれかを行わない限り、自動的に thinking モードが有効になります：

* `--thinking off` を指定する
* 自分で `agents.defaults.models["zai/<model>"].params.thinking` を定義する

OpenClaw には、いくつかの組み込みエイリアス略記も用意されています。デフォルトは、そのモデルがすでに
`agents.defaults.models` に存在している場合にのみ適用されます:

* `opus` -&gt; `anthropic/claude-opus-4-5`
* `sonnet` -&gt; `anthropic/claude-sonnet-4-5`
* `gpt` -&gt; `openai/gpt-5.2`
* `gpt-mini` -&gt; `openai/gpt-5-mini`
* `gemini` -&gt; `google/gemini-3-pro-preview`
* `gemini-flash` -&gt; `google/gemini-3-flash-preview`

同じエイリアス名（大文字小文字は無視）を自分で設定した場合は、その設定値が優先されます（デフォルトが上書きされることはありません）。

例: Opus 4.5 をプライマリ、MiniMax M2.1 をフォールバックとして利用する構成（ホスト型 MiniMax）:

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-5": { alias: "opus" },
        "minimax/MiniMax-M2.1": { alias: "minimax" }
      },
      model: {
        primary: "anthropic/claude-opus-4-5",
        fallbacks: ["minimax/MiniMax-M2.1"]
      }
    }
  }
}
```

MiniMax の認証: `MINIMAX_API_KEY`（環境変数）を設定するか、`models.providers.minimax` を構成します。

<div id="agentsdefaultsclibackends-cli-fallback">
  #### `agents.defaults.cliBackends` (CLI フォールバック)
</div>

テキストのみのフォールバック実行（ツール呼び出しなし）のためのオプションの CLI バックエンドです。これは API プロバイダーが失敗した場合のバックアップ経路として有用です。ファイルパスを受け取る `imageArg` を設定すれば、画像のパススルーもサポートされます。

注意:

* CLI バックエンドは **テキスト優先** であり、ツールは常に無効化されます。
* `sessionArg` が設定されている場合、セッションがサポートされます。セッション ID はバックエンドごとに永続化されます。
* `claude-cli` の場合、デフォルトは組み込み済みです。PATH が最小構成
  （launchd/systemd）の場合は、コマンドパスを上書きしてください。

例:

```json5
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude"
        },
        "my-cli": {
          command: "my-cli",
          args: ["--json"],
          output: "json",
          modelArg: "--model",
          sessionArg: "--session",
          sessionMode: "existing",
          systemPromptArg: "--system",
          systemPromptWhen: "first",
          imageArg: "--image",
          imageMode: "repeat"
        }
      }
    }
  }
}
```

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-5": { alias: "Opus" },
        "anthropic/claude-sonnet-4-1": { alias: "Sonnet" },
        "openrouter/deepseek/deepseek-r1:free": {},
        "zai/glm-4.7": {
          alias: "GLM",
          params: {
            thinking: {
              type: "enabled",
              clear_thinking: false
            }
          }
        }
      },
      model: {
        primary: "anthropic/claude-opus-4-5",
        fallbacks: [
          "openrouter/deepseek/deepseek-r1:free",
          "openrouter/meta-llama/llama-3.3-70b-instruct:free"
        ]
      },
      imageModel: {
        primary: "openrouter/qwen/qwen-2.5-vl-72b-instruct:free",
        fallbacks: [
          "openrouter/google/gemini-2.0-flash-vision:free"
        ]
      },
      thinkingDefault: "low",
      verboseDefault: "off",
      elevatedDefault: "on",
      timeoutSeconds: 600,
      mediaMaxMb: 5,
      heartbeat: {
        every: "30m",
        target: "last"
      },
      maxConcurrent: 3,
      subagents: {
        model: "minimax/MiniMax-M2.1",
        maxConcurrent: 1,
        archiveAfterMinutes: 60
      },
      exec: {
        backgroundMs: 10000,
        timeoutSec: 1800,
        cleanupMs: 1800000
      },
      contextTokens: 200000
    }
  }
}
```

<div id="agentsdefaultscontextpruning-tool-result-pruning">
  #### `agents.defaults.contextPruning` (ツール結果のプルーニング)
</div>

`agents.defaults.contextPruning` は、リクエストが LLM に送信される直前に、インメモリのコンテキストから **古いツール結果** を削除します。
ディスク上のセッション履歴は **一切変更されません**（`*.jsonl` は完全なまま保持されます）。

これは、時間の経過とともに大きなツール出力を大量に蓄積しがちな「おしゃべりな」エージェントのトークン使用量を削減することを目的としています。

概要:

* ユーザー／アシスタントのメッセージには一切触れません。
* 直近の `keepLastAssistants` 個のアシスタントメッセージを保護します（その時点以降のツール結果はプルーニングされません）。
* ブートストラップ用プレフィックスを保護します（最初のユーザーメッセージより前は何もプルーニングされません）。
* モード:
  * `adaptive`: 推定コンテキスト占有率が `softTrimRatio` を超えたときに、サイズ超過したツール結果をソフトトリムします（先頭と末尾を保持）。
    その後、推定コンテキスト占有率が `hardClearRatio` を超え、かつプルーニング可能なツール結果の塊が十分にある場合（`minPrunableToolChars`）、
    最も古いプルーニング対象のツール結果をハードクリアします。
  * `aggressive`: 常に、カットオフより前のプルーニング対象ツール結果を `hardClear.placeholder` に置き換えます（比率チェックなし）。

ソフトプルーニングとハードプルーニング（LLM に送られるコンテキストで何が変わるか）:

* **ソフトトリム**: *サイズ超過* したツール結果にのみ適用されます。先頭と末尾を残し、中間に `...` を挿入します。
  * Before: `toolResult("…very long output…")`
  * After: `toolResult("HEAD…\n...\n…TAIL\n\n[Tool result trimmed: …]")`
* **ハードクリア**: ツール結果全体をプレースホルダーで置き換えます。
  * Before: `toolResult("…very long output…")`
  * After: `toolResult("[Old tool result content cleared]")`

補足 / 現時点での制限事項:

* **画像ブロックを含むツール結果はスキップ** されます（現時点ではトリム／クリアされません）。
* 推定される「コンテキスト占有率」は、厳密なトークン数ではなく、**文字数**（おおよその値）に基づいています。
* セッションにまだ `keepLastAssistants` 個以上のアシスタントメッセージが含まれていない場合、プルーニングはスキップされます。
* `aggressive` モードでは、`hardClear.enabled` は無視されます（プルーニング対象のツール結果は常に `hardClear.placeholder` に置き換えられます）。

デフォルト（adaptive）:

```json5
{
  agents: { defaults: { contextPruning: { mode: "adaptive" } } }
}
```

無効にするには：

```json5
{
  agents: { defaults: { contextPruning: { mode: "off" } } }
}
```

デフォルト設定（`mode` が `"adaptive"` または `"aggressive"` の場合）:

* `keepLastAssistants`: `3`
* `softTrimRatio`: `0.3`（adaptive のみ）
* `hardClearRatio`: `0.5`（adaptive のみ）
* `minPrunableToolChars`: `50000`（adaptive のみ）
* `softTrim`: `{ maxChars: 4000, headChars: 1500, tailChars: 1500 }`（adaptive のみ）
* `hardClear`: `{ enabled: true, placeholder: "[Old tool result content cleared]" }`

例（aggressive、最小構成）:

```json5
{
  agents: { defaults: { contextPruning: { mode: "aggressive" } } }
}
```

例（アダプティブチューニング）:

```json5
{
  agents: {
    defaults: {
      contextPruning: {
        mode: "adaptive",
        keepLastAssistants: 3,
        softTrimRatio: 0.3,
        hardClearRatio: 0.5,
        minPrunableToolChars: 50000,
        softTrim: { maxChars: 4000, headChars: 1500, tailChars: 1500 },
        hardClear: { enabled: true, placeholder: "[Old tool result content cleared]" },
        // オプション: 特定のツールに対してプルーニングを制限 (deny が優先; "*" ワイルドカードに対応)
        tools: { deny: ["browser", "canvas"] },
      }
    }
  }
}
```

挙動の詳細については [/concepts/session-pruning](/ja/concepts/session-pruning) を参照してください。

<div id="agentsdefaultscompaction-reserve-headroom-memory-flush">
  #### `agents.defaults.compaction`（ヘッドルームの確保 + メモリフラッシュ）
</div>

`agents.defaults.compaction.mode` はコンパクションの要約戦略を選択します。デフォルトは `default` です。`safeguard` を設定すると、非常に長い履歴に対してチャンク分割した要約が有効になります。[/concepts/compaction](/ja/concepts/compaction) を参照してください。

`agents.defaults.compaction.reserveTokensFloor` は、Pi コンパクションにおける最小 `reserveTokens` 値を強制します（デフォルト: `20000`）。この下限を無効化するには `0` を設定します。

`agents.defaults.compaction.memoryFlush` は、自動コンパクションの前に **サイレント** なエージェントターンを 1 回実行し、
モデルに対して永続メモリをディスク上へ保存するよう指示します（例:
`memory/YYYY-MM-DD.md`）。これは、セッションのトークン数の推定値がコンパクション上限より少し低いソフトしきい値を超えたときにトリガーされます。

レガシーなデフォルト設定:

* `memoryFlush.enabled`: `true`
* `memoryFlush.softThresholdTokens`: `4000`
* `memoryFlush.prompt` / `memoryFlush.systemPrompt`: `NO_REPLY` を含む組み込みデフォルト
* 注意: セッションのワークスペースが読み取り専用の場合はメモリフラッシュはスキップされます
  (`agents.defaults.sandbox.workspaceAccess: "ro"` または `"none"`)。

例（調整済み）:

```json5
{
  agents: {
    defaults: {
      compaction: {
        mode: "safeguard",
        reserveTokensFloor: 24000,
        memoryFlush: {
          enabled: true,
          softThresholdTokens: 6000,
          systemPrompt: "セッションが圧縮間近です。永続的なメモリを今すぐ保存してください。",
          prompt: "永続的なメモがあればmemory/YYYY-MM-DD.mdに書き込んでください。保存するものがない場合はNO_REPLYで返信してください。"
        }
      }
    }
  }
}
```

ブロックストリーミング:

* `agents.defaults.blockStreamingDefault`: `"on"`/`"off"`（デフォルトは off）。
* チャンネルごとのオーバーライド: `*.blockStreaming`（およびアカウント単位のバリアント）でブロックストリーミングのオン/オフを強制。
  Telegram 以外のチャンネルでは、ブロック返信を有効化するには明示的に `*.blockStreaming: true` が必要。
* `agents.defaults.blockStreamingBreak`: `"text_end"` または `"message_end"`（デフォルト: text&#95;end）。
* `agents.defaults.blockStreamingChunk`: ストリーミングされたブロックのソフトチャンク分割。デフォルトは
  800–1200 文字で、段落区切り（`\n\n`）を最優先し、その次に改行、その次に文区切りを優先。
  例:
  ```json5
  {
    agents: { defaults: { blockStreamingChunk: { minChars: 800, maxChars: 1200 } } }
  }
  ```
* `agents.defaults.blockStreamingCoalesce`: 送信前にストリーミングブロックをマージ。
  デフォルトは `{ idleMs: 1000 }` で、`blockStreamingChunk` から `minChars` を継承し、
  `maxChars` はチャンネルのテキスト上限に合わせて制限される。Signal/Slack/Discord/Google Chat のデフォルトは、
  上書きされない限り `minChars: 1500`。
  チャンネルごとのオーバーライド: `channels.whatsapp.blockStreamingCoalesce`, `channels.telegram.blockStreamingCoalesce`,
  `channels.discord.blockStreamingCoalesce`, `channels.slack.blockStreamingCoalesce`, `channels.mattermost.blockStreamingCoalesce`,
  `channels.signal.blockStreamingCoalesce`, `channels.imessage.blockStreamingCoalesce`, `channels.msteams.blockStreamingCoalesce`,
  `channels.googlechat.blockStreamingCoalesce`
  （およびアカウント単位のバリアント）。
* `agents.defaults.humanDelay`: 最初のブロック返信の後に続く **ブロック返信** の間に挟むランダムなポーズ。
  モード: `off`（デフォルト）、`natural`（800–2500ms）、`custom`（`minMs`/`maxMs` を使用）。
  エージェント単位のオーバーライド: `agents.list[].humanDelay`。
  例:
  ```json5
  {
    agents: { defaults: { humanDelay: { mode: "natural" } } }
  }
  ```

動作とチャンク分割の詳細は [/concepts/streaming](/ja/concepts/streaming) を参照。

タイピングインジケーター:

* `agents.defaults.typingMode`: `"never" | "instant" | "thinking" | "message"`。デフォルトは、
  ダイレクトチャット / メンションでは `instant`、メンションされていないグループチャットでは `message`。
* `session.typingMode`: モードに対するセッション単位のオーバーライド。
* `agents.defaults.typingIntervalSeconds`: タイピングシグナルをどの頻度で更新するか（デフォルト: 6 秒）。
* `session.typingIntervalSeconds`: 更新間隔に対するセッション単位のオーバーライド。
  動作の詳細は [/concepts/typing-indicators](/ja/concepts/typing-indicators) を参照。

`agents.defaults.model.primary` は `provider/model` 形式で設定する必要があります（例: `anthropic/claude-opus-4-5`）。
エイリアスは `agents.defaults.models.*.alias` から取得されます（例: `Opus`）。
provider を省略した場合、OpenClaw は現在、暫定的な非推奨フォールバックとして `anthropic` を仮定します。
Z.AI モデルは `zai/<model>`（例: `zai/glm-4.7`）として利用可能で、環境変数に
`ZAI_API_KEY`（もしくはレガシーな `Z_AI_API_KEY`）が必要です。

`agents.defaults.heartbeat` は定期的なハートビート実行を構成します:

* `every`: 期間文字列（`ms`, `s`, `m`, `h`）。デフォルトの単位は分。デフォルト値:
  `30m`。`0m` を設定すると無効化されます。
* `model`: ハートビート実行用の任意の上書きモデル（`provider/model`）。
* `includeReasoning`: `true` の場合、利用可能であれば、ハートビートは `/reasoning on` と同じ形式の別個の `Reasoning:` メッセージも配信します。デフォルト: `false`。
* `session`: ハートビートをどのセッションで実行するかを制御する任意のセッションキー。デフォルト: `main`。
* `to`: 任意の受信者上書き（チャネル固有の ID。例: WhatsApp 用の E.164、Telegram の chat id）。
* `target`: 任意の配信チャネル上書き（`last`, `whatsapp`, `telegram`, `discord`, `slack`, `msteams`, `signal`, `imessage`, `none`）。デフォルト: `last`。
* `prompt`: ハートビート本文用の任意の上書き（デフォルト: `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`）。上書きはそのまま送信されます。ファイルの読み取りを継続したい場合は、`Read HEARTBEAT.md` 行を含めてください。
* `ackMaxChars`: 配信前に `HEARTBEAT_OK` の後に許可される最大文字数（デフォルト: 300）。

エージェント単位のハートビート:

* 特定のエージェント用にハートビート設定を有効化または上書きするには、`agents.list[].heartbeat` を設定します。
* いずれかのエージェントエントリで `heartbeat` が定義されている場合、**そのエージェントのみが** ハートビートを実行します。デフォルト値はそれらのエージェントの共有ベースラインになります。

ハートビートはエージェントのフルターンを実行します。間隔を短くするとトークン消費が増えるため、`every` に注意し、`HEARTBEAT.md` を小さく保つ、あるいはより安価な `model` を選択してください。

`tools.exec` はバックグラウンド実行のデフォルトを構成します:

* `backgroundMs`: 自動でバックグラウンド化されるまでの時間（ms、デフォルト 10000）
* `timeoutSec`: この実行時間を超えた場合の自動 kill（秒、デフォルト 1800）
* `cleanupMs`: 完了したセッションをメモリ内に保持しておく時間（ms、デフォルト 1800000）
* `notifyOnExit`: バックグラウンド実行が終了したときにシステムイベントをキューし、ハートビートを要求する（デフォルト true）
* `applyPatch.enabled`: 実験的な `apply_patch` を有効化（OpenAI/OpenAI Codex のみ。デフォルト false）
* `applyPatch.allowModels`: 任意のモデル ID の許可リスト（例: `gpt-5.2` や `openai/gpt-5.2`）
  注: `applyPatch` は `tools.exec` の配下にのみ存在します。

`tools.web` は Web 検索および fetch ツールを構成します:

* `tools.web.search.enabled`（デフォルト: キーが存在する場合 true）
* `tools.web.search.apiKey`（推奨: `openclaw configure --section web` で設定するか、`BRAVE_API_KEY` 環境変数を使用）
* `tools.web.search.maxResults`（1～10、デフォルト 5）
* `tools.web.search.timeoutSeconds`（デフォルト 30）
* `tools.web.search.cacheTtlMinutes`（デフォルト 15）
* `tools.web.fetch.enabled`（デフォルト true）
* `tools.web.fetch.maxChars`（デフォルト 50000）
* `tools.web.fetch.timeoutSeconds`（デフォルト 30）
* `tools.web.fetch.cacheTtlMinutes`（デフォルト 15）
* `tools.web.fetch.userAgent`（任意の上書き）
* `tools.web.fetch.readability`（デフォルト true。無効化すると基本的な HTML クリーンアップのみを使用）
* `tools.web.fetch.firecrawl.enabled`（API キーが設定されている場合のデフォルトは true）
* `tools.web.fetch.firecrawl.apiKey`（任意。デフォルトは `FIRECRAWL_API_KEY`）
* `tools.web.fetch.firecrawl.baseUrl`（デフォルト https://api.firecrawl.dev）
* `tools.web.fetch.firecrawl.onlyMainContent`（デフォルト true）
* `tools.web.fetch.firecrawl.maxAgeMs`（任意）
* `tools.web.fetch.firecrawl.timeoutSeconds`（任意）

`tools.media` は受信メディア（画像/音声/動画）の理解を構成します:

* `tools.media.models`: 共有モデルリスト（機能タグ付き。個別の機能別リストの後段として使用される）。
* `tools.media.concurrency`: 同時に実行できる最大機能数（デフォルト 2）。
* `tools.media.image` / `tools.media.audio` / `tools.media.video`:
  * `enabled`: オプトアウト用スイッチ（モデルが設定されている場合のデフォルトは true）。
  * `prompt`: 任意のプロンプト上書き（image/video では `maxChars` ヒントが自動で付与される）。
  * `maxChars`: 出力文字数の上限（image/video のデフォルトは 500。audio では未設定）。
  * `maxBytes`: 送信するメディアサイズの上限（デフォルト: image 10MB、audio 20MB、video 50MB）。
  * `timeoutSeconds`: リクエストのタイムアウト（デフォルト: image 60s、audio 60s、video 120s）。
  * `language`: 任意の audio 用ヒント。
  * `attachments`: 添付ファイルポリシー（`mode`、`maxAttachments`、`prefer`）。
  * `scope`: 任意のゲーティング（先勝ち）で、`match.channel`、`match.chatType`、`match.keyPrefix` を使用可能。
  * `models`: モデルエントリの順序付きリスト。失敗やサイズ超過のメディアは次のエントリにフォールバックする。
* 各 `models[]` エントリ:
  * プロバイダーエントリ（`type: "provider"` または省略時）:
    * `provider`: API プロバイダー ID（`openai`、`anthropic`、`google`/`gemini`、`groq` など）。
    * `model`: モデル ID の上書き（image では必須。audio プロバイダーでは `gpt-4o-mini-transcribe`/`whisper-large-v3-turbo` がデフォルト、video では `gemini-3-flash-preview` がデフォルト）。
    * `profile` / `preferredProfile`: 認証プロファイルの選択。
  * CLI エントリ（`type: "cli"`）:
    * `command`: 実行する実行可能ファイル。
    * `args`: テンプレート引数（`{{MediaPath}}`、`{{Prompt}}`、`{{MaxChars}}` などをサポート）。
  * `capabilities`: 任意のリスト（`image`、`audio`、`video`）で共有エントリの対象機能を制限する。省略時のデフォルト: `openai`/`anthropic`/`minimax` → image、`google` → image+audio+video、`groq` → audio。
  * `prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language` はエントリごとに上書き可能。

モデルが一切設定されていない場合（または `enabled: false` の場合）、メディア理解処理はスキップされるが、モデルには元の添付ファイルがそのまま渡される。

プロバイダーの認証は標準的なモデル認証の順序に従う（認証プロファイル、`OPENAI_API_KEY`/`GROQ_API_KEY`/`GEMINI_API_KEY` のような環境変数、または `models.providers.*.apiKey`）。

例:

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        maxBytes: 20971520,
        scope: {
          default: "deny",
          rules: [{ action: "allow", match: { chatType: "direct" } }]
        },
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          { type: "cli", command: "whisper", args: ["--model", "base", "{{MediaPath}}"] }
        ]
      },
      video: {
        enabled: true,
        maxBytes: 52428800,
        models: [{ provider: "google", model: "gemini-3-flash-preview" }]
      }
    }
  }
}
```

`agents.defaults.subagents` はサブエージェントのデフォルト値を設定します:

* `model`: 生成されるサブエージェントのデフォルトモデル（文字列または `{ primary, fallbacks }`）。省略された場合、サブエージェントは、エージェント単位または呼び出し単位で上書きされない限り、呼び出し元のモデルを継承します。
* `maxConcurrent`: サブエージェントの同時実行数の上限（デフォルト 1）
* `archiveAfterMinutes`: N 分後にサブエージェントのセッションを自動アーカイブする（デフォルト 60。無効化するには `0` を設定）
* サブエージェントごとのツールポリシー: `tools.subagents.tools.allow` / `tools.subagents.tools.deny`（deny が優先）

`tools.profile` は、`tools.allow` / `tools.deny` の前に適用される **ベースのツール許可リスト**を設定します:

* `minimal`: `session_status` のみ
* `coding`: `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`
* `messaging`: `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status`
* `full`: 制限なし（未設定と同じ）

エージェント単位での上書き: `agents.list[].tools.profile`.

例（デフォルトではメッセージングのみだが、Slack と Discord のツールも許可する）:

```json5
{
  tools: {
    profile: "messaging",
    allow: ["slack", "discord"]
  }
}
```

例（コーディング向けプロファイルだが、exec/process は全域で禁止）：

```json5
{
  tools: {
    profile: "coding",
    deny: ["group:runtime"]
  }
}
```

`tools.byProvider` を使うと、特定のプロバイダー（または単一の `provider/model`）向けにツールを**さらに制限**できます。
エージェント単位の上書き: `agents.list[].tools.byProvider`。

適用順序: ベースプロファイル → プロバイダーごとのプロファイル → allow/deny ポリシー。
プロバイダーキーには `provider`（例: `google-antigravity`）または `provider/model`
（例: `openai/gpt-5.2`）を指定できます。

例（グローバルのコーディングプロファイルは維持しつつ、Google Antigravity には最小限のツールのみを許可する場合）:

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

例（プロバイダー／モデル固有の許可リストの例）：

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

`tools.allow` / `tools.deny` はグローバルなツール許可/拒否ポリシーを構成します（拒否側が優先されます）。
マッチングは大文字小文字を区別せず、`*` ワイルドカードをサポートします（`"*"` はすべてのツールを意味します）。
これは Docker サンドボックスが **off** のときでも有効です。

例（全体で browser/canvas を無効化）:

```json5
{
  tools: { deny: ["browser", "canvas"] }
}
```

ツールグループ（省略形）は、**グローバル**および**各エージェント**のツールポリシーで使用できます:

* `group:runtime`: `exec`, `bash`, `process`
* `group:fs`: `read`, `write`, `edit`, `apply_patch`
* `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
* `group:memory`: `memory_search`, `memory_get`
* `group:web`: `web_search`, `web_fetch`
* `group:ui`: `browser`, `canvas`
* `group:automation`: `cron`, `gateway`
* `group:messaging`: `message`
* `group:nodes`: `nodes`
* `group:openclaw`: すべての組み込み OpenClaw ツール（プロバイダーのプラグインは除く）

`tools.elevated` は昇格（ホスト）exec アクセスを制御します:

* `enabled`: 昇格モードを許可する（デフォルト true）
* `allowFrom`: チャネルごとの許可リスト（空の場合は無効）
  * `whatsapp`: E.164 番号
  * `telegram`: チャット ID またはユーザー名
  * `discord`: ユーザー ID またはユーザー名（省略時は `channels.discord.dm.allowFrom` が使用される）
  * `signal`: E.164 番号
  * `imessage`: ハンドル / チャット ID
  * `webchat`: セッション ID またはユーザー名

例:

```json5
{
  tools: {
    elevated: {
      enabled: true,
      allowFrom: {
        whatsapp: ["+15555550123"],
        discord: ["steipete", "1234567890123"]
      }
    }
  }
}
```

エージェントごとの上書き設定（さらに制限する）:

```json5
{
  agents: {
    list: [
      {
        id: "family",
        tools: {
          elevated: { enabled: false }
        }
      }
    ]
  }
}
```

注意:

* `tools.elevated` はグローバルなベースラインです。`agents.list[].tools.elevated` はそれ以上に制限を強めることしかできません（両方で許可されている必要があります）。
* `/elevated on|off|ask|full` はセッションキーごとに状態を保存します。インラインのディレクティブは単一メッセージのみに適用されます。
* 昇格モードの `exec` はホスト上で実行され、サンドボックス化をバイパスします。
* ツールポリシーは引き続き適用されます。`exec` が拒否されている場合、昇格は使用できません。

`agents.defaults.maxConcurrent` は、セッションをまたいで並列に実行できる埋め込みエージェントの実行数の上限を設定します。各セッション内では依然として直列実行です（同時に 1 セッションキーあたり 1 実行）。デフォルト: 1。

<div id="agentsdefaultssandbox">
  ### `agents.defaults.sandbox`
</div>

組み込みエージェント向けの **オプションの Docker サンドボックス化** 設定です。非メインセッションでの利用を想定しており、ホストシステムへアクセスできないようにします。

詳細: [Sandboxing](/ja/gateway/sandboxing)

(有効化した場合の)デフォルト:

* scope: `"agent"` (エージェントごとに 1 コンテナ + 1 ワークスペース)
* Debian bookworm-slim ベースのイメージ
* エージェントのワークスペースアクセス: `workspaceAccess: "none"` (デフォルト)
  * `"none"`: `~/.openclaw/sandboxes` 以下にスコープごとのサンドボックス用ワークスペースを使用
* `"ro"`: サンドボックス用ワークスペースを `/workspace` に保持しつつ、エージェントのワークスペースを `/agent` に read-only でマウント (`write` / `edit` / `apply_patch` を無効化)
  * `"rw"`: エージェントのワークスペースを `/workspace` に read/write でマウント
* 自動削除: アイドル時間 &gt; 24 時間 または 経過時間 &gt; 7 日
* ツールポリシー: `exec`, `process`, `read`, `write`, `edit`, `apply_patch`, `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status` のみ許可 (deny が優先)
  * `tools.sandbox.tools` で設定し、エージェントごとの上書きは `agents.list[].tools.sandbox.tools` で行う
  * サンドボックスポリシーではツールグループの省略記法をサポート: `group:runtime`, `group:fs`, `group:sessions`, `group:memory` (詳細は [Sandbox vs Tool Policy vs Elevated](/ja/gateway/sandbox-vs-tool-policy-vs-elevated#tool-groups-shorthands) を参照)
* オプションのサンドボックス化ブラウザ (Chromium + CDP, noVNC observer)
* ハードニング用の調整項目: `network`, `user`, `pidsLimit`, `memory`, `cpus`, `ulimits`, `seccompProfile`, `apparmorProfile`

警告: `scope: "shared"` はコンテナとワークスペースが共有されることを意味します。セッション間の分離はありません。セッション単位の分離には `scope: "session"` を使用してください。

レガシー: `perSession` も引き続きサポートされています (`true` → `scope: "session"`,
`false` → `scope: "shared"`).

`setupCommand` はコンテナ作成後に **1 回だけ** 実行されます (コンテナ内で `sh -lc` 経由で実行)。パッケージをインストールする場合は、外向き (egress) ネットワークアクセス、書き込み可能なルート FS、および root ユーザであることを確認してください。

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // off | non-main | all
        scope: "agent", // session | agent | shared (agent is default)
        workspaceAccess: "none", // none | ro | rw
        workspaceRoot: "~/.openclaw/sandboxes",
        docker: {
          image: "openclaw-sandbox:bookworm-slim",
          containerPrefix: "openclaw-sbx-",
          workdir: "/workspace",
          readOnlyRoot: true,
          tmpfs: ["/tmp", "/var/tmp", "/run"],
          network: "none",
          user: "1000:1000",
          capDrop: ["ALL"],
          env: { LANG: "C.UTF-8" },
          setupCommand: "apt-get update && apt-get install -y git curl jq",
          // エージェント単位での上書き（マルチエージェント）: agents.list[].sandbox.docker.*
          pidsLimit: 256,
          memory: "1g",
          memorySwap: "2g",
          cpus: 1,
          ulimits: {
            nofile: { soft: 1024, hard: 2048 },
            nproc: 256
          },
          seccompProfile: "/path/to/seccomp.json",
          apparmorProfile: "openclaw-sandbox",
          dns: ["1.1.1.1", "8.8.8.8"],
          extraHosts: ["internal.service:10.0.0.5"],
          binds: ["/var/run/docker.sock:/var/run/docker.sock", "/home/user/source:/source:rw"]
        },
        browser: {
          enabled: false,
          image: "openclaw-sandbox-browser:bookworm-slim",
          containerPrefix: "openclaw-sbx-browser-",
          cdpPort: 9222,
          vncPort: 5900,
          noVncPort: 6080,
          headless: false,
          enableNoVnc: true,
          allowHostControl: false,
          allowedControlUrls: ["http://10.0.0.42:18791"],
          allowedControlHosts: ["browser.lab.local", "10.0.0.42"],
          allowedControlPorts: [18791],
          autoStart: true,
          autoStartTimeoutMs: 12000
        },
        prune: {
          idleHours: 24,  // 0 disables idle pruning
          maxAgeDays: 7   // 0 disables max-age pruning
        }
      }
    }
  },
  tools: {
    sandbox: {
      tools: {
        allow: ["exec", "process", "read", "write", "edit", "apply_patch", "sessions_list", "sessions_history", "sessions_send", "sessions_spawn", "session_status"],
        deny: ["browser", "canvas", "nodes", "cron", "discord", "gateway"]
      }
    }
  }
}
```

次のコマンドでデフォルトのサンドボックスイメージを一度だけビルドします:

```bash
scripts/sandbox-setup.sh
```

注意: サンドボックスコンテナのデフォルトは `network: "none"` です。エージェントが外部へのネットワークアクセスを必要とする場合は、`agents.defaults.sandbox.docker.network` を `"bridge"`（または任意のカスタムネットワーク）に設定してください。

注意: 受信添付ファイルは、アクティブなワークスペース内の `media/inbound/*` に配置されます。`workspaceAccess: "rw"` の場合、これはファイルがエージェントのワークスペースに書き込まれることを意味します。

注意: `docker.binds` は追加のホストディレクトリをマウントします。グローバル設定とエージェントごとのバインド設定はマージされます。

オプションのブラウザイメージを次のコマンドでビルドします:

```bash
scripts/sandbox-browser-setup.sh
```

`agents.defaults.sandbox.browser.enabled=true` の場合、browser ツールはサンドボックス化された
Chromium インスタンス（CDP）を使用します。noVNC が有効な場合（headless=false のときデフォルト）、
noVNC の URL がシステムプロンプトに注入され、エージェントがそれを参照できるようになります。
これはメイン設定で `browser.enabled` を有効にする必要はありません。サンドボックス制御用
URL はセッションごとに注入されます。

`agents.defaults.sandbox.browser.allowHostControl`（デフォルト: false）を有効にすると、
サンドボックス化されたセッションが browser ツール（`target: "host"`）を通じて
**ホスト** のブラウザー制御サーバーを明示的にターゲットできるようになります。
厳格なサンドボックス分離を維持したい場合は、これはオフのままにしておいてください。

リモート制御用の許可リスト:

* `allowedControlUrls`: `target: "custom"` で許可される制御 URL（完全一致）。
* `allowedControlHosts`: 許可されるホスト名（ホスト名のみ、ポート指定なし）。
* `allowedControlPorts`: 許可されるポート（デフォルト: http=80, https=443）。
  デフォルトでは、すべての許可リストは未設定（制限なし）です。`allowHostControl` のデフォルトは false です。

<div id="models-custom-providers-base-urls">
  ### `models`（カスタムプロバイダー + ベース URL）
</div>

OpenClaw は **pi-coding-agent** モデルカタログを使用します。カスタムプロバイダー
（LiteLLM、ローカルの OpenAI 互換サーバー、Anthropic プロキシなど）を追加するには、
`~/.openclaw/agents/<agentId>/agent/models.json` を作成するか、
OpenClaw の設定ファイル内で同じスキーマを `models.providers` 以下に定義します。
プロバイダーごとの概要と例はこちら: [/concepts/model-providers](/ja/concepts/model-providers)。

`models.providers` が存在する場合、OpenClaw は起動時に
`~/.openclaw/agents/<agentId>/agent/` に `models.json` を書き出す／マージします:

* デフォルト動作: **merge**（既存のプロバイダーを保持しつつ、同名のものを上書き）
* ファイル内容を完全に上書きするには `models.mode: "replace"` を設定します

モデルは `agents.defaults.model.primary`（provider/model）で選択します。

```json5
{
  agents: {
    defaults: {
      model: { primary: "custom-proxy/llama-3.1-8b" },
      models: {
        "custom-proxy/llama-3.1-8b": {}
      }
    }
  },
  models: {
    mode: "merge",
    providers: {
      "custom-proxy": {
        baseUrl: "http://localhost:4000/v1",
        apiKey: "LITELLM_KEY",
        api: "openai-completions",
        models: [
          {
            id: "llama-3.1-8b",
            name: "Llama 3.1 8B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 128000,
            maxTokens: 32000
          }
        ]
      }
    }
  }
}
```

<div id="opencode-zen-multi-model-proxy">
  ### OpenCode Zen（マルチモデルプロキシ）
</div>

OpenCode Zen は、モデルごとにエンドポイントを持つマルチモデル対応の Gateway です。OpenClaw は
pi-ai のビルトイン `opencode` プロバイダーを使用します。https://opencode.ai/auth で
`OPENCODE_API_KEY`（または `OPENCODE_ZEN_API_KEY`）を設定してください。

補足事項:

* モデル参照には `opencode/<modelId>` を使います（例: `opencode/claude-opus-4-5`）。
* `agents.defaults.models` 経由で許可リストを有効にする場合は、使用予定の各モデルをすべて追加してください。
* ショートカット: `openclaw onboard --auth-choice opencode-zen`。

```json5
{
  agents: {
    defaults: {
      model: { primary: "opencode/claude-opus-4-5" },
      models: { "opencode/claude-opus-4-5": { alias: "Opus" } }
    }
  }
}
```

<div id="zai-glm-47-provider-alias-support">
  ### Z.AI (GLM-4.7) — プロバイダーエイリアスのサポート
</div>

Z.AI モデルは、組み込みの `zai` プロバイダー経由で利用できます。環境変数 `ZAI_API_KEY` を設定し、provider/model 形式でモデルを指定してください。

ショートカット: `openclaw onboard --auth-choice zai-api-key`。

```json5
{
  agents: {
    defaults: {
      model: { primary: "zai/glm-4.7" },
      models: { "zai/glm-4.7": {} }
    }
  }
}
```

Notes:

* `z.ai/*` と `z-ai/*` はエイリアスとして受け付けられ、`zai/*` に正規化されます。
* `ZAI_API_KEY` が存在しない場合、`zai/*` へのリクエストは実行時に認証エラーで失敗します。
* エラー例: `No API key found for provider "zai".`
* Z.AI の汎用 API エンドポイントは `https://api.z.ai/api/paas/v4` です。GLM 向けコーディング
  リクエストは専用の Coding エンドポイント `https://api.z.ai/api/coding/paas/v4` を使用します。
  組み込みの `zai` プロバイダーはこの Coding エンドポイントを使用します。汎用エンドポイントが必要な場合は、
  `models.providers` にベース URL のオーバーライド付きでカスタムプロバイダーを定義してください
  （上記のカスタムプロバイダーのセクションを参照）。
* ドキュメントや設定ファイルではダミーのプレースホルダー値を使用し、実際の API キーを決してコミットしないでください。

<div id="moonshot-ai-kimi">
  ### Moonshot AI (Kimi)
</div>

Moonshot の OpenAI 互換エンドポイントを使用します:

```json5
{
  env: { MOONSHOT_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "moonshot/kimi-k2.5" },
      models: { "moonshot/kimi-k2.5": { alias: "Kimi K2.5" } }
    }
  },
  models: {
    mode: "merge",
    providers: {
      moonshot: {
        baseUrl: "https://api.moonshot.ai/v1",
        apiKey: "${MOONSHOT_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "kimi-k2.5",
            name: "Kimi K2.5",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

注意事項:

* 環境変数 `MOONSHOT_API_KEY` を設定するか、`openclaw onboard --auth-choice moonshot-api-key` を使用します。
* 使用モデル: `moonshot/kimi-k2.5`。
* 中国リージョンのエンドポイントが必要な場合は、`https://api.moonshot.cn/v1` を使用します。

<div id="kimi-code">
  ### Kimi Code
</div>

Kimi Code 専用の OpenAI 互換エンドポイント（Moonshot のものとは別）を使用してください：

```json5
{
  env: { KIMICODE_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "kimi-code/kimi-for-coding" },
      models: { "kimi-code/kimi-for-coding": { alias: "Kimi Code" } }
    }
  },
  models: {
    mode: "merge",
    providers: {
      "kimi-code": {
        baseUrl: "https://api.kimi.com/coding/v1",
        apiKey: "${KIMICODE_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "kimi-for-coding",
            name: "Kimi For Coding",
            reasoning: true,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 262144,
            maxTokens: 32768,
            headers: { "User-Agent": "KimiCLI/0.77" },
            compat: { supportsDeveloperRole: false }
          }
        ]
      }
    }
  }
}
```

注意事項:

* 環境変数で `KIMICODE_API_KEY` を設定するか、`openclaw onboard --auth-choice kimi-code-api-key` を使用してください。
* モデル: `kimi-code/kimi-for-coding`。

<div id="synthetic-anthropic-compatible">
  ### Synthetic (Anthropic 互換)
</div>

SyntheticのAnthropic互換エンドポイントを使用する。

```json5
{
  env: { SYNTHETIC_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "synthetic/hf:MiniMaxAI/MiniMax-M2.1" },
      models: { "synthetic/hf:MiniMaxAI/MiniMax-M2.1": { alias: "MiniMax M2.1" } }
    }
  },
  models: {
    mode: "merge",
    providers: {
      synthetic: {
        baseUrl: "https://api.synthetic.new/anthropic",
        apiKey: "${SYNTHETIC_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "hf:MiniMaxAI/MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 192000,
            maxTokens: 65536
          }
        ]
      }
    }
  }
}
```

注意:

* `SYNTHETIC_API_KEY` を設定するか、`openclaw onboard --auth-choice synthetic-api-key` を使用してください。
* モデル指定: `synthetic/hf:MiniMaxAI/MiniMax-M2.1`。
* Anthropic クライアントが自動的に `/v1` を付加するため、ベースURLには `/v1` を含めないでください。

<div id="local-models-lm-studio-recommended-setup">
  ### ローカルモデル（LM Studio） — 推奨構成
</div>

最新のローカルモデル利用ガイドラインについては [/gateway/local-models](/ja/gateway/local-models) を参照してください。要約: 十分な性能を持つハードウェア上で LM Studio Responses API 経由で MiniMax M2.1 を実行し、フォールバック用としてホスト型モデルとのマージ設定も維持しておいてください。

<div id="minimax-m21">
  ### MiniMax M2.1
</div>

LM Studio を使わずに MiniMax M2.1 を直接利用する:

```json5
{
  agent: {
    model: { primary: "minimax/MiniMax-M2.1" },
    models: {
      "anthropic/claude-opus-4-5": { alias: "Opus" },
      "minimax/MiniMax-M2.1": { alias: "Minimax" }
    }
  },
  models: {
    mode: "merge",
    providers: {
      minimax: {
        baseUrl: "https://api.minimax.io/anthropic",
        apiKey: "${MINIMAX_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            // 価格: 正確なコスト追跡が必要な場合は models.json を更新してください。
            cost: { input: 15, output: 60, cacheRead: 2, cacheWrite: 10 },
            contextWindow: 200000,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

注意:

* `MINIMAX_API_KEY` 環境変数を設定するか、`openclaw onboard --auth-choice minimax-api` を使用してください。
* 利用可能なモデル: `MiniMax-M2.1`（デフォルト）。
* 正確なコストを追跡する必要がある場合は、`models.json` 内の料金設定を更新してください。

<div id="cerebras-glm-46-47">
  ### Cerebras（GLM 4.6 / 4.7）
</div>

Cerebras は OpenAI 互換エンドポイント経由で利用します：

```json5
{
  env: { CEREBRAS_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: {
        primary: "cerebras/zai-glm-4.7",
        fallbacks: ["cerebras/zai-glm-4.6"]
      },
      models: {
        "cerebras/zai-glm-4.7": { alias: "GLM 4.7 (Cerebras)" },
        "cerebras/zai-glm-4.6": { alias: "GLM 4.6 (Cerebras)" }
      }
    }
  },
  models: {
    mode: "merge",
    providers: {
      cerebras: {
        baseUrl: "https://api.cerebras.ai/v1",
        apiKey: "${CEREBRAS_API_KEY}",
        api: "openai-completions",
        models: [
          { id: "zai-glm-4.7", name: "GLM 4.7 (Cerebras)" },
          { id: "zai-glm-4.6", name: "GLM 4.6 (Cerebras)" }
        ]
      }
    }
  }
}
```

注意:

* Cerebras には `cerebras/zai-glm-4.7` を使用し、Z.AI へ直接接続する場合は `zai/glm-4.7` を使用します。
* `CEREBRAS_API_KEY` を環境変数またはコンフィグで設定します。

注意:

* サポートされている API: `openai-completions`, `openai-responses`, `anthropic-messages`,
  `google-generative-ai`
* カスタム認証が必要な場合は、`authHeader: true` と `headers` を使用します。
* `models.json` を別の場所に保存したい場合は、`OPENCLAW_AGENT_DIR`（または `PI_CODING_AGENT_DIR`）でエージェント設定ルートをオーバーライドします（デフォルト: `~/.openclaw/agents/main/agent`）。

<div id="session">
  ### `session`
</div>

セッションのスコープ、リセットポリシー、リセットトリガー、およびセッションストアの保存先を制御します。

```json5
{
  session: {
    scope: "per-sender",
    dmScope: "main",
    identityLinks: {
      alice: ["telegram:123456789", "discord:987654321012345678"]
    },
    reset: {
      mode: "daily",
      atHour: 4,
      idleMinutes: 60
    },
    resetByType: {
      thread: { mode: "daily", atHour: 4 },
      dm: { mode: "idle", idleMinutes: 240 },
      group: { mode: "idle", idleMinutes: 120 }
    },
    resetTriggers: ["/new", "/reset"],
    // デフォルトではエージェントごとに ~/.openclaw/agents/<agentId>/sessions/sessions.json に保存されます
    // {agentId} テンプレートを使用してオーバーライド可能:
    store: "~/.openclaw/agents/{agentId}/sessions/sessions.json",
    // ダイレクトチャットは agent:<agentId>:<mainKey> に集約されます (デフォルト: "main")。
    mainKey: "main",
    agentToAgent: {
      // リクエスト元/ターゲット間の最大ピンポン応答ターン数 (0–5)。
      maxPingPongTurns: 5
    },
    sendPolicy: {
      rules: [
        { action: "deny", match: { channel: "discord", chatType: "group" } }
      ],
      default: "allow"
    }
  }
}
```

Fields:

* `mainKey`: ダイレクトチャット用バケットキー（デフォルト: `"main"`）。`agentId` を変更せずにメインの DM スレッドの「名前だけを変えたい」場合に有用です。
  * サンドボックスに関する注意事項: `agents.defaults.sandbox.mode: "non-main"` は、このキーを使ってメインのセッションを検出します。`mainKey` に一致しない任意のセッションキー（グループ/チャンネル）はサンドボックス化されます。
* `dmScope`: DM セッションをどのようにグループ化するか（デフォルト: `"main"`）。
  * `main`: すべての DM が継続性のためにメインセッションを共有します。
  * `per-peer`: チャンネルをまたいで送信者 ID ごとに DM を分離します。
  * `per-channel-peer`: チャンネル + 送信者ごとに DM を分離します（複数ユーザーの受信トレイに推奨）。
  * `per-account-channel-peer`: アカウント + チャンネル + 送信者ごとに DM を分離します（複数アカウントの受信トレイに推奨）。
* `identityLinks`: 正規 ID をプロバイダー接頭辞付きの peer にマッピングし、`per-peer`、`per-channel-peer`、`per-account-channel-peer` 使用時でも同一人物がチャンネルをまたいで同じ DM セッションを共有できるようにします。
  * 例: `alice: ["telegram:123456789", "discord:987654321012345678"]`.
* `reset`: 主要なリセットポリシー。デフォルトでは Gateway ホストのローカル時間で毎日午前 4:00 にリセットされます。
  * `mode`: `daily` または `idle`（`reset` が存在する場合のデフォルトは `daily`）。
  * `atHour`: 毎日のリセット境界とするローカル時（0–23）。
  * `idleMinutes`: アイドル状態のスライディングウィンドウ（分）。`daily` と `idle` の両方が設定されている場合、先に期限切れになった方が優先されます。
* `resetByType`: `dm`、`group`、`thread` ごとのセッション単位の上書き設定。
  * もしレガシーな `session.idleMinutes` だけを設定し、`reset`/`resetByType` を一切設定しない場合、OpenClaw は後方互換性のためアイドル時のみのモードのまま動作します。
* `heartbeatIdleMinutes`: ハートビートチェック用のオプションのアイドル上書き設定（有効な場合でも毎日のリセットは引き続き適用されます）。
* `agentToAgent.maxPingPongTurns`: 要求元/ターゲット間の最大応答の往復ターン数（0–5、デフォルト 5）。
* `sendPolicy.default`: どのルールにも一致しない場合の `allow` または `deny` フォールバック。
* `sendPolicy.rules[]`: `channel`、`chatType`（`direct|group|room`）、または `keyPrefix`（例: `cron:`）でマッチさせます。最初にマッチした `deny` が優先され、それ以外は許可されます。

<div id="skills-skills-config">
  ### `skills` (スキル設定)
</div>

バンドル済みスキル用の許可リスト、インストール設定、追加スキルディレクトリ、およびスキルごとの
オーバーライドを制御します。**バンドル済み**スキルと `~/.openclaw/skills` に適用されます（名前が衝突する場合は、
引き続きワークスペース内スキルが優先されます）。

フィールド:

* `allowBundled`: **バンドル済み**スキル専用の任意の許可リスト。設定した場合、その
  バンドル済みスキルのみが利用可能になります（managed スキルやワークスペーススキルには影響しません）。
* `load.extraDirs`: 追加で走査するスキルディレクトリ（優先度は最も低い）。
* `install.preferBrew`: 利用可能な場合は brew インストーラを優先（デフォルト: true）。
* `install.nodeManager`: Node.js パッケージマネージャの優先指定（`npm` | `pnpm` | `yarn`, デフォルト: npm）。
* `entries.<skillKey>`: スキルごとの設定オーバーライド。

スキルごとのフィールド:

* `enabled`: バンドル済み/インストール済みであっても、そのスキルを無効化するには `false` を設定。
* `env`: エージェント実行時に注入する環境変数（既に設定されている場合は除く）。
* `apiKey`: メインとなる環境変数を宣言しているスキル向けの任意指定用ショートカット（例: `nano-banana-pro` → `GEMINI_API_KEY`）。

例:

```json5
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: [
        "~/Projects/agent-scripts/skills",
        "~/Projects/oss/some-skill-pack/skills"
      ]
    },
    install: {
      preferBrew: true,
      nodeManager: "npm"
    },
    entries: {
      "nano-banana-pro": {
        apiKey: "GEMINI_KEY_HERE",
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE"
        }
      },
      peekaboo: { enabled: true },
      sag: { enabled: false }
    }
  }
}
```

<div id="plugins-extensions">
  ### `plugins` (拡張機能)
</div>

プラグインの検出、許可/拒否、およびプラグインごとの設定を制御します。プラグインは
`~/.openclaw/extensions`、`<workspace>/.openclaw/extensions`、および
`plugins.load.paths` の各エントリから読み込まれます。**設定変更を反映するには Gateway を再起動する必要があります。**
完全な使い方は [/plugin](/ja/plugin) を参照してください。

フィールド:

* `enabled`: プラグイン読み込みのマスタートグル（デフォルト: true）。
* `allow`: プラグイン ID の任意指定の許可リスト。設定すると、記載されたプラグインのみが読み込まれます。
* `deny`: プラグイン ID の任意指定の拒否リスト（deny が優先されます）。
* `load.paths`: 追加で読み込むプラグインファイルまたはディレクトリ（絶対パスまたは `~`）。
* `entries.<pluginId>`: プラグインごとの上書き設定。
  * `enabled`: `false` を設定すると無効化。
  * `config`: プラグイン固有の設定オブジェクト（指定された場合、プラグイン側で検証されます）。

例:

```json5
{
  plugins: {
    enabled: true,
    allow: ["voice-call"],
    load: {
      paths: ["~/Projects/oss/voice-call-extension"]
    },
    entries: {
      "voice-call": {
        enabled: true,
        config: {
          provider: "twilio"
        }
      }
    }
  }
}
```

<div id="browser-openclaw-managed-browser">
  ### `browser` (openclaw 管理ブラウザ)
</div>

OpenClaw は、openclaw 用に **専用かつ分離された** Chrome/Brave/Edge/Chromium インスタンスを起動し、小さなループバック制御サービスを提供できます。
プロファイルは、`profiles.<name>.cdpUrl` を通じて **リモート** の Chromium 系ブラウザを指すことができます。リモート
プロファイルはアタッチのみ可能であり（start/stop/reset は無効）です。

`browser.cdpUrl` は、レガシーな単一プロファイル構成向けとして、また `cdpPort` だけを設定しているプロファイルに対する
ベースとなる scheme/host として残されています。

デフォルト値:

* enabled: `true`
* evaluateEnabled: `true`（`false` にすると `act:evaluate` と `wait --fn` を無効化）
* 制御サービス: ループバックのみ（ポートは `gateway.port` から導出、デフォルトは `18791`）
* CDP URL: `http://127.0.0.1:18792`（制御サービスのポート + 1、レガシーな単一プロファイル用）
* プロファイル色: `#FF4500`（lobster-orange）
* 注意: 制御サーバーは、実行中の Gateway（OpenClaw.app のメニューバー、または `openclaw gateway`）によって起動されます。
* 自動検出の順序: 既定ブラウザが Chromium 系であればそれを使用し、それ以外の場合は Chrome → Brave → Edge → Chromium → Chrome Canary の順で探索します。

```json5
{
  browser: {
    enabled: true,
    evaluateEnabled: true,
    // cdpUrl: "http://127.0.0.1:18792", // legacy single-profile override
    defaultProfile: "chrome",
    profiles: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work: { cdpPort: 18801, color: "#0066CC" },
      remote: { cdpUrl: "http://10.0.0.42:9222", color: "#00AA00" }
    },
    color: "#FF4500",
    // Advanced:
    // headless: false,
    // noSandbox: false,
    // executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    // attachOnly: false, // リモートCDPをlocalhostへトンネリングする際はtrueに設定
  }
}
```

<div id="ui-appearance">
  ### `ui` (外観)
</div>

ネイティブアプリが UI クローム用に使用する任意のアクセントカラーです（例: Talk Mode の吹き出しの色合い）。

未設定の場合、クライアントは控えめな淡い青色をデフォルトとして使用します。

```json5
{
  ui: {
    seamColor: "#FF4500", // hex (RRGGBB or #RRGGBB)
    // オプション: Control UI アシスタントのアイデンティティ上書き設定。
    // 未設定の場合、Control UI はアクティブなエージェントのアイデンティティ (config または IDENTITY.md) を使用します。
    assistant: {
      name: "OpenClaw",
      avatar: "CB" // emoji, short text, or image URL/data URI
    }
  }
}
```

<div id="gateway-gateway-server-mode-bind">
  ### `gateway` (Gateway サーバーモード + バインド)
</div>

このマシンで Gateway を実行するかどうかを明示的に指定するには、`gateway.mode` を使用します。

デフォルト値:

* mode: **未設定**（「自動起動しない」として扱われます）
* bind: `loopback`
* port: `18789`（WS + HTTP 用の単一ポート）

```json5
{
  gateway: {
    mode: "local", // or "remote"
    port: 18789, // WS + HTTP multiplex
    bind: "loopback",
    // controlUi: { enabled: true, basePath: "/openclaw" }
    // auth: { mode: "token", token: "your-token" } // トークンで WS + Control UI アクセスを制御
    // tailscale: { mode: "off" | "serve" | "funnel" }
  }
}
```

Control UI のベースパス:

* `gateway.controlUi.basePath` は、Control UI が配信される URL プレフィックスを設定します。
* 例: `"/ui"`, `"/openclaw"`, `"/apps/openclaw"`。
* デフォルト: ルート (`/`)（変更なし）。
* `gateway.controlUi.allowInsecureAuth` は、デバイス ID が省略された場合（通常は HTTP 経由）に、Control UI へのトークンのみの認証を許可します。デフォルト: `false`。HTTPS（Tailscale Serve）または `127.0.0.1` の利用を推奨します。
* `gateway.controlUi.dangerouslyDisableDeviceAuth` は、Control UI に対するデバイス ID チェックを無効化します（トークン/パスワードのみ）。デフォルト: `false`。緊急時のみ使用してください。

関連ドキュメント:

* [Control UI](/ja/web/control-ui)
* [Web overview](/ja/web)
* [Tailscale](/ja/gateway/tailscale)
* [Remote access](/ja/gateway/remote)

信頼済みプロキシ:

* `gateway.trustedProxies`: Gateway の前段で TLS を終端するリバースプロキシ IP のリスト。
* 接続元がこれらの IP のいずれかである場合、OpenClaw はローカルのペアリングチェックおよび HTTP 認証/ローカルチェックにおけるクライアント IP の決定に `x-forwarded-for`（または `x-real-ip`）を使用します。
* 完全に管理しているプロキシのみを列挙し、受信した `x-forwarded-for` を必ず**上書き**するようにしてください。

注意事項:

* `gateway.mode` が `local` に設定されていない限り（またはオーバーライドフラグを渡さない限り）、`openclaw gateway` は起動を拒否します。
* `gateway.port` は、WebSocket と HTTP（Control UI、hooks、A2UI）で共有される単一の多重化ポートを制御します。
* OpenAI Chat Completions エンドポイント: **デフォルトでは無効**。`gateway.http.endpoints.chatCompletions.enabled: true` で有効化します。
* 優先順位: `--port` &gt; `OPENCLAW_GATEWAY_PORT` &gt; `gateway.port` &gt; デフォルト `18789`。
* Gateway 認証はデフォルトで必須です（トークン/パスワード、または Tailscale Serve の ID）。ループバック以外へのバインドには共有トークン/パスワードが必要です。
* オンボーディングウィザードは、デフォルトで Gateway トークンを生成します（ループバック上でも同様）。
* `gateway.remote.token` はリモート CLI コール**専用**であり、ローカル Gateway 認証は有効化しません。`gateway.token` は無視されます。

認証と Tailscale:

* `gateway.auth.mode` はハンドシェイク要件（`token` または `password`）を設定します。未設定の場合、トークン認証が前提となります。
* `gateway.auth.token` はトークン認証のための共有トークンを保存します（同一マシン上の CLI が使用）。
* `gateway.auth.mode` が設定されている場合、その方式のみが受け付けられます（+ 任意の Tailscale ヘッダ）。
* `gateway.auth.password` はここに設定するか、`OPENCLAW_GATEWAY_PASSWORD` 経由で設定できます（推奨）。
* `gateway.auth.allowTailscale` は、リクエストがループバック上に `x-forwarded-for`、`x-forwarded-proto`、`x-forwarded-host` 付きで到達した場合に、Tailscale Serve の ID ヘッダ（`tailscale-user-login`）による認証を許可します。OpenClaw は受け入れ前に、`tailscale whois` を用いて `x-forwarded-for` アドレスを解決することで ID を検証します。`true` のとき、Serve からのリクエストにはトークン/パスワードが不要になります。明示的な資格情報を必須にするには `false` を設定してください。`tailscale.mode = "serve"` かつ auth mode が `password` でない場合、デフォルトは `true` です。
* `gateway.tailscale.mode: "serve"` は Tailscale Serve を使用します（tailnet のみ、ループバックバインド）。
* `gateway.tailscale.mode: "funnel"` はダッシュボードをパブリックに公開します。認証が必須です。
* `gateway.tailscale.resetOnExit` は、シャットダウン時に Serve/Funnel 設定をリセットします。

リモートクライアントのデフォルト設定（CLI）:

* `gateway.remote.url` は、`gateway.mode = "remote"` のときに CLI 呼び出しで使用されるデフォルトの Gateway WebSocket URL を設定します。
* `gateway.remote.transport` は macOS のリモート接続方式を選択します（デフォルトは `ssh`、`direct` は ws/wss 用）。`direct` の場合、`gateway.remote.url` は `ws://` または `wss://` でなければなりません。`ws://host` の場合、デフォルトでポート `18789` を使用します。
* `gateway.remote.token` はリモート呼び出し用のトークンを指定します（認証なしの場合は未設定のままにします）。
* `gateway.remote.password` はリモート呼び出し用のパスワードを指定します（認証なしの場合は未設定のままにします）。

macOS アプリの動作:

* OpenClaw.app は `~/.openclaw/openclaw.json` を監視し、`gateway.mode` または `gateway.remote.url` が変更されるとモードをリアルタイムに切り替えます。
* `gateway.mode` が未設定で `gateway.remote.url` が設定されている場合、macOS アプリはリモートモードとして扱います。
* macOS アプリで接続モードを変更すると、`gateway.mode`（およびリモートモードでは `gateway.remote.url` と `gateway.remote.transport`）を設定ファイルに書き戻します。

```json5
{
  gateway: {
    mode: "remote",
    remote: {
      url: "ws://gateway.tailnet:18789",
      token: "your-token",
      password: "your-password"
    }
  }
}
```

直接転送の例（macOS アプリ）：

```json5
{
  gateway: {
    mode: "remote",
    remote: {
      transport: "direct",
      url: "wss://gateway.example.ts.net",
      token: "your-token"
    }
  }
}
```

<div id="gatewayreload-config-hot-reload">
  ### `gateway.reload` (設定のホットリロード)
</div>

Gateway は `~/.openclaw/openclaw.json`（または `OPENCLAW_CONFIG_PATH`）を監視し、変更を自動的に適用します。

モード:

* `hybrid`（デフォルト）: 安全な変更はホット適用し、重大な変更は Gateway を再起動します。
* `hot`: ホット適用可能な安全な変更のみを適用し、再起動が必要な場合はログに記録します。
* `restart`: いかなる設定変更でも検知したら Gateway を再起動します。
* `off`: ホットリロードを無効にします。

```json5
{
  gateway: {
    reload: {
      mode: "hybrid",
      debounceMs: 300
    }
  }
}
```

<div id="hot-reload-matrix-files-impact">
  #### ホットリロードのマトリックス（ファイル + 影響範囲）
</div>

監視対象ファイル:

* `~/.openclaw/openclaw.json`（または `OPENCLAW_CONFIG_PATH`）

ホット適用（Gateway の完全再起動は不要）:

* `hooks`（webhook の認証／パス／マッピング）+ `hooks.gmail`（Gmail ウォッチャー再起動）
* `browser`（ブラウザー制御サーバー再起動）
* `cron`（cron サービス再起動 + 同時実行数の更新）
* `agents.defaults.heartbeat`（ハートビート実行プロセス再起動）
* `web`（WhatsApp Web チャンネル再起動）
* `telegram`、`discord`、`signal`、`imessage`（チャンネル再起動）
* `agent`、`models`、`routing`、`messages`、`session`、`whatsapp`、`logging`、`skills`、`ui`、`talk`、`identity`、`wizard`（動的読み取り）

Gateway の完全再起動が必要:

* `gateway`（ポート／バインド／認証／Control UI／Tailscale）
* `bridge`（レガシー）
* `discovery`
* `canvasHost`
* `plugins`
* 不明／未サポートの任意の設定パス（安全のためデフォルトで再起動）

<div id="multi-instance-isolation">
  ### 複数インスタンスの分離
</div>

1台のホスト上で複数の Gateway を動かす場合（冗長構成やレスキューボット用など）、インスタンスごとに状態と設定を分離し、ポート番号を一意に設定してください:

* `OPENCLAW_CONFIG_PATH`（インスタンスごとの設定）
* `OPENCLAW_STATE_DIR`（セッション/クレデンシャル）
* `agents.defaults.workspace`（メモリデータ）
* `gateway.port`（インスタンスごとに一意な値）

便利なフラグ（CLI）:

* `openclaw --dev …` → `~/.openclaw-dev` を使用し、基準ポート `19001` からポートをシフト
* `openclaw --profile <name> …` → `~/.openclaw-<name>` を使用（ポートは config/env/フラグで指定）

導出されるポートマッピング（gateway/browser/canvas）については [Gateway runbook](/ja/gateway) を参照してください。
ブラウザ/CDP ポートの分離の詳細は [Multiple gateways](/ja/gateway/multiple-gateways) を参照してください。

例:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

<div id="hooks-gateway-webhooks">
  ### `hooks` (Gateway webhooks)
</div>

Gateway の HTTP サーバー上で、シンプルな HTTP webhook エンドポイントを有効にします。

デフォルト値:

* enabled: `false`
* path: `/hooks`
* maxBodyBytes: `262144` (256 KB)

```json5
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
    presets: ["gmail"],
    transformsDir: "~/.openclaw/hooks",
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
        wakeMode: "now",
        name: "Gmail",
        sessionKey: "hook:gmail:{{messages[0].id}}",
        messageTemplate:
          "From: {{messages[0].from}}\nSubject: {{messages[0].subject}}\n{{messages[0].snippet}}",
        deliver: true,
        channel: "last",
        model: "openai/gpt-5.2-mini",
      },
    ],
  }
}
```

リクエストには hook トークンを含める必要があります:

* `Authorization: Bearer <token>` **または**
* `x-openclaw-token: <token>` **または**
* `?token=<token>`

エンドポイント:

* `POST /hooks/wake` → `{ text, mode?: "now"|"next-heartbeat" }`
* `POST /hooks/agent` → `{ message, name?, sessionKey?, wakeMode?, deliver?, channel?, to?, model?, thinking?, timeoutSeconds? }`
* `POST /hooks/<name>` → `hooks.mappings` で解決されます

`/hooks/agent` は常にメインのセッションに要約を投稿し（任意で `wakeMode: "now"` により即時のハートビートをトリガーできます）。

マッピングに関する補足:

* `match.path` は `/hooks` の後ろのサブパスにマッチします（例: `/hooks/gmail` → `gmail`）。
* `match.source` はペイロードフィールドにマッチします（例: `{ source: "gmail" }`）。これにより汎用的な `/hooks/ingest` パスを使うことができます。
* `{{messages[0].subject}}` のようなテンプレートはペイロードから値を読み取ります。
* `transform` には、hook アクションを返す JS/TS モジュールを指定できます。
* `deliver: true` は最終的な返信をチャネルに送信します。`channel` のデフォルトは `last` で（WhatsApp をフォールバックとして使用します）。
* 既存の配信ルートがない場合は、`channel` と `to` を明示的に指定してください（Telegram/Discord/Google Chat/Slack/Signal/iMessage/MS Teams では必須です）。
* `model` はこの hook 実行に対して使用する LLM を上書きします（`provider/model` またはエイリアス。`agents.defaults.models` が設定されている場合、その中で許可されている必要があります）。

Gmail 用ヘルパー設定（`openclaw webhooks gmail setup` / `run` で使用）:

```json5
{
  hooks: {
    gmail: {
      account: "openclaw@gmail.com",
      topic: "projects/<project-id>/topics/gog-gmail-watch",
      subscription: "gog-gmail-watch-push",
      pushToken: "shared-push-token",
      hookUrl: "http://127.0.0.1:18789/hooks/gmail",
      includeBody: true,
      maxBytes: 20000,
      renewEveryMinutes: 720,
      serve: { bind: "127.0.0.1", port: 8788, path: "/" },
      tailscale: { mode: "funnel", path: "/gmail-pubsub" },

      // オプション: Gmail フック処理に低コストのモデルを使用
      // 認証/レート制限/タイムアウト時は agents.defaults.model.fallbacks、次にプライマリにフォールバック
      model: "openrouter/meta-llama/llama-3.3-70b-instruct:free",
      // Optional: default thinking level for Gmail hooks
      thinking: "off",
    }
  }
}
```

Gmail フック用のモデル上書き設定:

* `hooks.gmail.model` は、Gmail フック処理に使用するモデルを指定します（デフォルトはセッションのプライマリモデル）。
* `provider/model` 形式の参照、または `agents.defaults.models` からのエイリアスを受け付けます。
* 認証エラー / レート制限 / タイムアウト発生時には、`agents.defaults.model.fallbacks`、次に `agents.defaults.model.primary` へフォールバックします。
* `agents.defaults.models` が設定されている場合は、フック用モデルを許可リストに含めてください。
* 起動時、設定されたモデルがモデルカタログまたは許可リストに存在しない場合は警告を出します。
* `hooks.gmail.thinking` は Gmail フックのデフォルトの思考レベルを設定し、フックごとの `thinking` によって上書きされます。

Gateway の自動起動:

* `hooks.enabled=true` かつ `hooks.gmail.account` が設定されている場合、Gateway は起動時に
  `gog gmail watch serve` を起動し、監視を自動更新します。
* 自動起動を無効化するには（手動実行する場合）、`OPENCLAW_SKIP_GMAIL_WATCHER=1` を設定します。
* Gateway とは別に `gog gmail watch serve` を別途起動することは避けてください。その場合、
  `listen tcp 127.0.0.1:8788: bind: address already in use` というエラーで失敗します。

注意: `tailscale.mode` がオンのとき、OpenClaw は `serve.path` のデフォルト値を `/` に設定し、
Tailscale が `/gmail-pubsub` を正しくプロキシできるようにします
（Tailscale が設定されたパスプレフィックスを削除するため）。
プレフィックス付きパスをバックエンド側で受け取りたい場合は、
`hooks.gmail.tailscale.target` を完全な URL に設定し（かつ `serve.path` をそれに合わせて調整）してください。

<div id="canvashost-lantailnet-canvas-file-server-live-reload">
  ### `canvasHost` (LAN/Tailnet Canvas ファイルサーバー + ライブリロード)
</div>

Gateway は HTML/CSS/JS のディレクトリを HTTP で配信し、iOS/Android ノードが単に `canvas.navigate` するだけでアクセスできるようにします。

デフォルトルート: `~/.openclaw/workspace/canvas`
デフォルトポート: `18793`（openclaw ブラウザの CDP ポート `18792` と衝突しないように選択）
サーバーは **gateway bind host**（LAN または Tailnet）で待ち受けるので、ノードから接続できます。

このサーバーは次を行います:

* `canvasHost.root` 配下のファイルを配信する
* 配信する HTML に小さなライブリロードクライアントを挿入する
* ディレクトリを監視し、`/__openclaw__/ws` の WebSocket エンドポイント経由でリロードをブロードキャストする
* ディレクトリが空の場合、スターター用の `index.html` を自動作成する（すぐに何かが表示されるようにする）
* `/__openclaw__/a2ui/` で A2UI も配信し、ノードには `canvasHostUrl` としてアドバタイズされる
  （ノードは Canvas/A2UI 用に常にこれを使用）

ディレクトリが大きい場合や `EMFILE` が発生する場合は、ライブリロード（およびファイル監視）を無効化します:

* 設定: `canvasHost: { liveReload: false }`

```json5
{
  canvasHost: {
    root: "~/.openclaw/workspace/canvas",
    port: 18793,
    liveReload: true
  }
}
```

`canvasHost.*` の変更には Gateway の再起動が必要です（config のリロードでも再起動されます）。

無効化するには次のいずれかを使用します:

* config: `canvasHost: { enabled: false }`
* env: `OPENCLAW_SKIP_CANVAS_HOST=1`

<div id="bridge-legacy-tcp-bridge-removed">
  ### `bridge` (レガシー TCP ブリッジ、削除済み)
</div>

現在のビルドには TCP ブリッジリスナーは含まれておらず、`bridge.*` の設定キーは無視されます。
ノードは Gateway の WebSocket 経由で接続します。このセクションは歴史的な参考情報として残されています。

従来の動作:

* Gateway はノード（iOS/Android）向けに、シンプルな TCP ブリッジを公開できました（通常はポート `18790`）。

デフォルト値:

* enabled: `true`
* port: `18790`
* bind: `lan`（`0.0.0.0` にバインド）

バインドモード:

* `lan`: `0.0.0.0`（LAN/Wi‑Fi および Tailscale を含む任意のインターフェイスから到達可能）
* `tailnet`: マシンの Tailscale IP のみにバインド（Vienna ⇄ London に推奨）
* `loopback`: `127.0.0.1`（ローカルのみ）
* `auto`: tailnet IP があればそれを優先し、なければ `lan`

TLS:

* `bridge.tls.enabled`: ブリッジ接続に TLS を有効化（有効な場合は TLS のみ）。
* `bridge.tls.autoGenerate`: cert/key が存在しない場合に自己署名証明書を生成（デフォルト: true）。
* `bridge.tls.certPath` / `bridge.tls.keyPath`: ブリッジ用証明書および秘密鍵の PEM パス。
* `bridge.tls.caPath`: 任意の PEM CA バンドル（カスタムルートや将来の mTLS 用）。

TLS が有効な場合、Gateway はディスカバリー用 TXT レコード内で `bridgeTls=1` と
`bridgeTlsSha256` をアドバタイズし、ノード側で証明書ピン留めができるようにします。手動接続では、
フィンガープリントがまだ保存されていない場合、trust-on-first-use（初回接続時信頼）を使用します。
自己署名証明書の自動生成には PATH に `openssl` が存在している必要があります。生成に失敗した場合、ブリッジは起動しません。

```json5
{
  bridge: {
    enabled: true,
    port: 18790,
    bind: "tailnet",
    tls: {
      enabled: true,
      // 省略した場合は ~/.openclaw/bridge/tls/bridge-{cert,key}.pem が使用されます。
      // certPath: "~/.openclaw/bridge/tls/bridge-cert.pem",
      // keyPath: "~/.openclaw/bridge/tls/bridge-key.pem"
    }
  }
}
```

<div id="discoverymdns-bonjour-mdns-broadcast-mode">
  ### `discovery.mdns` (Bonjour / mDNS ブロードキャストモード)
</div>

LAN 上の mDNS ディスカバリーブロードキャスト（`_openclaw-gw._tcp`）を制御します。

* `minimal` (デフォルト): TXT レコードから `cliPath` と `sshPort` を省略
* `full`: TXT レコードに `cliPath` と `sshPort` を含める
* `off`: mDNS ブロードキャストを完全に無効化
* ホスト名: デフォルトは `openclaw`（`openclaw.local` としてアドバタイズ）。`OPENCLAW_MDNS_HOSTNAME` で上書き可能。

```json5
{
  discovery: { mdns: { mode: "minimal" } }
}
```

<div id="discoverywidearea-wide-area-bonjour-unicast-dnssd">
  ### `discovery.wideArea` (広域 Bonjour / ユニキャスト DNS‑SD)
</div>

有効化すると、Gateway は設定された discovery ドメイン（例: `openclaw.internal.`）配下向けに、`~/.openclaw/dns/` 内へ `_openclaw-gw._tcp` のユニキャスト DNS-SD ゾーンを書き出します。

iOS/Android からネットワークをまたいで検出できるようにする場合（例: Vienna ⇄ London）、次の設定と組み合わせて使用します:

* 選択したドメインを提供する、Gateway ホスト上の DNS サーバー（CoreDNS 推奨）
* クライアントがそのドメインを Gateway の DNS サーバー経由で解決するようにするための、Tailscale の **split DNS**

ワンタイムセットアップ用ヘルパー（Gateway ホスト側）:

```bash
openclaw dns setup --apply
```

```json5
{
  discovery: { wideArea: { enabled: true } }
}
```

## テンプレート変数

テンプレートプレースホルダーは、`tools.media.*.models[].args` および `tools.media.models[].args`（および将来追加されるテンプレート引数フィールド）で展開されます。

| Variable | Description |
|----------|-------------|
| `{{Body}}` | 受信メッセージ本文全体 |
| `{{RawBody}}` | 生の受信メッセージ本文（履歴／送信者ラッパーなし。コマンド解析に最適） |
| `{{BodyStripped}}` | グループメンションを除去した本文（エージェント向けのデフォルト） |
| `{{From}}` | 送信者識別子（WhatsApp では E.164。他チャネルでは異なる場合あり） |
| `{{To}}` | 宛先識別子 |
| `{{MessageSid}}` | チャネルメッセージ ID（利用可能な場合） |
| `{{SessionId}}` | 現在のセッション UUID |
| `{{IsNewSession}}` | 新しいセッションが作成された場合は `"true"` |
| `{{MediaUrl}}` | 受信メディアの疑似 URL（存在する場合） |
| `{{MediaPath}}` | ローカルメディアパス（ダウンロード済みの場合） |
| `{{MediaType}}` | メディアタイプ（image/audio/document/…） |
| `{{Transcript}}` | 音声の書き起こし（有効な場合） |
| `{{Prompt}}` | CLI エントリ用に解決されたメディアプロンプト |
| `{{MaxChars}}` | CLI エントリ用に解決された最大出力文字数 |
| `{{ChatType}}` | `"direct"` または `"group"` |
| `{{GroupSubject}}` | グループの件名（ベストエフォート） |
| `{{GroupMembers}}` | グループメンバーのプレビュー（ベストエフォート） |
| `{{SenderName}}` | 送信者の表示名（ベストエフォート） |
| `{{SenderE164}}` | 送信者の電話番号（ベストエフォート） |
| `{{Provider}}` | プロバイダーのヒント（whatsapp|telegram|discord|googlechat|slack|signal|imessage|msteams|webchat|…） |

<div id="cron-gateway-scheduler">
  ## Cron (Gateway スケジューラ)
</div>

Cron は、ウェイクアップやスケジュールされたジョブのための、Gateway 管理のスケジューラです。機能の概要と CLI の例については、[Cron ジョブ](/ja/automation/cron-jobs) を参照してください。

```json5
{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2
  }
}
```

***

*次へ: [エージェントのランタイム](/ja/concepts/agent)* 🦞
