---
title: モデルフェイルオーバー
summary: "OpenClaw における認証プロファイルのローテーションとモデル間フォールバックの仕組み"
read_when:
  - 認証プロファイルのローテーション、クールダウン、またはモデルのフォールバック動作を診断するとき
  - 認証プロファイルまたはモデルのフェイルオーバールールを更新するとき
---

<div id="model-failover">
  # モデルのフェイルオーバー
</div>

OpenClaw は障害を 2 段階で処理します:

1. 現在のプロバイダー内での **認証プロファイルのローテーション**。
2. `agents.defaults.model.fallbacks` 内で次のモデルへの **モデルフォールバック**。

このドキュメントでは、実行時のルールと、それを支えるデータについて説明します。

<div id="auth-storage-keys-oauth">
  ## 認証情報ストレージ（キー + OAuth）
</div>

OpenClaw は、API キーと OAuth トークンの両方に **認証プロファイル** を使用します。

* シークレットは `~/.openclaw/agents/<agentId>/agent/auth-profiles.json` に保存されます（レガシー: `~/.openclaw/agent/auth-profiles.json`）。
* 設定 `auth.profiles` / `auth.order` は **メタデータとルーティング専用** です（シークレットは含まない）。
* レガシーのインポート専用 OAuth ファイル: `~/.openclaw/credentials/oauth.json`（初回利用時に `auth-profiles.json` にインポートされる）。

詳細は [/concepts/oauth](/ja/concepts/oauth) を参照してください。

認証情報の種類:

* `type: "api_key"` → `{ provider, key }`
* `type: "oauth"` → `{ provider, access, refresh, expires, email? }`（一部のプロバイダーでは `projectId` / `enterpriseUrl` も含む）

<div id="profile-ids">
  ## プロファイル ID
</div>

OAuth ログインは個別のプロファイルを作成するため、複数のアカウントを同時に利用できます。

* デフォルト: メールアドレスが利用できない場合は `provider:default`。
* メールアドレス付き OAuth: `provider:<email>`（例: `google-antigravity:user@gmail.com`）。

プロファイルは `~/.openclaw/agents/<agentId>/agent/auth-profiles.json` の `profiles` 配下に保存されます。

<div id="rotation-order">
  ## ローテーション順序
</div>

1つのプロバイダーに複数のプロファイルがある場合、OpenClaw は次のような順序で選択します:

1. **明示的な設定**: `auth.order[provider]`（設定されている場合）。
2. **設定済みプロファイル**: プロバイダーでフィルタリングされた `auth.profiles`。
3. **保存済みプロファイル**: 対象プロバイダー向けの `auth-profiles.json` 内のエントリ。

明示的な順序が設定されていない場合、OpenClaw はラウンドロビン方式で順序付けします:

* **第1キー:** プロファイル種別（**OAuth が API キーより優先**）。
* **第2キー:** `usageStats.lastUsed`（各種別の中で最も古く使われていないものが優先）。
* **クールダウン中／無効化されたプロファイル** は末尾に回され、有効期限が早い順に並びます。

<div id="session-stickiness-cache-friendly">
  ### セッションのスティッキネス（キャッシュフレンドリー）
</div>

OpenClaw は、プロバイダーのキャッシュをウォームな状態に保つために、**選択された認証プロファイルをセッション単位でピン留め（固定）**します。
リクエストごとにローテーションすることは**ありません**。ピン留めされたプロファイルは、次のいずれかが起きるまで再利用されます。

* セッションがリセットされる（`/new` / `/reset`）
* コンパクションが完了する（コンパクションカウントが増加する）
* プロファイルがクールダウン中／無効化されている

`/model …@<profileId>` による手動選択は、そのセッションに対する**ユーザーのオーバーライド**を設定し、
新しいセッションが開始されるまで自動ローテーションされません。

