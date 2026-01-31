---
title: Firecrawl
summary: "web_fetch 向けの Firecrawl フォールバック（ボット対策 + キャッシュ付き抽出）"
read_when:
  - Firecrawl バックエンドによる Web コンテンツ抽出を行いたいとき
  - Firecrawl の API キーが必要なとき
  - web_fetch でボット対策付きの抽出を行いたいとき
---

<div id="firecrawl">
  # Firecrawl
</div>

OpenClaw では `web_fetch` のフォールバック用抽出ツールとして **Firecrawl** を利用できます。これはボット回避やキャッシュ機能を備えたホスティング型のコンテンツ抽出サービスで、JavaScript を多用するサイトや、通常の HTTP fetch をブロックするページの取得に役立ちます。

<div id="get-an-api-key">
  ## API キーを取得する
</div>

1. Firecrawl アカウントを作成し、API キーを生成します。
2. 生成したキーを設定ファイルに保存するか、Gateway の環境で環境変数 `FIRECRAWL_API_KEY` として設定します。

<div id="configure-firecrawl">
  ## Firecrawl の設定
</div>

```json5
{
  tools: {
    web: {
      fetch: {
        firecrawl: {
          apiKey: "FIRECRAWL_API_KEY_HERE",
          baseUrl: "https://api.firecrawl.dev",
          onlyMainContent: true,
          maxAgeMs: 172800000,
          timeoutSeconds: 60
        }
      }
    }
  }
}
```

補足:

* `firecrawl.enabled` は、API キーが設定されている場合、デフォルトで true になります。
* `maxAgeMs` は、キャッシュされた結果をどれだけ古いものまで許容するか（ミリ秒単位）を制御します。デフォルトは 2 日です。

<div id="stealth-bot-circumvention">
  ## ステルス / ボット回避
</div>

Firecrawl は、ボット回避のための **proxy mode** パラメータ（`basic`、`stealth`、`auto` のいずれか）を公開しています。
OpenClaw は、Firecrawl へのリクエストでは常に `proxy: "auto"` と `storeInCache: true` を使用します。
`proxy` が省略された場合、Firecrawl のデフォルトは `auto` です。`auto` は `basic` での試行が失敗した場合に `stealth` プロキシで再試行するため、
`basic` のみでスクレイピングする場合よりも多くのクレジットを消費する可能性があります。

<div id="how-web_fetch-uses-firecrawl">
  ## `web_fetch` による Firecrawl の利用方法
</div>

`web_fetch` の抽出順序:

1. Readability（ローカル）
2. Firecrawl（設定されている場合）
3. 基本的な HTML クリーンアップ（最終フォールバック）

Web ツール一式のセットアップについては、[Web tools](/ja/tools/web) を参照してください。