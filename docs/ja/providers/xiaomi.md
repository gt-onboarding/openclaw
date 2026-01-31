---
title: Xiaomi
summary: "OpenClaw で Xiaomi MiMo（mimo-v2-flash）を使用する"
read_when:
  - OpenClaw で Xiaomi MiMo モデルを使用したい
  - XIAOMI_API_KEY を設定する必要がある
---

<div id="xiaomi-mimo">
  # Xiaomi MiMo
</div>

Xiaomi MiMo は **MiMo** モデル向けの API プラットフォームです。OpenAI および Anthropic のフォーマットと互換性のある REST API を提供し、認証には API キーを使用します。API キーは
[Xiaomi MiMo コンソール](https://platform.xiaomimimo.com/#/console/api-keys) で作成します。OpenClaw は
`xiaomi` プロバイダーを Xiaomi MiMo の API キーと併せて使用します。

<div id="model-overview">
  ## モデル概要
</div>

* **mimo-v2-flash**: 262144トークンのコンテキストウィンドウをサポートし、Anthropic Messages API と互換性があります。
* ベース URL: `https://api.xiaomimimo.com/anthropic`
* 認証方式: `Bearer $XIAOMI_API_KEY`

<div id="cli-setup">
  ## CLI セットアップ
</div>

```bash
openclaw onboard --auth-choice xiaomi-api-key
# または非対話モード
openclaw onboard --auth-choice xiaomi-api-key --xiaomi-api-key "$XIAOMI_API_KEY"
```

<div id="config-snippet">
  ## 設定スニペット
</div>

```json5
{
  env: { XIAOMI_API_KEY: "your-key" },
  agents: { defaults: { model: { primary: "xiaomi/mimo-v2-flash" } } },
  models: {
    mode: "merge",
    providers: {
      xiaomi: {
        baseUrl: "https://api.xiaomimimo.com/anthropic",
        api: "anthropic-messages",
        apiKey: "XIAOMI_API_KEY",
        models: [
          {
            id: "mimo-v2-flash",
            name: "Xiaomi MiMo V2 Flash",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 262144,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

<div id="notes">
  ## 注意事項
</div>

* モデルリファレンス: `xiaomi/mimo-v2-flash`。
* `XIAOMI_API_KEY` が設定されている（または認証プロファイルが存在している）場合、プロバイダーは自動的に設定されます。
* プロバイダーに関するルールについては [/concepts/model-providers](/ja/concepts/model-providers) を参照してください。