---
title: Zai
summary: "OpenClaw で Z.AI（GLM モデル）を使用する"
read_when:
  - OpenClaw で Z.AI / GLM モデルを使いたい
  - ZAI_API_KEY を簡単にセットアップしたい
---

<div id="zai">
  # Z.AI
</div>

Z.AI は **GLM** モデル向けの API プラットフォームです。GLM 用の REST API を提供し、認証には API キーを使用します。Z.AI コンソールで API キーを作成してください。OpenClaw では、Z.AI の API キーを設定した `zai` プロバイダーを使用します。

<div id="cli-setup">
  ## CLI の設定
</div>

```bash
openclaw onboard --auth-choice zai-api-key
# または非対話モード
openclaw onboard --zai-api-key "$ZAI_API_KEY"
```

<div id="config-snippet">
  ## 設定スニペット
</div>

```json5
{
  env: { ZAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "zai/glm-4.7" } } }
}
```

<div id="notes">
  ## メモ
</div>

* GLM モデルは `zai/<model>`（例: `zai/glm-4.7`）として利用できます。
* モデルファミリーの概要については [/providers/glm](/ja/providers/glm) を参照してください。
* Z.AI は API キーによる Bearer 認証を使用します。