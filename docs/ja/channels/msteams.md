---
title: Msteams
summary: "Microsoft Teams ボットのサポート状況、機能、設定"
read_when:
  - MS Teams チャネル機能に取り組んでいるとき
---

<div id="microsoft-teams-plugin">
  # Microsoft Teams (プラグイン)
</div>

> 「この門をくぐる者は一切の希望を捨てよ。」

更新日：2026-01-21

ステータス：テキストと DM（ダイレクトメッセージ）の添付ファイルはサポートされています。チャネル／グループチャットでのファイル送信には、`sharePointSiteId` と Graph 権限が必要です（[グループチャットでファイルを送信する](#sending-files-in-group-chats)を参照）。投票は Adaptive Cards を通じて送信されます。

<div id="plugin-required">
  ## 必須プラグイン
</div>

Microsoft Teams はプラグインとして提供されており、コアインストールには同梱されていません。

**後方互換性のない変更 (2026.1.15):** MS Teams はコアから分離されました。利用する場合はプラグインをインストールする必要があります。

理由（背景）：コアインストールをより軽量に保ち、MS Teams の依存関係を独立して更新できるようにするためです。

CLI（npm レジストリ）経由でインストールしてください：

```bash
openclaw plugins install @openclaw/msteams
```

ローカルでのチェックアウト（Git リポジトリから実行する場合）:

```bash
openclaw plugins install ./extensions/msteams
```

configure / onboarding の際に Teams を選択し、Git のチェックアウトが検出された場合、
OpenClaw はローカルのインストールパスを自動で提示します。

詳細: [プラグイン](/ja/plugin)

<div id="quick-setup-beginner">
  ## クイックセットアップ（初心者向け）
</div>

1. Microsoft Teams プラグインをインストールします。
2. **Azure Bot**（App ID + client secret + tenant ID）を作成します。
3. それらの認証情報で OpenClaw を設定します。
4. `/api/messages`（デフォルトのポートは 3978）をパブリック URL またはトンネル経由で公開します。
5. Teams アプリパッケージをインストールし、Gateway を起動します。

最小構成:

```json5
{
  channels: {
    msteams: {
      enabled: true,
      appId: "<APP_ID>",
      appPassword: "<APP_PASSWORD>",
      tenantId: "<TENANT_ID>",
      webhook: { port: 3978, path: "/api/messages" }
    }
  }
}
```

注: グループチャットはデフォルトでブロックされています（`channels.msteams.groupPolicy: "allowlist"`）。グループへの返信を許可するには、`channels.msteams.groupAllowFrom` を設定するか（または `groupPolicy: "open"` を使用して、メンションされた場合に限り、任意のメンバーからのメッセージ受信を制限なしに許可します）。

<div id="goals">
  ## 目的
</div>

* Teams の DM、グループ チャット、またはチャネル経由で OpenClaw と会話できるようにする。
* ルーティングを決定的に保ち、返信は必ず受信元のチャネルに戻す。
* デフォルトでは安全なチャネル動作にし（別途設定しない限りメンションが必須）、予期しない投稿を防止する。

<div id="config-writes">
  ## 設定の書き込み
</div>

デフォルトでは、Microsoft Teams は `/config set|unset` によってトリガーされる設定更新を書き込むことが許可されています（`commands.config: true` が必要）。

無効化するには次を実行します:

```json5
{
  channels: { msteams: { configWrites: false } }
}
```

<div id="access-control-dms-groups">
  ## アクセス制御（DM とグループ）
</div>

**DM アクセス**

* デフォルト: `channels.msteams.dmPolicy = "pairing"`。承認されるまで未知の送信者は無視されます。
* `channels.msteams.allowFrom` には AAD オブジェクト ID、UPN、または表示名を指定できます。ウィザードは、必要な資格情報がある場合は Microsoft Graph 経由で名前を ID に解決します。

**グループ アクセス**

* デフォルト: `channels.msteams.groupPolicy = "allowlist"`（`groupAllowFrom` を追加しない限り許可されません）。未設定時のデフォルトを上書きするには `channels.defaults.groupPolicy` を使用します。
* `channels.msteams.groupAllowFrom` は、どの送信者がグループチャット／チャネルでトリガーできるかを制御します（指定がない場合は `channels.msteams.allowFrom` にフォールバックします）。
* 任意のメンバーを許可するには `groupPolicy: "open"` を設定します（デフォルトでは依然としてメンション付きメッセージに限定されます）。
* **どのチャネルも許可しない** 場合は、`channels.msteams.groupPolicy: "disabled"` を設定します。

例:

```json5
{
  channels: {
    msteams: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["user@org.com"]
    }
  }
}
```

**Teams + チャネル許可リスト**

* `channels.msteams.teams` 配下に Teams とチャネルを列挙して、グループ／チャネル返信のスコープを設定します。
* キーにはチーム ID またはチーム名を使えます。チャネルキーには conversation ID またはチャネル名を使えます。
* `groupPolicy="allowlist"` かつ Teams の許可リストが存在する場合、記載された Teams／チャネルのみが受け付けられます（メンション付きメッセージに限定）。
* 設定ウィザードは `Team/Channel` のエントリを受け取り、それらを保存します。
* 起動時に OpenClaw は、Teams／チャネルおよびユーザー許可リストの名前を ID に解決し（Graph の権限が許可している場合）、
  対応関係をログに記録します。解決できなかったエントリは入力された文字列のまま保持されます。

例:

```json5
{
  channels: {
    msteams: {
      groupPolicy: "allowlist",
      teams: {
        "My Team": {
          channels: {
            "General": { requireMention: true }
          }
        }
      }
    }
  }
}
```

<div id="how-it-works">
  ## 仕組み
</div>

1. Microsoft Teams プラグインをインストールします。
2. **Azure Bot**（App ID + secret + tenant ID）を作成します。
3. ボットを参照し、下記の RSC 権限を含む **Teams アプリ パッケージ**を作成します。
4. Teams アプリをチームに（または DM 用に個人用スコープで）アップロード／インストールします。
5. `~/.openclaw/openclaw.json`（または環境変数）で `msteams` を設定して Gateway を起動します。
6. Gateway は、デフォルトで `/api/messages` 上の Bot Framework の webhook トラフィックを待ち受けます。

<div id="azure-bot-setup-prerequisites">
  ## Azure Bot セットアップ（前提条件）
</div>

OpenClaw を構成する前に、Azure Bot リソースを作成する必要があります。

<div id="step-1-create-azure-bot">
  ### ステップ 1: Azure Bot を作成する
</div>

1. [Create Azure Bot](https://portal.azure.com/#create/Microsoft.AzureBot) にアクセスします
2. **Basics** タブで次を入力します:

   | 項目 | 値 |
   |-------|-------|
   | **Bot handle** | Bot 名。例: `openclaw-msteams`（一意である必要があります） |
   | **Subscription** | 利用する Azure サブスクリプションを選択 |
   | **Resource group** | 新規作成するか既存のものを使用 |
   | **Pricing tier** | 開発/テスト用途なら **Free** |
   | **Type of App** | **Single Tenant**（推奨 - 下記の注記を参照） |
   | **Creation type** | **Create new Microsoft App ID** を選択 |

> **非推奨のお知らせ:** 新しいマルチテナント Bot の作成は 2025-07-31 以降、非推奨です。新規 Bot には **Single Tenant** を使用してください。

3. **Review + create** → **Create** をクリックし、作成が完了するまで約 1〜2 分待ちます

<div id="step-2-get-credentials">
  ### ステップ 2: 資格情報を取得する
</div>

1. Azure Bot リソースを開き → **Configuration**
2. **Microsoft App ID** をコピー → これが `appId`
3. **Manage Password** をクリック → App Registration に移動
4. **Certificates &amp; secrets** → **New client secret** → **Value** をコピー → これが `appPassword`
5. **Overview** を開き → **Directory (tenant) ID** をコピー → これが `tenantId`

<div id="step-3-configure-messaging-endpoint">
  ### ステップ 3: メッセージングエンドポイントの設定
</div>

1. Azure Bot の **Configuration** 画面を開きます
2. **Messaging endpoint** を自分の webhook URL に設定します:
   * 本番環境: `https://your-domain.com/api/messages`
   * ローカル開発: トンネルを使用します（下記の [Local Development](#local-development-tunneling) を参照）

<div id="step-4-enable-teams-channel">
  ### ステップ 4: Teams チャネルを有効化する
</div>

1. Azure Bot の **Channels** に移動します
2. **Microsoft Teams** をクリック → Configure → Save をクリックします
3. 利用規約に同意します

<div id="local-development-tunneling">
  ## ローカル開発（トンネリング）
</div>

Teams からは `localhost` にはアクセスできません。ローカル開発時はトンネルを使用してください:

**オプション A: ngrok**

```bash
ngrok http 3978
# https URL をコピーします（例: https://abc123.ngrok.io）
# メッセージングエンドポイントを次のように設定します: https://abc123.ngrok.io/api/messages
```

**オプションB：Tailscale Funnel**

```bash
tailscale funnel 3978
# Tailscale funnel の URL をメッセージングエンドポイントとして使用する
```

<div id="teams-developer-portal-alternative">
  ## Teams Developer Portal（別の方法）
</div>

マニフェスト ZIP を手動で作成する代わりに、[Teams Developer Portal](https://dev.teams.microsoft.com/apps) を使用できます。

1. **+ New app** をクリックします
2. 基本情報（名前、説明、開発者情報）を入力します
3. **App features** → **Bot** に移動します
4. **Enter a bot ID manually** を選択し、Azure Bot App ID を貼り付けます
5. スコープを確認します: **Personal**、**Team**、**Group Chat**
6. **Distribute** → **Download app package** をクリックします
7. Teams 上で: **Apps** → **Manage your apps** → **Upload a custom app** を選択し、ZIP を指定します

この方法の方が、JSON マニフェストを手作業で編集するよりも簡単な場合が多いです。

<div id="testing-the-bot">
  ## Bot のテスト
</div>

**オプション A: Azure Web Chat（先に webhook を確認）**

1. Azure Portal → 対象の Azure Bot リソース → **Test in Web Chat**
2. メッセージを送信すると、応答が返ってくるはずです
3. Teams のセットアップ前に、webhook エンドポイントが正しく動作していることを確認できます

**オプション B: Teams（アプリインストール後）**

1. Teams アプリをインストールします（サイドロードまたは組織カタログ）
2. Teams で Bot を見つけて DM を送信します
3. Gateway のログで、受信したアクティビティを確認します

<div id="setup-minimal-text-only">
  ## セットアップ（最小限のテキストのみ）
</div>

1. **Microsoft Teams プラグインをインストールする**
   * npm から: `openclaw plugins install @openclaw/msteams`
   * ローカルのチェックアウトから: `openclaw plugins install ./extensions/msteams`

2. **Bot の登録**
   * Azure Bot（上記参照）を作成し、次を記録しておく:
     * App ID
     * Client secret（App password）
     * Tenant ID（single-tenant）

3. **Teams アプリのマニフェスト**
   * `bot` エントリを追加し、`botId = <App ID>` を指定する。
   * スコープ: `personal`, `team`, `groupChat`。
   * `supportsFiles: true`（personal scope でのファイル処理に必須）。
   * RSC permissions（RSC 権限）を追加する（下記参照）。
   * アイコンを作成する: `outline.png`（32x32）および `color.png`（192x192）。
   * これら 3 つのファイル `manifest.json`, `outline.png`, `color.png` を 1 つの zip アーカイブにまとめる。

4. **OpenClaw を設定する**

   ```json
   {
     "msteams": {
       "enabled": true,
       "appId": "<APP_ID>",
       "appPassword": "<APP_PASSWORD>",
       "tenantId": "<TENANT_ID>",
       "webhook": { "port": 3978, "path": "/api/messages" }
     }
   }
   ```

   構成キーの代わりに環境変数を使用することもできる:

   * `MSTEAMS_APP_ID`
   * `MSTEAMS_APP_PASSWORD`
   * `MSTEAMS_TENANT_ID`

5. **Bot エンドポイント**
   * Azure Bot の Messaging Endpoint を次の値に設定する:
     * `https://<host>:3978/api/messages`（または任意のパス / ポート）。

6. **Gateway を実行する**
   * プラグインがインストールされ、`msteams` 設定に認証情報が存在する場合、Teams チャンネルは自動的に開始される。

<div id="history-context">
  ## 履歴コンテキスト
</div>

* `channels.msteams.historyLimit` は、直近のチャネル / グループメッセージをいくつプロンプトに含めるかを制御します。
* 指定がない場合は `messages.groupChat.historyLimit` が使われます。無効化するには `0` を設定します（デフォルトは 50）。
* DM の履歴は `channels.msteams.dmHistoryLimit`（ユーザーターン数）で制限できます。ユーザーごとの個別オーバーライドは `channels.msteams.dms["<user_id>"].historyLimit` を使用します。

<div id="current-teams-rsc-permissions-manifest">
  ## 現在の Teams RSC 権限 (Manifest)
</div>

これは、Teams アプリの manifest に定義されている**既存の resourceSpecific 権限**です。アプリがインストールされているチーム／チャット内にのみ適用されます。

**チャネル向け (チーム スコープ):**

* `ChannelMessage.Read.Group` (Application) - @mention なしですべてのチャネル メッセージを受信
* `ChannelMessage.Send.Group` (Application)
* `Member.Read.Group` (Application)
* `Owner.Read.Group` (Application)
* `ChannelSettings.Read.Group` (Application)
* `TeamMember.Read.Group` (Application)
* `TeamSettings.Read.Group` (Application)

**グループ チャット向け:**

* `ChatMessage.Read.Chat` (Application) - @mention なしですべてのグループ チャット メッセージを受信

<div id="example-teams-manifest-redacted">
  ## Teams マニフェストの例（一部伏せ字）
</div>

必須フィールドのみを含む最小限かつ有効な例です。ID と URL を実際の値に置き換えてください。

```json
{
  "$schema": "https://developer.microsoft.com/en-us/json-schemas/teams/v1.23/MicrosoftTeams.schema.json",
  "manifestVersion": "1.23",
  "version": "1.0.0",
  "id": "00000000-0000-0000-0000-000000000000",
  "name": { "short": "OpenClaw" },
  "developer": {
    "name": "Your Org",
    "websiteUrl": "https://example.com",
    "privacyUrl": "https://example.com/privacy",
    "termsOfUseUrl": "https://example.com/terms"
  },
  "description": { "short": "OpenClaw in Teams", "full": "OpenClaw in Teams" },
  "icons": { "outline": "outline.png", "color": "color.png" },
  "accentColor": "#5B6DEF",
  "bots": [
    {
      "botId": "11111111-1111-1111-1111-111111111111",
      "scopes": ["personal", "team", "groupChat"],
      "isNotificationOnly": false,
      "supportsCalling": false,
      "supportsVideo": false,
      "supportsFiles": true
    }
  ],
  "webApplicationInfo": {
    "id": "11111111-1111-1111-1111-111111111111"
  },
  "authorization": {
    "permissions": {
      "resourceSpecific": [
        { "name": "ChannelMessage.Read.Group", "type": "Application" },
        { "name": "ChannelMessage.Send.Group", "type": "Application" },
        { "name": "Member.Read.Group", "type": "Application" },
        { "name": "Owner.Read.Group", "type": "Application" },
        { "name": "ChannelSettings.Read.Group", "type": "Application" },
        { "name": "TeamMember.Read.Group", "type": "Application" },
        { "name": "TeamSettings.Read.Group", "type": "Application" },
        { "name": "ChatMessage.Read.Chat", "type": "Application" }
      ]
    }
  }
}
```

<div id="manifest-caveats-must-have-fields">
  ### マニフェストの注意点（必須フィールド）
</div>

* `bots[].botId` は Azure Bot アプリ ID と**必ず**一致している必要があります。
* `webApplicationInfo.id` は Azure Bot アプリ ID と**必ず**一致している必要があります。
* `bots[].scopes` には、使用する予定のスコープ（`personal`、`team`、`groupChat`）を含める必要があります。
* personal スコープでファイルを扱うには、`bots[].supportsFiles: true` が必須です。
* チャネルのトラフィックを扱う場合は、`authorization.permissions.resourceSpecific` にチャネルの読み取り/送信権限を含める必要があります。

<div id="updating-an-existing-app">
  ### 既存アプリの更新
</div>

既にインストール済みの Teams アプリを更新する場合（例: RSC 権限を追加する場合）:

1. 新しい設定を反映するように `manifest.json` を更新する
2. **`version` フィールドをインクリメントする**（例: `1.0.0` → `1.1.0`）
3. マニフェストとアイコンをまとめて**再度 zip 圧縮する**（`manifest.json`, `outline.png`, `color.png`）
4. 新しい zip をアップロードする:
   * **オプション A (Teams 管理センター):** Teams 管理センター → Teams アプリ → アプリの管理 → 対象アプリを探す → 新しいバージョンをアップロード
   * **オプション B (サイドロード):** Teams → アプリ → 自分のアプリの管理 → カスタム アプリをアップロード
5. **チーム チャネル向け:** 新しい権限を反映させるため、各チームでアプリを再インストールする
6. キャッシュされたアプリ メタデータをクリアするため、**Teams を完全に終了してから再起動する**（ウィンドウを閉じるだけでは不十分）

<div id="capabilities-rsc-only-vs-graph">
  ## 機能: RSC のみ vs Graph
</div>

<div id="with-teams-rsc-only-app-installed-no-graph-api-permissions">
  ### **Teams RSC のみ** を使用する場合（アプリはインストール済み、Graph API 権限なし）
</div>

できること:

* チャネルメッセージの **テキスト** コンテンツを読み取る。
* チャネルメッセージの **テキスト** コンテンツを送信する。
* **個人（DM）** のファイル添付を受信する。

できないこと:

* チャネル／グループの **画像やファイルの内容**（ペイロードには HTML のスタブのみが含まれる）。
* SharePoint/OneDrive に保存されている添付ファイルのダウンロード。
* メッセージ履歴の読み取り（リアルタイムな webhook イベント以外）。

<div id="with-teams-rsc-microsoft-graph-application-permissions">
  ### **Teams RSC + Microsoft Graph アプリケーション権限**がある場合
</div>

次のことが可能になります:

* ホストされたコンテンツ（メッセージに貼り付けられた画像）のダウンロード
* SharePoint/OneDrive に保存されている添付ファイルのダウンロード
* Graph 経由でのチャネル/チャット メッセージ履歴の読み取り

<div id="rsc-vs-graph-api">
  ### RSC と Graph API の比較
</div>

| 機能 | RSC の権限 | Graph API |
|------------|-----------------|-----------|
| **リアルタイム メッセージ** | はい（webhook 経由） | いいえ（ポーリングのみ） |
| **過去のメッセージ** | いいえ | はい（履歴をクエリ可能） |
| **セットアップの複雑さ** | アプリ マニフェストのみ | 管理者の同意 + トークン フローが必要 |
| **オフラインで動作** | いいえ（常時稼働している必要あり） | はい（いつでもクエリ可能） |

**要点:** RSC はリアルタイム受信（リッスン）向け、Graph API は履歴アクセス向けです。オフライン中に届いたメッセージを後から取得して追いつくには、`ChannelMessage.Read.All` スコープ（管理者の同意が必要）を付与した Graph API が必要です。

<div id="graph-enabled-media-history-required-for-channels">
  ## Graph 対応メディア + 履歴（チャネルで必須）
</div>

**チャネル** で画像やファイルを扱う、または **メッセージ履歴 (message history)** を取得したい場合は、Microsoft Graph のアクセス許可を有効にし、管理者の同意を付与する必要があります。

1. Entra ID (Azure AD) の **App Registration (アプリの登録)** で、Microsoft Graph の **Application permissions (アプリケーションのアクセス許可)** を追加します:
   * `ChannelMessage.Read.All`（チャネルの添付ファイル + 履歴）
   * `Chat.Read.All` または `ChatMessage.Read.All`（グループ チャット）
2. テナントに対して **管理者の同意 (admin consent)** を付与します。
3. Teams アプリの **manifest version (マニフェスト バージョン)** を上げて再アップロードし、**Teams にアプリを再インストール**します。
4. キャッシュされたアプリ メタデータをクリアするため、**Teams を完全に終了してから再起動**します。

<div id="known-limitations">
  ## 既知の制限事項
</div>

<div id="webhook-timeouts">
  ### Webhook のタイムアウト
</div>

Teams は HTTP Webhook を通じてメッセージを配信します。処理に時間がかかりすぎる場合（例: LLM の応答が遅い場合）、次のような事象が発生する可能性があります:

* Gateway のタイムアウト
* Teams によるメッセージの再試行（重複メッセージの発生）
* 返信の破棄

OpenClaw はすばやくレスポンスを返し、その後に能動的に返信を送信することでこれに対処しますが、非常に遅い応答は依然として問題を引き起こす可能性があります。

<div id="formatting">
  ### 書式設定
</div>

Teams の Markdown は Slack や Discord よりも機能が限られています：

* 基本的な書式は利用できます：**太字**、*斜体*、`code`、リンク
* 複雑な Markdown（表、入れ子リストなど）は正しくレンダリングされない場合があります
* 投票や任意のカードを送信するための Adaptive Cards がサポートされています（後述）

<div id="configuration">
  ## 設定
</div>

主要な設定項目（共通のチャンネルパターンについては `/gateway/configuration` を参照）:

* `channels.msteams.enabled`: チャンネルの有効化/無効化。
* `channels.msteams.appId`, `channels.msteams.appPassword`, `channels.msteams.tenantId`: ボットの認証情報。
* `channels.msteams.webhook.port` (デフォルト `3978`)
* `channels.msteams.webhook.path` (デフォルト `/api/messages`)
* `channels.msteams.dmPolicy`: `pairing | allowlist | open | disabled`（デフォルト: pairing）
* `channels.msteams.allowFrom`: DM 用の許可リスト（AAD オブジェクト ID、UPN、または表示名）。Microsoft Graph にアクセス可能な場合、セットアップ時にウィザードが名前を ID に解決します。
* `channels.msteams.textChunkLimit`: 送信テキストのチャンクサイズ。
* `channels.msteams.chunkMode`: `length`（デフォルト）または `newline`。`newline` を指定すると、長さで分割する前に空行（段落境界）で分割します。
* `channels.msteams.mediaAllowHosts`: 受信添付ファイルのホスト用の許可リスト（デフォルトでは Microsoft/Teams ドメイン）。
* `channels.msteams.requireMention`: チャンネル/グループで @メンションを必須にする（デフォルト true）。
* `channels.msteams.replyStyle`: `thread | top-level`（[Reply Style](#reply-style-threads-vs-posts) を参照）。
* `channels.msteams.teams.<teamId>.replyStyle`: チーム単位の上書き設定。
* `channels.msteams.teams.<teamId>.requireMention`: チーム単位の上書き設定。
* `channels.msteams.teams.<teamId>.tools`: チーム単位のデフォルトツールポリシーの上書き設定（`allow`/`deny`/`alsoAllow`）。チャンネル単位の上書きがない場合に使用されます。
* `channels.msteams.teams.<teamId>.toolsBySender`: チーム単位の送信者別デフォルトツールポリシー上書き設定（`"*"` ワイルドカード対応）。
* `channels.msteams.teams.<teamId>.channels.<conversationId>.replyStyle`: チャンネル単位の上書き設定。
* `channels.msteams.teams.<teamId>.channels.<conversationId>.requireMention`: チャンネル単位の上書き設定。
* `channels.msteams.teams.<teamId>.channels.<conversationId>.tools`: チャンネル単位のツールポリシー上書き設定（`allow`/`deny`/`alsoAllow`）。
* `channels.msteams.teams.<teamId>.channels.<conversationId>.toolsBySender`: チャンネル単位の送信者別ツールポリシー上書き設定（`"*"` ワイルドカード対応）。
* `channels.msteams.sharePointSiteId`: グループチャット/チャンネルでのファイルアップロード用の SharePoint サイト ID（[Sending files in group chats](#sending-files-in-group-chats) を参照）。

<div id="routing-sessions">
  ## ルーティングとセッション
</div>

* セッションキーは標準的なエージェントのフォーマットに従います（[/concepts/session](/ja/concepts/session) を参照）：
  * ダイレクト メッセージはメインセッション（`agent:<agentId>:<mainKey>`）を共有します。
  * チャンネル／グループ メッセージは会話 ID を使用します：
    * `agent:<agentId>:msteams:channel:<conversationId>`
    * `agent:<agentId>:msteams:group:<conversationId>`

<div id="reply-style-threads-vs-posts">
  ## 返信スタイル: スレッド vs 投稿
</div>

Teams では、同じ基盤データモデルの上に、2 種類のチャネル UI スタイルが最近導入されました:

| Style                    | Description                         | Recommended `replyStyle` |
| ------------------------ | ----------------------------------- | ------------------------ |
| **Posts** (classic)      | メッセージはカードとして表示され、その下にスレッド返信がぶら下がる形式 | `thread` (default)       |
| **Threads** (Slack-like) | メッセージが Slack に近い形で、時系列に一列で流れていく形式   | `top-level`              |

**問題点:** Teams API は、そのチャネルがどの UI スタイルを使用しているかを提供していません。誤った `replyStyle` を使うと、次のような挙動になります:

* Threads スタイルのチャネルで `thread` → 返信が不自然にネストされて表示される
* Posts スタイルのチャネルで `top-level` → 返信がスレッド内ではなく、別個のトップレベル投稿として表示される

**解決策:** チャネルの設定に合わせて、チャネルごとに `replyStyle` を設定してください:

```json
{
  "msteams": {
    "replyStyle": "thread",
    "teams": {
      "19:abc...@thread.tacv2": {
        "channels": {
          "19:xyz...@thread.tacv2": {
            "replyStyle": "top-level"
          }
        }
      }
    }
  }
}
```

<div id="attachments-images">
  ## 添付ファイルと画像
</div>

**現在の制限事項:**

* **DM:** 画像およびファイル添付は、Teams ボットのファイル API 経由で動作します。
* **チャネル/グループ:** 添付ファイルは M365 ストレージ（SharePoint/OneDrive）に保存されます。Webhook ペイロードには実際のファイルバイトではなく、HTML スタブのみが含まれます。チャネルの添付ファイルをダウンロードするには **Graph API の権限が必要です**。

Graph API の権限がない場合、画像付きのチャネルメッセージはテキストのみとして受信されます（画像コンテンツにはボットからアクセスできません）。
デフォルトでは、OpenClaw は Microsoft/Teams のホスト名からのみメディアをダウンロードします。`channels.msteams.mediaAllowHosts` で設定を変更できます（任意のホストを許可するには `["*"]` を使用してください）。

<div id="sending-files-in-group-chats">
  ## グループチャットでのファイル送信
</div>

Bot は組み込みの FileConsentCard フローを使って DM でファイルを送信できます。ただし、**グループチャット／チャネルでファイルを送信する**場合は、追加のセットアップが必要です:

| コンテキスト | ファイルの送信方法 | 必要なセットアップ |
|---------|-------------------|--------------|
| **DMs** | FileConsentCard → ユーザーが承諾 → Bot がアップロード | 追加のセットアップなしでそのまま利用可能 |
| **グループチャット／チャネル** | SharePoint にアップロード → リンクを共有 | `sharePointSiteId` + Graph 権限が必要 |
| **画像（任意のコンテキスト）** | Base64 エンコードされたインライン | 追加のセットアップなしでそのまま利用可能 |

<div id="why-group-chats-need-sharepoint">
  ### なぜグループチャットには SharePoint が必要なのか
</div>

Bot には個人用 OneDrive ドライブがありません（`/me/drive` Graph API エンドポイントはアプリケーション ID では利用できません）。グループチャットやチャネルでファイルを送信するには、Bot はファイルを **SharePoint サイト** にアップロードし、共有リンクを作成します。

<div id="setup">
  ### セットアップ
</div>

1. Entra ID (Azure AD) → App Registration で **Graph API のアクセス許可を追加**:
   * `Sites.ReadWrite.All` (Application) - SharePoint へのファイルアップロードを許可
   * `Chat.Read.All` (Application) - 任意。ユーザー単位の共有リンクを有効化

2. テナントに対して **管理者の同意 (admin consent) を付与** します。

3. **SharePoint サイト ID を取得します:**
   ```bash
   # Graph Explorer または有効なトークンを使った curl で実行:
   curl -H "Authorization: Bearer $TOKEN" \
     "https://graph.microsoft.com/v1.0/sites/{hostname}:/{site-path}"

   # 例: "contoso.sharepoint.com/sites/BotFiles" にあるサイトの場合
   curl -H "Authorization: Bearer $TOKEN" \
     "https://graph.microsoft.com/v1.0/sites/contoso.sharepoint.com:/sites/BotFiles"

   # レスポンスには次のような項目が含まれる: "id": "contoso.sharepoint.com,guid1,guid2"
   ```

4. **OpenClaw を設定します:**
   ```json5
   {
     channels: {
       msteams: {
         // ... その他の設定 ...
         sharePointSiteId: "contoso.sharepoint.com,guid1,guid2"
       }
     }
   }
   ```

<div id="sharing-behavior">
  ### 共有動作
</div>

| Permission | 共有動作 |
|------------|------------------|
| `Sites.ReadWrite.All` のみ | 組織全体の共有リンク（組織内の誰でもアクセス可能） |
| `Sites.ReadWrite.All` + `Chat.Read.All` | ユーザー単位の共有リンク（チャットメンバーのみアクセス可能） |

ユーザー単位での共有の方が、チャット参加者だけがファイルにアクセスできるため、より安全です。`Chat.Read.All` 権限がない場合、ボットは組織全体の共有リンクにフォールバックします。

<div id="fallback-behavior">
  ### フォールバック動作
</div>

| シナリオ | 結果 |
|----------|--------|
| グループチャット + ファイル + `sharePointSiteId` が設定されている場合 | SharePoint にアップロードし、共有リンクを送信 |
| グループチャット + ファイル + `sharePointSiteId` が未設定の場合 | OneDrive へのアップロードを試行（失敗する場合があります）、テキストのみ送信 |
| 個人チャット + ファイル | FileConsentCard フローで処理（SharePoint なしでも動作） |
| 任意のコンテキスト + 画像 | Base64 エンコードされたインライン画像として送信（SharePoint なしでも動作） |

<div id="files-stored-location">
  ### ファイルの保存場所
</div>

アップロードされたファイルは、設定済みの SharePoint サイトの既定のドキュメント ライブラリ配下の `/OpenClawShared/` フォルダーに保存されます。

<div id="polls-adaptive-cards">
  ## 投票 (Adaptive Cards)
</div>

OpenClaw は、Teams の投票を Adaptive Cards として送信します (ネイティブな Teams の投票用 API は存在しません)。

* CLI: `openclaw message poll --channel msteams --target conversation:<id> ...`
* 投票は Gateway によって `~/.openclaw/msteams-polls.json` に記録されます。
* 投票を記録するには Gateway がオンラインの状態を維持している必要があります。
* 投票結果の概要は、まだ自動で投稿されません (必要に応じてストアファイルを確認してください)。

<div id="adaptive-cards-arbitrary">
  ## Adaptive Cards（任意）
</div>

`message` ツールまたは CLI を使用して、任意の Adaptive Card JSON を Teams のユーザーまたは会話に送信できます。

`card` パラメータには Adaptive Card の JSON オブジェクトを指定します。`card` が指定されている場合、メッセージ本文は省略可能です。

**エージェントツール:**

```json
{
  "action": "send",
  "channel": "msteams",
  "target": "user:<id>",
  "card": {
    "type": "AdaptiveCard",
    "version": "1.5",
    "body": [{"type": "TextBlock", "text": "Hello!"}]
  }
}
```

**CLI：**

```bash
openclaw message send --channel msteams \
  --target "conversation:19:abc...@thread.tacv2" \
  --card '{"type":"AdaptiveCard","version":"1.5","body":[{"type":"TextBlock","text":"Hello!"}]}'
```

カード スキーマとサンプルについては、[Adaptive Cards のドキュメント](https://adaptivecards.io/) を参照してください。ターゲット形式の詳細については、以下の [Target formats](#target-formats) を参照してください。

<div id="target-formats">
  ## ターゲット形式
</div>

MSTeams のターゲットは、ユーザーと会話を区別するためにプレフィックスを使用します：

| ターゲット種別          | 形式                               | 例                                            |
| ---------------- | -------------------------------- | -------------------------------------------- |
| ユーザー（ID 指定）      | `user:<aad-object-id>`           | `user:40a1a0ed-4ff2-4164-a219-55518990c197`  |
| ユーザー（名前指定）       | `user:<display-name>`            | `user:John Smith`（Graph API が必要）             |
| グループ/チャネル        | `conversation:<conversation-id>` | `conversation:19:abc123...@thread.tacv2`     |
| グループ/チャネル（生の ID） | `<conversation-id>`              | `19:abc123...@thread.tacv2`（`@thread` を含む場合） |

**CLI の例：**

```bash
# Send to a user by ID
openclaw message send --channel msteams --target "user:40a1a0ed-..." --message "Hello"

# 表示名でユーザーに送信(Graph APIルックアップをトリガー)
openclaw message send --channel msteams --target "user:John Smith" --message "Hello"

# Send to a group chat or channel
openclaw message send --channel msteams --target "conversation:19:abc...@thread.tacv2" --message "Hello"

# Send an Adaptive Card to a conversation
openclaw message send --channel msteams --target "conversation:19:abc...@thread.tacv2" \
  --card '{"type":"AdaptiveCard","version":"1.5","body":[{"type":"TextBlock","text":"Hello"}]}'
```

**エージェント用ツールの例:**

```json
{
  "action": "send",
  "channel": "msteams",
  "target": "user:John Smith",
  "message": "Hello!"
}
```

```json
{
  "action": "send",
  "channel": "msteams",
  "target": "conversation:19:abc...@thread.tacv2",
  "card": {"type": "AdaptiveCard", "version": "1.5", "body": [{"type": "TextBlock", "text": "Hello"}]}
}
```

注意: `user:` 接頭辞を付けない場合、名前はデフォルトでグループ／チーム宛てとして解決されます。表示名で特定のユーザーを指定する場合は、必ず `user:` を付けてください。

<div id="proactive-messaging">
  ## プロアクティブメッセージング
</div>

* プロアクティブメッセージの送信が可能になるのは、ユーザーと一度やり取りを行った**後**のみです。そのタイミングで会話の参照情報を保存するためです。
* `dmPolicy` と許可リストによる制御については `/gateway/configuration` を参照してください。

<div id="team-and-channel-ids-common-gotcha">
  ## チーム ID とチャネル ID（よくあるハマりどころ）
</div>

Teams の URL に含まれる `groupId` クエリ パラメータは、構成で使用するチーム ID では**ありません**。代わりに、URL のパス部分から ID を抽出してください。

**チーム URL:**

```
https://teams.microsoft.com/l/team/19%3ABk4j...%40thread.tacv2/conversations?groupId=...
                                    └────────────────────────────┘
                                    Team ID (これをURLデコードする)
```

**チャネル URL:**

```
https://teams.microsoft.com/l/channel/19%3A15bc...%40thread.tacv2/ChannelName?groupId=...
                                      └─────────────────────────┘
                                      Channel ID (URL-decode this)
```

**設定用:**

* Team ID = `/team/` の後ろのパスセグメント（URL デコード後。例: `19:Bk4j...@thread.tacv2`）
* Channel ID = `/channel/` の後ろのパスセグメント（URL デコード後）
* `groupId` クエリパラメーターは **無視** する

<div id="private-channels">
  ## プライベートチャネル
</div>

ボットはプライベートチャネルではサポートが限定的です:

| 機能 | 標準チャネル | プライベートチャネル |
|---------|-------------------|------------------|
| ボットのインストール | 可能 | 制限あり |
| リアルタイムメッセージ（Webhook） | 可能 | 動作しない場合あり |
| RSC 権限 | あり | 挙動が異なる場合あり |
| @メンション | 可能 | ボットにアクセス可能な場合 |
| Graph API による履歴 | 可能 | 可能（権限が必要） |

**プライベートチャネルで動作しない場合の回避策:**

1. ボットとのやり取りには標準チャネルを使用する
2. 個人チャット（DM）を使用する - ユーザーは常にボットに直接メッセージを送信できる
3. 履歴アクセスには Graph API を使用する（`ChannelMessage.Read.All` が必要）

<div id="troubleshooting">
  ## トラブルシューティング
</div>

<div id="common-issues">
  ### よくある問題
</div>

* **チャネル内で画像が表示されない:** Graph の権限または管理者による同意が不足しています。Teams アプリを再インストールし、Teams を完全に終了してから再起動してください。
* **チャネルでレスポンスが返ってこない:** デフォルトではメンションが必須です。`channels.msteams.requireMention=false` を設定するか、チーム／チャネルごとに設定してください。
* **バージョン不一致（Teams に古い manifest が表示される）:** アプリを削除してから再追加し、Teams を完全に終了して再起動して更新を反映させてください。
* **webhook から 401 Unauthorized が返る:** Azure JWT なしで手動テストしている場合に想定される動作です。エンドポイントには到達できているものの、認証に失敗しています。正しくテストするには Azure Web Chat を使用してください。

<div id="manifest-upload-errors">
  ### マニフェストのアップロードエラー
</div>

* **&quot;Icon file cannot be empty&quot;:** マニフェストが参照しているアイコンファイルが 0 バイトです。有効な PNG アイコンを作成してください（`outline.png` は 32x32、`color.png` は 192x192）。
* **&quot;webApplicationInfo.Id already in use&quot;:** アプリが他のチーム／チャットにまだインストールされています。先に該当箇所でアンインストールするか、伝播が完了するまで 5〜10 分ほど待機してください。
* **アップロード時に &quot;Something went wrong&quot; が表示される:** https://admin.teams.microsoft.com からアップロードし、ブラウザの DevTools（F12）→ Network タブを開き、実際のエラー内容をレスポンスボディで確認してください。
* **サイドロードが失敗する:** &quot;Upload a custom app&quot; ではなく &quot;Upload an app to your org&#39;s app catalog&quot; を試してください。こちらの方法ではサイドロード制限を回避できる場合があります。

<div id="rsc-permissions-not-working">
  ### RSC アクセス許可が機能しない場合
</div>

1. `webApplicationInfo.id` がボットの App ID と完全に一致していることを確認する
2. アプリを再アップロードし、チーム／チャットに再インストールする
3. 組織の管理者が RSC アクセス許可をブロックしていないか確認する
4. 正しいスコープを使用していることを確認する: Teams のチームには `ChannelMessage.Read.Group`、グループ チャットには `ChatMessage.Read.Chat` を使用する

<div id="references">
  ## 参考資料
</div>

* [Create Azure Bot](https://learn.microsoft.com/en-us/azure/bot-service/bot-service-quickstart-registration) - Azure Bot のセットアップガイド
* [Teams Developer Portal](https://dev.teams.microsoft.com/apps) - Teams アプリの作成／管理
* [Teams app manifest schema](https://learn.microsoft.com/en-us/microsoftteams/platform/resources/schema/manifest-schema)
* [Receive channel messages with RSC](https://learn.microsoft.com/en-us/microsoftteams/platform/bots/how-to/conversations/channel-messages-with-rsc)
* [RSC permissions reference](https://learn.microsoft.com/en-us/microsoftteams/platform/graph-api/rsc/resource-specific-consent)
* [Teams bot file handling](https://learn.microsoft.com/en-us/microsoftteams/platform/bots/how-to/bots-filesv4) (チャネル／グループの場合は Graph が必要)
* [Proactive messaging](https://learn.microsoft.com/en-us/microsoftteams/platform/bots/how-to/conversations/send-proactive-messages)