（セッションルーターによって）自動的にピン留めされたプロファイルは**優先候補**として扱われます。
まずそのプロファイルが試されますが、レート制限やタイムアウトが発生した場合には、OpenClaw が別のプロファイルにローテーションすることがあります。
ユーザーがピン留めしたプロファイルはそのプロファイルに固定されたままです。そのプロファイルが失敗し、モデルのフォールバックが
設定されている場合、OpenClaw はプロファイルを切り替えるのではなく、次のモデルに切り替えます。

<div id="why-oauth-can-look-lost">
  ### OAuth が「行方不明」に見える理由
</div>

同じプロバイダーに対して OAuth プロファイルと API キー プロファイルの両方がある場合、ピン留めされていなければ、ラウンドロビン方式によりメッセージごとにそれらの間で切り替わることがあります。特定のプロファイルだけを使いたい場合は、次のようにします。

* `auth.order[provider] = ["provider:profileId"]` でピン留めする、または
* UI やチャット画面が対応している場合は、`/model …` を使ってプロファイル指定付きでセッション単位のオーバーライドを行う。

<div id="cooldowns">
  ## クールダウン
</div>

認証エラーやレート制限エラー（またはレート制限のように見えるタイムアウト）によって
プロファイルでエラーが発生した場合、OpenClaw はそのプロファイルをクールダウン状態としてマークし、
次のプロファイルへ切り替えます。
フォーマットエラー／不正なリクエストエラー（たとえば Cloud Code Assist ツール呼び出し ID
検証の失敗）はフェイルオーバー対象として扱われ、同じクールダウンポリシーが適用されます。

クールダウンは指数バックオフを使用します:

* 1 分
* 5 分
* 25 分
* 1 時間（上限）

この状態情報は `auth-profiles.json` の `usageStats` 配下に保存されます:

```json
{
  "usageStats": {
    "provider:profile": {
      "lastUsed": 1736160000000,
      "cooldownUntil": 1736160600000,
      "errorCount": 2
    }
  }
}
```

<div id="billing-disables">
  ## 課金の無効化
</div>

課金／クレジットのエラー（例: 「insufficient credits」「credit balance too low」）はフェイルオーバー対象の事象として扱われますが、通常は短期的なものではありません。短時間のクールダウンの代わりに、OpenClaw はそのプロファイルを**無効**（より長いバックオフ期間付き）としてマークし、次のプロファイル／プロバイダーへ切り替えます。

状態は `auth-profiles.json` に保存されます:

```json
{
  "usageStats": {
    "provider:profile": {
      "disabledUntil": 1736178000000,
      "disabledReason": "billing"
    }
  }
}
```

デフォルト:

* 課金バックオフは **5時間** から開始し、課金失敗が発生するたびに 2 倍になり、上限は **24時間** です。
* プロファイルで **24時間** 連続して失敗が発生していない場合、バックオフカウンタはリセットされます（設定可能）。

<div id="model-fallback">
  ## モデルフォールバック
</div>

特定のプロバイダーに対して定義されたすべてのプロファイルで失敗した場合、OpenClaw は
`agents.defaults.model.fallbacks` 内の次のモデルに進みます。これは、認証エラー、レート制限、
およびプロファイルのローテーションをすべて試し切った結果のタイムアウトに適用されます（それ以外のエラーではフォールバックは進みません）。

実行がモデルのオーバーライド（hooks または CLI）から始まった場合でも、設定されたフォールバックをすべて試した後の
最終的なフォールバック先は `agents.defaults.model.primary` になります。

<div id="related-config">
  ## 関連する設定
</div>

次を参照してください: [Gateway の設定](/ja/gateway/configuration)

* `auth.profiles` / `auth.order`
* `auth.cooldowns.billingBackoffHours` / `auth.cooldowns.billingBackoffHoursByProvider`
* `auth.cooldowns.billingMaxHours` / `auth.cooldowns.failureWindowHours`
* `agents.defaults.model.primary` / `agents.defaults.model.fallbacks`
* `agents.defaults.imageModel` のルーティング

モデル選択とフェイルオーバーの全体像については、[Models](/ja/concepts/models) を参照してください。