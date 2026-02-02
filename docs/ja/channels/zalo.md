---
title: Zalo
summary: "Zalo ボットのサポート状況、機能、および設定"
read_when:
  - Zalo の機能や Webhook を扱っているとき
---

<div id="zalo-bot-api">
  # Zalo (Bot API)
</div>

ステータス: 実験的。現在はダイレクトメッセージのみ対応。Zalo のドキュメントによれば、グループ対応は今後追加予定。

<div id="plugin-required">
  ## 必要なプラグイン
</div>

Zalo はプラグインとして提供されており、コアのインストールには含まれていません。

* CLI からインストール: `openclaw plugins install @openclaw/zalo`
* または、オンボーディング時に **Zalo** を選択し、表示されるインストール確認を承認する
* 詳細: [プラグイン](/ja/plugin)

<div id="quick-setup-beginner">
  ## クイックセットアップ（初心者向け）
</div>

1. Zalo プラグインをインストールします:
   * ソースコードのチェックアウトから: `openclaw plugins install ./extensions/zalo`
   * npm から（公開されている場合）: `openclaw plugins install @openclaw/zalo`
   * またはオンボーディングで **Zalo** を選択し、インストールプロンプトを確認してインストールを承認します
2. トークンを設定します:
   * 環境変数: `ZALO_BOT_TOKEN=...`
   * または設定: `channels.zalo.botToken: "..."`
3. Gateway を再起動します（またはオンボーディングを完了します）。
4. DM アクセスはデフォルトでペアリング制です。最初のコンタクト時にペアリングコードを承認してください。

最小構成:

```json5
{
  channels: {
    zalo: {
      enabled: true,
      botToken: "12345689:abc-xyz",
      dmPolicy: "pairing"
    }
  }
}
```

<div id="what-it-is">
  ## これは何か
</div>

Zalo はベトナム向けのメッセージングアプリです。その Bot API を使うことで、Gateway は 1:1 の個別会話用ボットを実行できます。
Zalo に確実にルーティングを戻したいサポート用途や通知用途に適しています。

* Gateway が所有する Zalo Bot API チャンネル。
* 決定的なルーティング: 返信は必ず Zalo に戻り、モデルがチャンネルを選択することはありません。
* DM はエージェントのメインセッションを共有します。
* グループはまだサポートされていません (Zalo のドキュメントでは「coming soon」とされています)。

<div id="setup-fast-path">
  ## セットアップ（クイックパス）
</div>

<div id="1-create-a-bot-token-zalo-bot-platform">
  ### 1) ボットトークンを作成する (Zalo Bot Platform)
</div>

1. **https://bot.zaloplatforms.com** にアクセスしてサインインします。
2. 新しいボットを作成し、設定を行います。
3. ボットトークン（形式: `12345689:abc-xyz`）をコピーします。

<div id="2-configure-the-token-env-or-config">
  ### 2) トークンを設定する（env または config）
</div>

例：

```json5
{
  channels: {
    zalo: {
      enabled: true,
      botToken: "12345689:abc-xyz",
      dmPolicy: "pairing"
    }
  }
}
```

環境変数オプション: `ZALO_BOT_TOKEN=...`（デフォルトアカウントでのみ動作します）。

マルチアカウント対応: アカウントごとのトークンと任意の `name` を `channels.zalo.accounts` で指定します。

3. Gateway を再起動します。トークンが（環境変数または設定から）取得できると Zalo が起動します。
4. DM アクセスはデフォルトでペアリングです。ボットに初めて連絡があったときにコードを承認してください。

<div id="how-it-works-behavior">
  ## 動作の仕組み（挙動）
</div>

* 受信メッセージは、メディアのプレースホルダーを含む共通チャネル用エンベロープに正規化されます。
* 返信は常に同じ Zalo チャットに返送されます。
* デフォルトではロングポーリングを使用しますが、`channels.zalo.webhookUrl` を設定することで webhook モードも利用できます。

<div id="limits">
  ## 制限
</div>

* 送信テキストは 2000 文字単位で分割されます（Zalo API の制限）。
* メディアのダウンロード／アップロードは `channels.zalo.mediaMaxMb`（デフォルト 5）で上限が設定されています。
* 2000 文字制限によりストリーミングの有用性が低いため、ストリーミングはデフォルトで無効になっています。

<div id="access-control-dms">
  ## DM のアクセス制御
</div>

<div id="dm-access">
  ### DM アクセス
</div>

* デフォルト: `channels.zalo.dmPolicy = "pairing"`。不明な送信者にはペアリングコードが送信され、承認されるまでメッセージは無視されます（コードの有効期限は 1 時間です）。
* 承認方法:
  * `openclaw pairing list zalo`
  * `openclaw pairing approve zalo <CODE>`
* ペアリングはデフォルトのトークン交換方式です。詳細は [Pairing](/ja/start/pairing) を参照してください。
* `channels.zalo.allowFrom` は数値のユーザー ID を受け付けます（ユーザー名による検索はできません）。

