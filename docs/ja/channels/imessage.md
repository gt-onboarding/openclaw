---
title: iMessage
summary: "imsg（stdio 経由の JSON-RPC）を介した iMessage サポート、セットアップ、および chat_id ルーティング"
read_when:
  - iMessage サポートをセットアップするとき
  - iMessage の送受信をデバッグしているとき
---

<div id="imessage-imsg">
  # iMessage (imsg)
</div>

状態: 外部CLI統合。Gateway が `imsg rpc`（標準入出力経由の JSON-RPC）を起動します。

<div id="quick-setup-beginner">
  ## クイックセットアップ（初心者向け）
</div>

1. この Mac の「メッセージ」アプリにサインインしていることを確認します。
2. `imsg` をインストールします:
   * `brew install steipete/tap/imsg`
3. OpenClaw の `channels.imessage.cliPath` と `channels.imessage.dbPath` を設定します。
4. Gateway を起動し、macOS のプロンプト（オートメーション + フルディスクアクセス）をすべて許可します。

最小構成:

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "/usr/local/bin/imsg",
      dbPath: "/Users/<あなた>/Library/Messages/chat.db"
    }
  }
}
```

<div id="what-it-is">
  ## これは何か
</div>

* macOS 上の `imsg` をバックエンドにした iMessage チャンネル。
* 決定論的ルーティング：返信は必ず iMessage 側に返される。
* DM（ダイレクトメッセージ）はエージェントのメインセッションを共有し、グループは分離される（`agent:<agentId>:imessage:group:<chat_id>`）。
* 複数人参加のスレッドが `is_group=false` のまま届いた場合でも、`channels.imessage.groups` を使って `chat_id` ごとに分離できる（後述の「Group-ish threads」を参照）。

<div id="config-writes">
  ## Config の書き込み
</div>

デフォルトでは、iMessage から `/config set|unset` によってトリガーされた設定更新を書き込むことが許可されています（`commands.config: true` が必要）。

これを無効化するには次のようにします:

```json5
{
  channels: { imessage: { configWrites: false } }
}
```

<div id="requirements">
  ## 要件
</div>

* Messages にサインイン済みの macOS。
* OpenClaw と `imsg` に対するフルディスクアクセス権（Messages データベースへのアクセス）。
* 送信時のオートメーション（自動化）実行の許可。
* `channels.imessage.cliPath` には、stdin/stdout をプロキシする任意のコマンドを指定できる（例: 別の Mac へ SSH して `imsg rpc` を実行するラッパースクリプト）。

<div id="setup-fast-path">
  ## セットアップ（クイックパス）
</div>

1. このMacで「メッセージ」にサインインしていることを確認します。
2. iMessage を設定して Gateway を起動します。

<div id="dedicated-bot-macos-user-for-isolated-identity">
  ### 専用のボット用 macOS ユーザー（アイデンティティ分離用）
</div>

ボットが **別の iMessage アイデンティティ** から送信するようにし（自分の個人用メッセージはそのままきれいに保ちたい）場合は、専用の Apple ID と専用の macOS ユーザーを使用します。

1. 専用の Apple ID を作成します（例: `my-cool-bot@icloud.com`）。
   * Apple が、認証や 2 要素認証（2FA）用の電話番号を要求する場合があります。
2. macOS ユーザー（例: `openclawhome`）を作成し、そのユーザーでサインインします。
3. その macOS ユーザーで Messages を開き、ボット用 Apple ID を使って iMessage にサインインします。
4. リモートログインを有効にします（System Settings → General → Sharing → Remote Login）。
5. `imsg` をインストールします:
   * `brew install steipete/tap/imsg`
6. `ssh <bot-macos-user>@localhost true` がパスワードなしで動作するように SSH を設定します。
7. `channels.imessage.accounts.bot.cliPath` を、ボットユーザーとして `imsg` を実行する SSH ラッパーに向けます。

初回実行時の注意: 送信/受信には、*ボット用 macOS ユーザー* 側での GUI による承認（Automation + Full Disk Access）が必要になる場合があります。`imsg rpc` が固まっているように見える、またはすぐに終了してしまう場合は、そのユーザーにログインし（Screen Sharing が役立ちます）、一度だけ `imsg chats --limit 1` / `imsg send ...` を実行してプロンプトを承認し、その後再試行してください。

ラッパーの例（`chmod +x` を実行）。`<bot-macos-user>` を実際の macOS ユーザー名に置き換えてください:

```bash
#!/usr/bin/env bash
set -euo pipefail

# 最初に対話型SSHを一度実行してホストキーを承認します:
#   ssh <bot-macos-user>@localhost true
exec /usr/bin/ssh -o BatchMode=yes -o ConnectTimeout=5 -T <bot-macos-user>@localhost \
  "/usr/local/bin/imsg" "$@"
