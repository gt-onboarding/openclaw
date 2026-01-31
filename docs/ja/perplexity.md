---
title: Perplexity
summary: "web_search で Perplexity Sonar を利用するためのセットアップ"
read_when:
  - Perplexity Sonar を Web 検索に利用したい場合
  - PERPLEXITY_API_KEY または OpenRouter のセットアップが必要な場合
---

<div id="perplexity-sonar">
  # Perplexity Sonar
</div>

OpenClaw は `web_search` ツールとして Perplexity Sonar を利用できます。Perplexity が提供する api を直接利用するか、OpenRouter 経由で接続できます。

<div id="api-options">
  ## API オプション
</div>

<div id="perplexity-direct">
  ### Perplexity（直接利用）
</div>

* ベース URL: https://api.perplexity.ai
* 環境変数: `PERPLEXITY_API_KEY`

<div id="openrouter-alternative">
  ### OpenRouter（代替）
</div>

* ベース URL：https://openrouter.ai/api/v1
* 環境変数：`OPENROUTER_API_KEY`
* プリペイドおよび暗号資産クレジットをサポート。

<div id="config-example">
  ## 設定例
</div>

```json5
{
  tools: {
    web: {
      search: {
        provider: "perplexity",
        perplexity: {
          apiKey: "pplx-...",
          baseUrl: "https://api.perplexity.ai",
          model: "perplexity/sonar-pro"
        }
      }
    }
  }
}
```

<div id="switching-from-brave">
  ## Brave からの切り替え
</div>

```json5
{
  tools: {
    web: {
      search: {
        provider: "perplexity",
        perplexity: {
          apiKey: "pplx-...",
          baseUrl: "https://api.perplexity.ai"
        }
      }
    }
  }
}
```

`PERPLEXITY_API_KEY` と `OPENROUTER_API_KEY` の両方が設定されている場合は、
どちらを使うかを明示するために `tools.web.search.perplexity.baseUrl`（または `tools.web.search.perplexity.apiKey`）を設定してください。

ベース URL が設定されていない場合、OpenClaw は API キーの発行元に基づいてデフォルトを選択します：

* `PERPLEXITY_API_KEY` または `pplx-...` → Perplexity への直接接続（`https://api.perplexity.ai`）
* `OPENROUTER_API_KEY` または `sk-or-...` → OpenRouter（`https://openrouter.ai/api/v1`）
* 不明なキー形式 → OpenRouter（安全なフォールバック先）

<div id="models">
  ## モデル
</div>

* `perplexity/sonar` — Web 検索対応の高速な Q&amp;A
* `perplexity/sonar-pro` (デフォルト) — マルチステップ推論 + Web 検索
* `perplexity/sonar-reasoning-pro` — 詳細なリサーチ向け

`web_search` の完全な構成については、[Web tools](/ja/tools/web) を参照してください。