---
title: Tlon
summary: "Tlon/Urbit のサポート状況、機能、設定方法"
read_when:
  - Tlon/Urbit チャンネル機能の実装や作業をしているとき
---

<div id="tlon-plugin">
  # Tlon (plugin)
</div>

Tlon は Urbit 上に構築された分散型メッセンジャーです。OpenClaw は Urbit シップに接続し、
DM とグループチャットメッセージに応答できます。グループへの返信にはデフォルトで @ メンションが必要で、
さらに許可リストによって制限できます。

ステータス: プラグイン経由でサポートされています。DM、グループメンション、スレッド返信、およびメディアのテキストのみフォールバック
（キャプションに URL を付加）が可能です。リアクション、投票、ネイティブメディアのアップロードはサポートされていません。

<div id="plugin-required">
  ## 必要なプラグイン
</div>

Tlon はプラグインとして提供されており、コアインストールには含まれていません。

CLI（npm レジストリ）経由でインストールしてください：

```bash
openclaw plugins install @openclaw/tlon
```

ローカルでのチェックアウト（Git リポジトリから実行する場合）:

```bash
openclaw plugins install ./extensions/tlon
```

詳細: [プラグイン](/ja/plugin)

<div id="setup">
  ## セットアップ
</div>

1. Tlon プラグインをインストールします。
2. ship の URL とログインコードを用意します。
3. `channels.tlon` を設定します。
4. Gateway を再起動します。
5. ボットに DM を送るか、グループチャンネルでメンションします。

最小構成（単一アカウント）:

```json5
{
  channels: {
    tlon: {
      enabled: true,
      ship: "~sampel-palnet",
      url: "https://your-ship-host",
      code: "lidlut-tabwed-pillex-ridrup"
    }
  }
}
```

<div id="group-channels">
  ## グループチャンネル
</div>

自動検出はデフォルトで有効です。チャンネルを手動でピン留めすることもできます。

```json5
{
  channels: {
    tlon: {
      groupChannels: [
        "chat/~host-ship/general",
        "chat/~host-ship/support"
      ]
    }
  }
}
```

自動検出を無効にする：

```json5
{
  channels: {
    tlon: {
      autoDiscoverChannels: false
    }
  }
}
```

<div id="access-control">
  ## アクセス制御
</div>

DM 許可リスト（空の場合はすべて許可）:

```json5
{
  channels: {
    tlon: {
      dmAllowlist: ["~zod", "~nec"]
    }
  }
}
```

グループ認可（デフォルトで制限される）:

```json5
{
  channels: {
    tlon: {
      defaultAuthorizedShips: ["~zod"],
      authorization: {
        channelRules: {
          "chat/~host-ship/general": {
            mode: "restricted",
            allowedShips: ["~zod", "~nec"]
          },
          "chat/~host-ship/announcements": {
            mode: "open"
          }
        }
      }
    }
  }
}
```

<div id="delivery-targets-clicron">
  ## 配信ターゲット（CLI/cron）
</div>

`openclaw message send` または cron 経由の配信で、次の値を指定します:

* DM: `~sampel-palnet` または `dm/~sampel-palnet`
* グループ: `chat/~host-ship/channel` または `group:~host-ship/channel`

<div id="notes">
  ## 注意事項
</div>

* グループへの返信には、対象へのメンション（例: `~your-bot-ship`）が必要です。
* スレッド返信: 受信メッセージがスレッド内にある場合、OpenClaw はそのスレッド内で返信します。
* メディア: `sendMedia` は、テキスト + URL にフォールバックします（ネイティブアップロードは行われません）。