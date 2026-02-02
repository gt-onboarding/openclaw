---
title: 設定
summary: "`openclaw configure` の CLI リファレンス（対話的な設定プロンプト）"
read_when:
  - 認証情報、デバイス、またはエージェントのデフォルト設定を対話的に微調整したいとき
---

<div id="openclaw-configure">
  # `openclaw configure`
</div>

資格情報、デバイス、およびエージェントのデフォルトを設定するための対話型ウィザード。

注意: **Model** セクションには、`agents.defaults.models` 許可リスト（`/model` やモデルピッカーに表示されるもの）用の複数選択が追加されています。

Tip: サブコマンドなしの `openclaw config` は同じウィザードを開きます。非対話的な編集には `openclaw config get|set|unset` を使用してください。

関連:

* Gateway 設定リファレンス: [Configuration](/ja/gateway/configuration)
* 設定 CLI: [Config](/ja/cli/config)

メモ:

* Gateway をどこで実行するかを選択すると、必ず `gateway.mode` が更新されます。それだけでよい場合は、他のセクションはスキップして「Continue」を選択できます。
* チャンネル指向のサービス（Slack/Discord/Matrix/Microsoft Teams）は、セットアップ時にチャンネル／ルームの許可リストを尋ねます。名前または ID を入力できます。ウィザードは可能な場合、名前を ID に解決します。

<div id="examples">
  ## 使用例
</div>

```bash
openclaw configure
openclaw configure --section models --section channels
```
