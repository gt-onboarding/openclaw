---
title: Nostr
summary: "NIP-04 暗号化メッセージを用いる Nostr DM チャネル"
read_when:
  - Nostr 経由で OpenClaw が DM を受信できるようにしたいとき
  - 分散型メッセージングを設定しているとき
---

<div id="nostr">
  # Nostr
</div>

**ステータス:** オプションのプラグイン（デフォルトでは無効）。

Nostr は、分散型ソーシャルネットワーキング用のプロトコルです。このチャネルを有効にすると、OpenClaw は NIP-04 経由で暗号化されたダイレクトメッセージ（DM）を受信し、応答できるようになります。

<div id="install-on-demand">
  ## オンデマンドインストール
</div>

<div id="onboarding-recommended">
  ### オンボーディング（推奨）
</div>

- オンボーディングウィザード（`openclaw onboard`）および `openclaw channels add` は、オプションのチャネル用プラグインを一覧表示します。
- Nostr を選択すると、その場でプラグインをインストールするように求められます。

インストール時のデフォルト動作:

- **Dev チャネル + git checkout が利用可能な場合:** ローカルのプラグインパスを使用します。
- **Stable/Beta:** npm からダウンロードします。

プロンプトでは、この選択をいつでも上書きできます。

<div id="manual-install">
  ### 手動インストール
</div>

```bash
openclaw plugins install @openclaw/nostr
```

ローカルチェックアウトを使う（開発ワークフロー向け）:

```bash
openclaw plugins install --link <openclawへのパス>/extensions/nostr
```

プラグインをインストールまたは有効化した後は、Gateway を再起動してください。


<div id="quick-setup">
  ## クイックセットアップ
</div>

1. 必要に応じて Nostr のキーペアを生成します：

```bash
# nak の使用
nak key generate
```

2. 設定ファイルに追加する:

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}"
    }
  }
}
```

3. キーをエクスポートする:

```bash
export NOSTR_PRIVATE_KEY="nsec1..."
```

4. Gateway を再起動します。


<div id="configuration-reference">
  ## 設定リファレンス
</div>

| Key | Type | Default | Description |
| --- | --- | --- | --- |
| `privateKey` | string | required | `nsec` または 16 進数形式の秘密鍵 |
| `relays` | string[] | `['wss://relay.damus.io', 'wss://nos.lol']` | リレーの URL（WebSocket） |
| `dmPolicy` | string | `pairing` | DM アクセスポリシー |
| `allowFrom` | string[] | `[]` | 許可された送信元の公開鍵（pubkey） |
| `enabled` | boolean | `true` | チャンネルを有効/無効にする |
| `name` | string | - | 表示名 |
| `profile` | object | - | NIP-01 プロファイルメタデータ |

<div id="profile-metadata">
  ## プロフィールメタデータ
</div>

プロフィールデータは NIP-01 の `kind:0` イベントとして公開されます。Control UI (Channels -&gt; Nostr -&gt; Profile) から管理するか、config で直接設定できます。

例:

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "profile": {
        "name": "openclaw",
        "displayName": "OpenClaw",
        "about": "パーソナルアシスタントDMボット",
        "picture": "https://example.com/avatar.png",
        "banner": "https://example.com/banner.png",
        "website": "https://example.com",
        "nip05": "openclaw@example.com",
        "lud16": "openclaw@example.com"
      }
    }
  }
}
```

注意:

* プロフィール URL には必ず `https://` を使用してください。
* リレーからインポートするとフィールドがマージされ、ローカル側での上書きは保持されます。


<div id="access-control">
  ## アクセス制御
</div>

<div id="dm-policies">
  ### DM ポリシー
</div>

- **pairing** (デフォルト): 未知の送信者にはペアリングコードが表示されます。
- **allowlist**: `allowFrom` に含まれる pubkey のみが DM を送信できます。
- **open**: 任意のユーザーからの受信 DM を許可する公開モードです（`allowFrom: ["*"]` が必要）。
- **disabled**: 受信 DM を無視します。

<div id="allowlist-example">
  ### 許可リストの例
</div>

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "dmPolicy": "allowlist",
      "allowFrom": ["npub1abc...", "npub1xyz..."]
    }
  }
}
```


<div id="key-formats">
  ## キー形式
</div>

サポートされる形式:

- **秘密鍵:** `nsec...` または 64 文字の 16 進数文字列
- **公開鍵 (`allowFrom`):** `npub...` または 16 進数文字列

<div id="relays">
  ## リレー
</div>

デフォルト: `relay.damus.io` と `nos.lol`。

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "relays": [
        "wss://relay.damus.io",
        "wss://relay.primal.net",
        "wss://nostr.wine"
      ]
    }
  }
}
```

ヒント:

* 冗長性のために 2～3 個のリレーを使用してください。
* リレーを増やしすぎると、遅延や重複が発生しやすくなります。
* 有料リレーを利用すると、信頼性が向上する場合があります。
* ローカルリレーはテスト用途として問題ありません（`ws://localhost:7777`）。


<div id="protocol-support">
  ## プロトコル対応状況
</div>

| NIP | ステータス | 説明 |
| --- | --- | --- |
| NIP-01 | 対応済み | 基本的なイベント形式 + プロフィールメタデータ |
| NIP-04 | 対応済み | 暗号化された DM（`kind:4`） |
| NIP-17 | 対応予定 | Gift-wrapped 形式の DM |
| NIP-44 | 対応予定 | バージョン管理された暗号化 |

<div id="testing">
  ## テスト
</div>

<div id="local-relay">
  ### ローカルリレー
</div>

```bash
# strfry を起動
docker run -p 7777:7777 ghcr.io/hoytech/strfry
```

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "relays": ["ws://localhost:7777"]
    }
  }
}
```


<div id="manual-test">
  ### 手動テスト
</div>

1) ログからボットの pubkey（npub）を控えておきます。
2) Nostr クライアント（Damus、Amethyst など）を開きます。
3) ボットの pubkey 宛てに DM を送信します。
4) レスポンスを確認します。

<div id="troubleshooting">
  ## トラブルシューティング
</div>

<div id="not-receiving-messages">
  ### メッセージを受信できない
</div>

- 秘密鍵が有効か確認してください。
- リレー URL にアクセスでき、`wss://`（ローカルの場合は `ws://`）を使用していることを確認してください。
- `enabled` が `false` になっていないことを確認してください。
- リレー接続エラーについて Gateway のログを確認してください。

<div id="not-sending-responses">
  ### レスポンスが送信されない
</div>

- リレーが書き込みを受け付けていることを確認する。
- 外向き接続が確立できているか確認する。
- リレーのレート制限が発生していないか確認する。

<div id="duplicate-responses">
  ### 重複レスポンス
</div>

- 複数のリレーを使用している場合には想定される動作です。
- メッセージはイベント ID 単位で重複排除され、最初の配信のみが応答をトリガーします。

<div id="security">
  ## セキュリティ
</div>

- 秘密鍵を決してコミットしないでください。
- キーの管理には環境変数を使用してください。
- 本番環境のボットでは許可リストの利用を検討してください。

<div id="limitations-mvp">
  ## 制限事項（MVP）
</div>

- ダイレクトメッセージのみ対応（グループチャットは不可）。
- メディアの添付は不可。
- NIP-04 のみ対応（NIP-17 gift-wrap は今後対応予定）。