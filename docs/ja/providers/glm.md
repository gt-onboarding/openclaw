---
title: Glm
summary: "GLM モデルファミリーの概要と OpenClaw での利用方法"
read_when:
  - OpenClaw で GLM モデルを使いたいとき
  - モデルの命名規則とセットアップ方法を知りたいとき
---

<div id="glm-models">
  # GLM モデル
</div>

GLM は Z.AI プラットフォーム経由で利用可能な **モデルファミリー**（企業ではありません）です。OpenClaw では、GLM
モデルには `zai` プロバイダー経由で、`zai/glm-4.7` のようなモデル ID を指定してアクセスします。

<div id="cli-setup">
  ## CLI 設定
</div>

```bash
openclaw onboard --auth-choice zai-api-key
```

<div id="config-snippet">
  ## 設定例
</div>

```json5
{
  env: { ZAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "zai/glm-4.7" } } }
}
```

<div id="notes">
  ## 注意事項
</div>

* GLM のバージョンおよび提供状況は変更される可能性があります。最新の情報については Z.AI のドキュメントを確認してください。
* モデル ID の例として、`glm-4.7` や `glm-4.6` などがあります。
* プロバイダーの詳細については [/providers/zai](/ja/providers/zai) を参照してください。