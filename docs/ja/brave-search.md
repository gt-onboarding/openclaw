---
title: Brave Search
summary: "web_search 用の Brave Search API のセットアップ"
read_when:
  - web_search に Brave Search を使用したい場合
  - BRAVE_API_KEY またはプランの詳細が必要な場合
---

<div id="brave-search-api">
  # Brave Search API
</div>

OpenClaw は、`web_search` のデフォルトプロバイダーとして Brave Search を使用します。

<div id="get-an-api-key">
  ## API キーを取得する
</div>

1. https://brave.com/search/api/ で Brave Search API アカウントを作成します。
2. ダッシュボードで **Data for Search** プランを選択し、API キーを生成します。
3. 生成したキーを config に保存する（推奨）か、Gateway の実行環境で `BRAVE_API_KEY` を設定します。

<div id="config-example">
  ## 設定例
</div>

```json5
{
  tools: {
    web: {
      search: {
        provider: "brave",
        apiKey: "BRAVE_API_KEY_HERE",
        maxResults: 5,
        timeoutSeconds: 30
      }
    }
  }
}
```

<div id="notes">
  ## 補足
</div>

* Data for AI プランは `web_search` と**互換性がありません**。
* Brave には無料枠と有料プランがあります。最新の制限については Brave API ポータルを確認してください。

`web_search` の詳細な設定については [Web tools](/ja/tools/web) を参照してください。