```

設定例:

```json5
{
  channels: {
    imessage: {
      enabled: true,
      accounts: {
        bot: {
          name: "Bot",
          enabled: true,
          cliPath: "/path/to/imsg-bot",
          dbPath: "/Users/<bot-macos-user>/Library/Messages/chat.db"
        }
      }
    }
  }
}
```

単一アカウント構成では、`accounts` マップの代わりにフラットなオプション（`channels.imessage.cliPath`、`channels.imessage.dbPath`）を使用してください。

<div id="remotessh-variant-optional">
  ### リモート/SSH バリアント（オプション）
</div>

別の Mac 上で iMessage を使いたい場合は、`channels.imessage.cliPath` を、SSH 経由でリモートの macOS ホスト上の `imsg` を実行するラッパーに設定してください。OpenClaw が必要とするのは標準入出力（stdio）のみです。

ラッパーの例:

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

**リモート添付ファイル:** `cliPath` が SSH 経由でリモートホストを指している場合、Messages データベース内の添付ファイルのパスはリモートマシン上のファイルを参照します。`channels.imessage.remoteHost` を設定すると、OpenClaw がこれらを SCP 経由で自動的に取得します。

```json5
{
  channels: {
    imessage: {
      cliPath: "~/imsg-ssh",                     // リモートMacへのSSHラッパー
      remoteHost: "user@gateway-host",           // for SCP file transfer
      includeAttachments: true
    }
  }
}
```

`remoteHost` が設定されていない場合、OpenClaw はラッパースクリプト内の SSH コマンドを解析して自動検出を試みます。信頼性を高めるために、明示的に設定しておくことを推奨します。

<div id="remote-mac-via-tailscale-example">
  #### Tailscale 経由のリモート Mac（例）
</div>

Gateway が Linux ホスト／VM 上で動作しているが、iMessage は Mac 上で実行する必要がある場合、Tailscale が最も簡単なブリッジとなります。Gateway は tailnet 経由で Mac と通信し、SSH で `imsg` を実行し、添付ファイルを SCP で取得します。

アーキテクチャ:

```
┌──────────────────────────────┐          SSH (imsg rpc)          ┌──────────────────────────┐
│ Gateway host (Linux/VM)      │──────────────────────────────────▶│ Mac with Messages + imsg │
│ - openclaw gateway           │          SCP (attachments)        │ - Messages signed in     │
│ - channels.imessage.cliPath  │◀──────────────────────────────────│ - Remote Login enabled   │
└──────────────────────────────┘                                   └──────────────────────────┘
              ▲
              │ Tailscale tailnet (hostname or 100.x.y.z)
              ▼
        user@gateway-host
```

具体的な設定例（Tailscale のホスト名）:

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.openclaw/scripts/imsg-ssh",
      remoteHost: "bot@mac-mini.tailnet-1234.ts.net",
      includeAttachments: true,
      dbPath: "/Users/bot/Library/Messages/chat.db"
    }
  }
}
```

ラッパースクリプトの例（`~/.openclaw/scripts/imsg-ssh`）：

```bash
#!/usr/bin/env bash
exec ssh -T bot@mac-mini.tailnet-1234.ts.net imsg "$@"
```

注意:

* Mac がメッセージにサインインしており、リモートログインが有効になっていることを確認します。
* SSH 鍵を使用し、`ssh bot@mac-mini.tailnet-1234.ts.net` がプロンプトなしで実行できるようにしておきます。
* SCP が添付ファイルを取得できるように、`remoteHost` は SSH の接続先と一致させてください。

マルチアカウント対応: `channels.imessage.accounts` を使用し、アカウントごとの設定および任意の `name` を指定します。共通パターンについては [`gateway/configuration`](/ja/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) を参照してください。`~/.openclaw/openclaw.json` はコミットしないでください (トークンが含まれていることが多いため)。

<div id="access-control-dms-groups">
  ## アクセス制御（DM + グループ）
</div>

DM:

* デフォルト: `channels.imessage.dmPolicy = "pairing"`。
* 未知の送信者にはペアリングコードが送信され、承認されるまでメッセージは無視されます（コードは 1 時間で失効します）。
* 承認方法:
  * `openclaw pairing list imessage`
  * `openclaw pairing approve imessage <CODE>`
* ペアリングは iMessage DM におけるデフォルトのトークン交換方式です。詳細: [Pairing](/ja/start/pairing)

グループ:

* `channels.imessage.groupPolicy = open | allowlist | disabled`。`open` は、任意のユーザーからメッセージを無制限に受け付ける設定です。
* `channels.imessage.groupAllowFrom` は、`allowlist` が設定されているときに、誰がグループ内でエージェントをトリガーできるかを制御します。
* iMessage にはネイティブなメンション用メタデータがないため、メンションによるゲート制御には `agents.list[].groupChat.mentionPatterns`（または `messages.groupChat.mentionPatterns`）を使用します。
* マルチエージェント構成での上書き: `agents.list[].groupChat.mentionPatterns` にエージェント単位のパターンを設定します。