<div id="long-polling-vs-webhook">
  ## ロングポーリング vs Webhook
</div>

* デフォルト: ロングポーリング（公開 URL は不要）。
* Webhook モード: `channels.zalo.webhookUrl` と `channels.zalo.webhookSecret` を設定する。
  * Webhook シークレットは 8〜256 文字でなければならない。
  * Webhook URL は HTTPS を使用しなければならない。
  * Zalo は検証用に、`X-Bot-Api-Secret-Token` ヘッダー付きでイベントを送信する。
  * Gateway の HTTP サーバーは `channels.zalo.webhookPath` で Webhook リクエストを処理する（デフォルトは Webhook URL と同じパス）。

**注:** Zalo API ドキュメントによると、getUpdates（ポーリング）と Webhook は排他的であり、同時には利用できない。

<div id="supported-message-types">
  ## サポートされているメッセージタイプ
</div>

* **テキストメッセージ**: 最大 2000 文字ごとに分割して送受信をフルサポート。
* **画像メッセージ**: 受信画像のダウンロードと処理を行い、`sendPhoto` で画像を送信。
* **ステッカー**: ログには記録されるが、完全には処理されない（エージェントからの応答なし）。
* **サポート対象外のタイプ**: ログには記録される（例: 保護されたユーザーからのメッセージ）。

<div id="capabilities">
  ## 機能
</div>

| 機能 | ステータス |
|---------|--------|
| ダイレクトメッセージ | ✅ サポート |
| グループ | ❌ 近日サポート予定（Zalo のドキュメントに基づく） |
| メディア（画像） | ✅ サポート |
| リアクション | ❌ 未サポート |
| スレッド | ❌ 未サポート |
| 投票 | ❌ 未サポート |
| ネイティブコマンド | ❌ 未サポート |
| ストリーミング | ⚠️ ブロック（2000 文字上限） |

<div id="delivery-targets-clicron">
  ## 配信対象（CLI/cron）
</div>

* 配信先としてチャットIDを指定します。
* 例: `openclaw message send --channel zalo --target 123456789 --message "hi"`。

<div id="troubleshooting">
  ## トラブルシューティング
</div>

**Bot が応答しない場合:**

* トークンが有効か確認する: `openclaw channels status --probe`
* 送信者が承認されていることを確認する（ペアリングまたは allowFrom）
* Gateway のログを確認する: `openclaw logs --follow`

**Webhook がイベントを受信しない場合:**

* Webhook URL が HTTPS を使用していることを確認する
* シークレットトークンが 8～256 文字であることを確認する
* 設定されたパスで Gateway の HTTP エンドポイントに到達できることを確認する
* `getUpdates` ポーリングが実行されていないことを確認する（両者は排他的で、同時には使用できません）

<div id="configuration-reference-zalo">
  ## 設定リファレンス（Zalo）
</div>

完全な設定: [Configuration](/ja/gateway/configuration)

プロバイダーオプション:

* `channels.zalo.enabled`: チャンネルの起動を有効化/無効化します。
* `channels.zalo.botToken`: Zalo Bot Platform から取得したボットトークン。
* `channels.zalo.tokenFile`: ファイルパスからトークンを読み込みます。
* `channels.zalo.dmPolicy`: `pairing | allowlist | open | disabled`（デフォルト: `pairing`）。
* `channels.zalo.allowFrom`: DM 許可リスト（ユーザー ID）。`open` の場合は `"*"` が必要です。ウィザードで数値 ID の入力を求められます。
* `channels.zalo.mediaMaxMb`: 受信/送信メディアの上限（MB、デフォルト 5）。
* `channels.zalo.webhookUrl`: webhook モードを有効化します（HTTPS 必須）。
* `channels.zalo.webhookSecret`: webhook シークレット（8〜256文字）。
* `channels.zalo.webhookPath`: Gateway の HTTP サーバー上の webhook パス。
* `channels.zalo.proxy`: API リクエスト用のプロキシ URL。

マルチアカウントオプション:

* `channels.zalo.accounts.<id>.botToken`: アカウントごとのトークン。
* `channels.zalo.accounts.<id>.tokenFile`: アカウントごとのトークンファイル。
* `channels.zalo.accounts.<id>.name`: 表示名。
* `channels.zalo.accounts.<id>.enabled`: アカウントの有効化/無効化。
* `channels.zalo.accounts.<id>.dmPolicy`: アカウントごとの DM ポリシー。
* `channels.zalo.accounts.<id>.allowFrom`: アカウントごとの許可リスト。
* `channels.zalo.accounts.<id>.webhookUrl`: アカウントごとの webhook URL。
* `channels.zalo.accounts.<id>.webhookSecret`: アカウントごとの webhook シークレット。
* `channels.zalo.accounts.<id>.webhookPath`: アカウントごとの webhook パス。
* `channels.zalo.accounts.<id>.proxy`: アカウントごとのプロキシ URL。