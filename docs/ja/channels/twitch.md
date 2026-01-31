---
title: Twitch
summary: "Twitch チャットボットの設定とセットアップ"
read_when:
  - OpenClaw の Twitch チャット連携を設定するとき
---

<div id="twitch-plugin">
  # Twitch（プラグイン）
</div>

IRC 接続を介した Twitch チャットのサポートを提供します。OpenClaw は Twitch ユーザー（ボットアカウント）として接続し、チャンネル内のメッセージを受信・送信します。

<div id="plugin-required">
  ## 必要なプラグイン
</div>

Twitch はプラグインとして提供されており、コアインストールには含まれていません。

CLI（npm レジストリ）からインストールします:

```bash
openclaw plugins install @openclaw/twitch
```

ローカルチェックアウト（Git リポジトリから実行する場合）:

```bash
openclaw plugins install ./extensions/twitch
```

詳細: [プラグイン](/ja/plugin)

<div id="quick-setup-beginner">
  ## クイックセットアップ（初心者向け）
</div>

1. ボット用の専用 Twitch アカウントを作成します（既存アカウントを使っても構いません）。
2. 認証情報を生成します: [Twitch Token Generator](https://twitchtokengenerator.com/)
   * **Bot Token** を選択
   * `chat:read` と `chat:write` のスコープが選択されていることを確認
   * **Client ID** と **Access Token** をコピー
3. 自分の Twitch ユーザー ID を取得します: https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/
4. トークンを設定します:
   * 環境変数: `OPENCLAW_TWITCH_ACCESS_TOKEN=...`（デフォルトアカウントのみ）
   * または設定: `channels.twitch.accessToken`
   * 両方設定されている場合、設定ファイルが優先されます（環境変数はデフォルトアカウント専用のフォールバック）。
5. Gateway を起動します。

**⚠️ 重要:** 不正なユーザーがボットをトリガーできないように、アクセス制御（`allowFrom` または `allowedRoles`）を追加してください。`requireMention` のデフォルト値は `true` です。

最小構成:

```json5
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw",              // Bot's Twitch account
      accessToken: "oauth:abc123...",    // OAuth Access Token (or use OPENCLAW_TWITCH_ACCESS_TOKEN env var)
      clientId: "xyz789...",             // Client ID from Token Generator
      channel: "vevisk",                 // Which Twitch channel's chat to join (required)
      allowFrom: ["123456789"]           // （推奨）自分のTwitchユーザーIDのみ - https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/ から取得
    }
  }
}
```

<div id="what-it-is">
  ## これは何か
</div>

* Gateway が所有する Twitch チャンネル。
* 決定的なルーティング: 応答は必ず Twitch に返される。
* 各アカウントは、分離されたセッションキー `agent:<agentId>:twitch:<accountName>` にマッピングされる。
* `username` はボットのアカウント（認証を行う側）、`channel` は参加するチャットルームのチャンネル名。

<div id="setup-detailed">
  ## セットアップ（詳細手順）
</div>

<div id="generate-credentials">
  ### 認証情報を生成する
</div>

[Twitch Token Generator](https://twitchtokengenerator.com/) を使用します:

* **Bot Token** を選択する
* `chat:read` と `chat:write` のスコープが選択されていることを確認する
* **Client ID** と **Access Token** をコピーする

手動でアプリを登録する必要はありません。トークンは数時間で失効します。

<div id="configure-the-bot">
  ### ボットを設定する
</div>

**環境変数（デフォルトアカウントのみ）：**

```bash
OPENCLAW_TWITCH_ACCESS_TOKEN=oauth:abc123...
```

**または設定：**

```json5
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw",
      accessToken: "oauth:abc123...",
      clientId: "xyz789...",
      channel: "vevisk"
    }
  }
}
```

env と config の両方を指定した場合は、config が優先されます。

<div id="access-control-recommended">
  ### アクセス制御（推奨）
</div>

```json5
{
  channels: {
    twitch: {
      allowFrom: ["123456789"],       // (推奨) 自分のTwitchユーザーIDのみ
      allowedRoles: ["moderator"]     // Or restrict to roles
    }
  }
}
```

**利用可能なロール:** `"moderator"`, `"owner"`, `"vip"`, `"subscriber"`, `"all"`。

**なぜユーザー ID なのか？** ユーザー名は変更できるため、なりすましが可能になります。ユーザー ID は変更されません。

Twitch のユーザー ID を確認する: https://www.streamweasels.com/tools/convert-twitch-username-%20to-user-id/ （Twitch のユーザー名を ID に変換）

<div id="token-refresh-optional">
  ## トークンの自動更新（任意）
</div>

[Twitch Token Generator](https://twitchtokengenerator.com/) から取得したトークンは自動更新できません。有効期限が切れたら再生成してください。

トークンを自動更新するには、[Twitch Developer Console](https://dev.twitch.tv/console) で独自の Twitch アプリケーションを作成し、設定に追加します。

```json5
{
  channels: {
    twitch: {
      clientSecret: "your_client_secret",
      refreshToken: "your_refresh_token"
    }
  }
}
```

ボットは有効期限切れ前にトークンを自動更新し、その更新イベントをログに記録します。

<div id="multi-account-support">
  ## 複数アカウント対応
</div>

`channels.twitch.accounts` を使用し、アカウントごとのトークンを設定します。共通パターンについては [`gateway/configuration`](/ja/gateway/configuration) を参照してください。

例（1つのボットアカウントを2つのチャンネルで使用する場合）:

```json5
{
  channels: {
    twitch: {
      accounts: {
        channel1: {
          username: "openclaw",
          accessToken: "oauth:abc123...",
          clientId: "xyz789...",
          channel: "vevisk"
        },
        channel2: {
          username: "openclaw",
          accessToken: "oauth:def456...",
          clientId: "uvw012...",
          channel: "secondchannel"
        }
      }
    }
  }
}
```

**注意:** 各アカウントは専用のトークンを個別に用意する必要があります（チャネルごとに 1 つのトークン）。

<div id="access-control">
  ## アクセス制御
</div>

<div id="role-based-restrictions">
  ### ロールベースの制限
</div>

```json5
{
  channels: {
    twitch: {
      accounts: {
        default: {
          allowedRoles: ["moderator", "vip"]
        }
      }
    }
  }
}
```

<div id="allowlist-by-user-id-most-secure">
  ### ユーザーIDベースの許可リスト（最も安全）
</div>

```json5
{
  channels: {
    twitch: {
      accounts: {
        default: {
          allowFrom: ["123456789", "987654321"]
        }
      }
    }
  }
}
```

<div id="combined-allowlist-roles">
  ### 許可リストとロールの併用
</div>

`allowFrom` に含まれるユーザーは、ロールチェックをスキップします：

```json5
{
  channels: {
    twitch: {
      accounts: {
        default: {
          allowFrom: ["123456789"],
          allowedRoles: ["moderator"]
        }
      }
    }
  }
}
```

<div id="disable-mention-requirement">
  ### @メンション必須を無効化する
</div>

デフォルトでは、`requireMention` は `true` です。これを無効化して、すべてのメッセージに応答するには次のようにします:

```json5
{
  channels: {
    twitch: {
      accounts: {
        default: {
          requireMention: false
        }
      }
    }
  }
}
```

<div id="troubleshooting">
  ## トラブルシューティング
</div>

まず、診断コマンドを実行してください。

```bash
openclaw doctor
openclaw channels status --probe
```

<div id="bot-doesnt-respond-to-messages">
  ### Bot がメッセージに反応しない
</div>

**アクセス制御を確認:** テストのため、一時的に `allowedRoles: ["all"]` を設定してください。

**Bot がチャンネルにいるか確認:** Bot が `channel` で指定したチャンネルに参加していることを確認してください。

<div id="token-issues">
  ### トークン関連の問題
</div>

**「Failed to connect」または認証エラーが発生する場合:**

* `accessToken` が OAuth のアクセストークン値であることを確認する（通常は `oauth:` 接頭辞で始まる）
* トークンに `chat:read` および `chat:write` のスコープが付与されていることを確認する
* トークンリフレッシュを使用している場合は、`clientSecret` と `refreshToken` が設定されていることを確認する

<div id="token-refresh-not-working">
  ### トークンの更新が動作しない
</div>

**更新イベントのログを確認する：**

```
Using env token source for mybot
Access token refreshed for user 123456 (expires in 14400s)
```

「token refresh disabled (no refresh token)」と表示された場合は、次を確認してください:

* `clientSecret` が指定されていることを確認してください
* `refreshToken` が指定されていることを確認してください

<div id="config">
  ## Config
</div>

**アカウント設定:**

* `username` - Bot のユーザー名
* `accessToken` - `chat:read` と `chat:write` を持つ OAuth アクセストークン
* `clientId` - Twitch Client ID（Token Generator または自分のアプリから取得）
* `channel` - 参加するチャンネル（必須）
* `enabled` - このアカウントを有効にする（デフォルト: `true`）
* `clientSecret` - 任意: トークン自動更新用
* `refreshToken` - 任意: トークン自動更新用
* `expiresIn` - トークンの有効期限（秒）
* `obtainmentTimestamp` - トークン取得時刻
* `allowFrom` - ユーザー ID の許可リスト
* `allowedRoles` - ロールベースのアクセス制御（`"moderator" | "owner" | "vip" | "subscriber" | "all"`）
* `requireMention` - @メンションを必須にする（デフォルト: `true`）

**プロバイダーオプション:**

* `channels.twitch.enabled` - チャンネルを起動するかどうかを有効化/無効化
* `channels.twitch.username` - Bot のユーザー名（単一アカウント用の簡易設定）
* `channels.twitch.accessToken` - OAuth アクセストークン（単一アカウント用の簡易設定）
* `channels.twitch.clientId` - Twitch Client ID（単一アカウント用の簡易設定）
* `channels.twitch.channel` - 参加するチャンネル（単一アカウント用の簡易設定）
* `channels.twitch.accounts.<accountName>` - 複数アカウント設定（上記のすべてのアカウントフィールド）

完全な例:

```json5
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw",
      accessToken: "oauth:abc123...",
      clientId: "xyz789...",
      channel: "vevisk",
      clientSecret: "secret123...",
      refreshToken: "refresh456...",
      allowFrom: ["123456789"],
      allowedRoles: ["moderator", "vip"],
      accounts: {
        default: {
          username: "mybot",
          accessToken: "oauth:abc123...",
          clientId: "xyz789...",
          channel: "your_channel",
          enabled: true,
          clientSecret: "secret123...",
          refreshToken: "refresh456...",
          expiresIn: 14400,
          obtainmentTimestamp: 1706092800000,
          allowFrom: ["123456789", "987654321"],
          allowedRoles: ["moderator"]
        }
      }
    }
  }
}
```

<div id="tool-actions">
  ## ツールアクション
</div>

エージェントは次のアクションを指定して `twitch` を呼び出せます:

* `send` - チャンネルにメッセージを送信

例:

```json5
{
  "action": "twitch",
  "params": {
    "message": "Hello Twitch!",
    "to": "#mychannel"
  }
}
```

<div id="safety-ops">
  ## セキュリティと運用
</div>

* **トークンはパスワードと同様に扱う** - トークンを git にコミットしない
* **長時間稼働するボットには自動トークン更新を使用する**
* **アクセス制御にはユーザー名ではなくユーザー ID の許可リストを使用する**
* **ログを監視して** トークン更新イベントと接続ステータスを確認する
* **トークンのスコープは最小限にする** - `chat:read` と `chat:write` のみを要求する
* **問題が解決しない場合**: 他のプロセスがセッションを占有していないことを確認したうえで Gateway を再起動する

<div id="limits">
  ## 制限
</div>

* メッセージごとに**500文字**（単語境界で自動的にチャンク分割される）
* チャンク分割の前に Markdown は除去される
* 独自のレート制限はなし（Twitch の組み込みレート制限のみを使用）