<div id="how-it-works-behavior">
  ## 動作概要（挙動）
</div>

* `imsg` はメッセージイベントをストリーミングし、Gateway がそれらを共通のチャネルエンベロープに正規化します。
* 返信は常に同じチャット ID またはハンドルにルーティングされます。

<div id="group-ish-threads-is_groupfalse">
  ## グループに近いスレッド (`is_group=false`)
</div>

一部の iMessage スレッドは、参加者が複数いても、Messages アプリがチャット識別子をどのように保存しているかによっては `is_group=false` のまま届く場合があります。

`channels.imessage.groups` 配下に `chat_id` を明示的に設定すると、OpenClaw はそのスレッドを次の目的で「グループ」として扱います:

* セッション分離（個別の `agent:<agentId>:imessage:group:<chat_id>` セッションキー）
* グループに対する許可リスト / メンション制御の挙動

例:

```json5
{
  channels: {
    imessage: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15555550123"],
      groups: {
        "42": { "requireMention": false }
      }
    }
  }
}
```

特定のスレッド専用の分離された人格／モデルを用意したい場合に有用です（[Multi-agent routing](/ja/concepts/multi-agent) を参照）。ファイルシステムレベルでの分離については、[Sandboxing](/ja/gateway/sandboxing) を参照してください。

<div id="media-limits">
  ## メディアと制限
</div>

* `channels.imessage.includeAttachments` による添付ファイルのオプション取り込み。
* `channels.imessage.mediaMaxMb` によるメディアサイズ上限（MB）の設定。

<div id="limits">
  ## 制限
</div>

* 送信テキストは `channels.imessage.textChunkLimit`（デフォルト 4000）に従って分割されます。
* 改行単位でのオプション分割を行うには、`channels.imessage.chunkMode="newline"` を設定すると、長さによる分割の前に空行（段落境界）で分割されます。
* メディアのアップロードサイズは `channels.imessage.mediaMaxMb`（デフォルト 16）で上限が設定されています。

<div id="addressing-delivery-targets">
  ## アドレス指定 / 配送ターゲット
</div>

安定したルーティングには `chat_id` を優先して使用します。

* `chat_id:123` (推奨)
* `chat_guid:...`
* `chat_identifier:...`
* 直接指定するハンドル: `imessage:+1555` / `sms:+1555` / `user@example.com`

チャットを一覧表示:

```
imsg chats --limit 20
```

<div id="configuration-reference-imessage">
  ## 設定リファレンス (iMessage)
</div>

完全な設定: [Configuration](/ja/gateway/configuration)

プロバイダーオプション:

* `channels.imessage.enabled`: チャンネルの起動を有効/無効にする。
* `channels.imessage.cliPath`: `imsg` へのパス。
* `channels.imessage.dbPath`: Messages DB のパス。
* `channels.imessage.remoteHost`: `cliPath` がリモート Mac（例: `user@gateway-host`）を指している場合の、添付ファイル転送用 SCP の SSH ホスト。未設定の場合は SSH ラッパーから自動検出される。
* `channels.imessage.service`: `imessage | sms | auto`。
* `channels.imessage.region`: SMS のリージョン。
* `channels.imessage.dmPolicy`: `pairing | allowlist | open | disabled`（デフォルト: pairing）。
* `channels.imessage.allowFrom`: DM 許可リスト（ハンドル、メールアドレス、E.164 番号、または `chat_id:*`）。`open` の場合は `"*"` が必須。iMessage にはユーザー名がないため、ハンドルまたはチャットの宛先を使用する。
* `channels.imessage.groupPolicy`: `open | allowlist | disabled`（デフォルト: allowlist）。
* `channels.imessage.groupAllowFrom`: グループ送信者の許可リスト。
* `channels.imessage.historyLimit` / `channels.imessage.accounts.*.historyLimit`: コンテキストとして含めるグループメッセージの最大数（0 で無効）。
* `channels.imessage.dmHistoryLimit`: DM の履歴上限（ユーザーターン数）。ユーザー単位のオーバーライド: `channels.imessage.dms["<handle>"].historyLimit`。
* `channels.imessage.groups`: グループごとのデフォルト設定 + 許可リスト（グローバルデフォルトには `"*"` を使用する）。
* `channels.imessage.includeAttachments`: 添付ファイルをコンテキストに取り込む。
* `channels.imessage.mediaMaxMb`: 受信/送信メディアの上限（MB）。
* `channels.imessage.textChunkLimit`: 送信時のチャンクサイズ（文字数）。
* `channels.imessage.chunkMode`: `length`（デフォルト）または `newline`。`newline` の場合、長さによる分割の前に空行（段落境界）で分割する。

関連するグローバルオプション:

* `agents.list[].groupChat.mentionPatterns`（または `messages.groupChat.mentionPatterns`）。
* `messages.responsePrefix`。