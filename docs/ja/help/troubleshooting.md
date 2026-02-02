---
title: トラブルシューティング
summary: "トラブルシューティングハブ: 症状 → 確認 → 対処"
read_when:
  - エラーが出て、解決までの手順を知りたいとき
  - インストーラーでは「成功」と表示されるのに CLI が動作しないとき
---

<div id="troubleshooting">
  # トラブルシューティング
</div>

<div id="first-60-seconds">
  ## 最初の60秒
</div>

次をこの順番で実行してください:

```bash
openclaw status
openclaw status --all
openclaw gateway probe
openclaw logs --follow
openclaw doctor
```

Gateway に到達できる場合は、より詳細なプローブを行います：

```bash
openclaw status --deep
```

<div id="common-it-broke-cases">
  ## よくあるトラブル
</div>

<div id="openclaw-command-not-found">
  ### `openclaw: command not found`
</div>

ほとんどの場合、Node.js/npm の PATH 設定の問題です。まずはここを確認してください:

* [インストール（Node.js/npm PATH の健全性チェック）](/ja/install#nodejs--npm-path-sanity)

<div id="installer-fails-or-you-need-full-logs">
  ### インストーラーが失敗する場合（または完全なログが必要な場合）
</div>

フルのトレースと npm の出力を確認するために、インストーラーを verbose モードで再実行します：

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --verbose
```

ベータ版をインストールしている場合：

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --beta --verbose
```

フラグを指定する代わりに `OPENCLAW_VERBOSE=1` を設定することもできます。

<div id="gateway-unauthorized-cant-connect-or-keeps-reconnecting">
  ### Gateway が「unauthorized」となって接続できない、または再接続を繰り返す
</div>

* [Gateway トラブルシューティング](/ja/gateway/troubleshooting)
* [Gateway 認証](/ja/gateway/authentication)

<div id="control-ui-fails-on-http-device-identity-required">
  ### Control UI が HTTP 経由で失敗する（デバイス識別が必要）
</div>

* [Gateway のトラブルシューティング](/ja/gateway/troubleshooting)
* [Control UI](/ja/web/control-ui#insecure-http)

<div id="docsopenclawai-shows-an-ssl-error-comcastxfinity">
  ### `docs.openclaw.ai` で SSL エラーが表示される (Comcast/Xfinity)
</div>

一部の Comcast/Xfinity 接続では、Xfinity Advanced Security によって `docs.openclaw.ai` へのアクセスがブロックされることがあります。
Advanced Security を無効化するか、`docs.openclaw.ai` を許可リストに追加してから再試行してください。

* Xfinity Advanced Security のヘルプ: https://www.xfinity.com/support/articles/using-xfinity-xfi-advanced-security
* 簡易な確認方法: モバイルホットスポットや VPN を試して、ISP レベルのフィルタリングかどうか確認してください

<div id="service-says-running-but-rpc-probe-fails">
  ### Service が稼働中と表示されるのに、RPC プローブが失敗する
</div>

* [Gateway のトラブルシューティング](/ja/gateway/troubleshooting)
* [バックグラウンドプロセス / サービス](/ja/gateway/background-process)

<div id="modelauth-failures-rate-limit-billing-all-models-failed">
  ### モデル／認証エラー（レート制限、課金、「すべてのモデルが失敗」）
</div>

* [Models](/ja/cli/models)
* [OAuth / 認証の概念](/ja/concepts/oauth)

<div id="model-says-model-not-allowed">
  ### `/model` が `model not allowed` と表示される
</div>

これは通常、`agents.defaults.models` が許可リストとして設定されていることを意味します。これが空でない場合は、
そのプロバイダー／モデルキーのみが選択可能になります。

* 許可リストを確認する: `openclaw config get agents.defaults.models`
* 使用したいモデルを追加する（または許可リストを空にして）、もう一度 `/model` を実行する
* `/models` を使って、許可されているプロバイダー／モデルを一覧する

<div id="when-filing-an-issue">
  ### Issue を作成するとき
</div>

安全なレポートを貼り付けてください:

```bash
openclaw status --all
```

可能であれば、`openclaw logs --follow` で得られる関連するログの末尾を含めてください。
