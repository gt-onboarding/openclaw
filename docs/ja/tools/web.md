---
title: Web
summary: "Web 検索 + fetch 用ツール（Brave Search API、Perplexity 直接 / OpenRouter）"
read_when:
  - web_search または web_fetch を有効にしたい場合
  - Brave Search API キーのセットアップが必要な場合
  - Web 検索に Perplexity Sonar を利用したい場合
---

<div id="web-tools">
  # Web ツール
</div>

OpenClaw には、軽量な Web ツールが 2 つ同梱されています:

* `web_search` — Brave Search API（デフォルト）または Perplexity Sonar（直接または OpenRouter 経由）を使って Web を検索します。
* `web_fetch` — HTTP フェッチ + 可読な形への抽出（HTML → Markdown/text）。

これらはブラウザの自動操作を行うツールでは **ありません**。JavaScript を多用するサイトやログインが必要なサイトには、
[Browser ツール](/ja/tools/browser) を使用してください。

<div id="how-it-works">
  ## 動作概要
</div>

* `web_search` は設定済みのプロバイダーを呼び出し、結果を返します。
  * **Brave**（デフォルト）：構造化された結果（タイトル、URL、スニペット）を返します。
  * **Perplexity**：リアルタイムのウェブ検索結果に基づく出典付きの、AI 生成回答を返します。
* 結果はクエリごとに 15 分間キャッシュされます（設定可能）。
* `web_fetch` は通常の HTTP GET リクエストを実行し、人間が読めるコンテンツを抽出します
  （HTML → markdown/text）。JavaScript は**実行しません**。
* `web_fetch` はデフォルトで有効になっています（明示的に無効化しない限り）。

<div id="choosing-a-search-provider">
  ## 検索プロバイダーの選択
</div>

| Provider            | Pros                     | Cons                                | API Key                                      |
| ------------------- | ------------------------ | ----------------------------------- | -------------------------------------------- |
| **Brave** (default) | 高速・構造化された結果・無料枠あり        | 従来型の検索結果                            | `BRAVE_API_KEY`                              |
| **Perplexity**      | AI による統合回答、引用付き、リアルタイム対応 | Perplexity または OpenRouter へのアクセスが必要 | `OPENROUTER_API_KEY` or `PERPLEXITY_API_KEY` |

プロバイダーごとの詳細については、[Brave Search のセットアップ](/ja/brave-search) および [Perplexity Sonar](/ja/perplexity) を参照してください。

設定ファイルでプロバイダーを指定します:

```json5
{
  tools: {
    web: {
      search: {
        provider: "brave"  // または "perplexity"
      }
    }
  }
}
```

例: Perplexity Sonar（直接api）に切り替える

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

<div id="getting-a-brave-api-key">
  ## Brave API キーの取得
</div>

1. https://brave.com/search/api/ で Brave Search API アカウントを作成します。
2. ダッシュボードで **Data for Search** プラン（「Data for AI」ではない）を選択し、API キーを生成します。
3. `openclaw configure --section web` を実行して設定ファイルにキーを保存する（推奨）、または環境変数 `BRAVE_API_KEY` を設定します。

Brave には無料枠と有料プランがあります。最新の利用制限や料金については Brave API ポータルを確認してください。

<div id="where-to-set-the-key-recommended">
  ### キーの設定場所（推奨）
</div>

**推奨:** `openclaw configure --section web` を実行します。これにより、キーは
`~/.openclaw/openclaw.json` の `tools.web.search.apiKey` に保存されます。

**環境変数を使う方法:** Gateway プロセスの環境変数として `BRAVE_API_KEY` を設定します。
Gateway インストールの場合は、`~/.openclaw/.env`（またはサービスの
環境設定）に記述します。詳しくは [Env vars](/ja/help/faq#how-does-openclaw-load-environment-variables) を参照してください。

<div id="using-perplexity-direct-or-via-openrouter">
  ## Perplexity を使う（直接利用または OpenRouter 経由）
</div>

Perplexity Sonar モデルにはウェブ検索機能が組み込まれており、出典付きの
AI 生成回答を返します。これらのモデルは OpenRouter 経由で利用でき（クレジットカード不要、
暗号資産/プリペイドに対応）、

<div id="getting-an-openrouter-api-key">
  ### OpenRouter の API キーを取得する
</div>

1. https://openrouter.ai/ でアカウントを作成します
2. クレジット残高を追加します（暗号資産、プリペイド、クレジットカードに対応）
3. アカウント設定で API キーを発行します

<div id="setting-up-perplexity-search">
  ### Perplexity 検索のセットアップ
</div>

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        provider: "perplexity",
        perplexity: {
          // APIキー（OPENROUTER_API_KEYまたはPERPLEXITY_API_KEYが設定されている場合は省略可）
          apiKey: "sk-or-v1-...",
          // Base URL (key-aware default if omitted)
          baseUrl: "https://openrouter.ai/api/v1",
          // Model (defaults to perplexity/sonar-pro)
          model: "perplexity/sonar-pro"
        }
      }
    }
  }
}
```

**環境変数で指定する場合:** Gateway の実行環境に `OPENROUTER_API_KEY` または `PERPLEXITY_API_KEY` を設定します。Gateway をインストールしている場合は、`~/.openclaw/.env` に記述します。

ベース URL が設定されていない場合、OpenClaw は API キーの種別に基づいてデフォルト値を選択します:

* `PERPLEXITY_API_KEY` または `pplx-...` → `https://api.perplexity.ai`
* `OPENROUTER_API_KEY` または `sk-or-...` → `https://openrouter.ai/api/v1`
* 不明なキー形式 → OpenRouter（安全なフォールバック先）

<div id="available-perplexity-models">
  ### 利用可能な Perplexity モデル
</div>

| モデル | 説明 | 最適な用途 |
|-------|-------------|----------|
| `perplexity/sonar` | Web検索対応の高速 Q&amp;A | 素早い調べ物 |
| `perplexity/sonar-pro` (default) | Web検索対応のマルチステップ推論 | 複雑な質問 |
| `perplexity/sonar-reasoning-pro` | 思考の連鎖に基づく分析 | 深い調査 |

<div id="web_search">
  ## web_search
</div>

設定済みのプロバイダーを使ってウェブを検索します。

<div id="requirements">
  ### 要件
</div>

* `tools.web.search.enabled` が `false` になっていないこと（デフォルト: 有効）
* 選択したプロバイダー用の API キーが必要:
  * **Brave**: `BRAVE_API_KEY` または `tools.web.search.apiKey`
  * **Perplexity**: `OPENROUTER_API_KEY`、`PERPLEXITY_API_KEY`、または `tools.web.search.perplexity.apiKey`

<div id="config">
  ### 設定
</div>

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        apiKey: "BRAVE_API_KEY_HERE", // BRAVE_API_KEY が設定されている場合はオプション
        maxResults: 5,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15
      }
    }
  }
}
```

<div id="tool-parameters">
  ### ツールパラメーター
</div>

* `query`（必須）
* `count`（1～10。デフォルト値は設定に従う）
* `country`（任意）：地域別結果用の2文字の国コード（例: &quot;DE&quot;, &quot;US&quot;, &quot;ALL&quot;）。省略した場合、Brave がデフォルトの地域を自動選択する。
* `search_lang`（任意）：検索結果用の ISO 言語コード（例: &quot;de&quot;, &quot;en&quot;, &quot;fr&quot;）
* `ui_lang`（任意）：UI 要素用の ISO 言語コード
* `freshness`（任意、Brave のみ）：発見時刻によるフィルタリング（`pd`、`pw`、`pm`、`py`、または `YYYY-MM-DDtoYYYY-MM-DD`）

**例:**

```javascript
// German-specific search
await web_search({
  query: "TV online schauen",
  count: 10,
  country: "DE",
  search_lang: "de"
});

// フランス語UIでのフランス語検索
await web_search({
  query: "actualités",
  country: "FR",
  search_lang: "fr",
  ui_lang: "fr"
});

// Recent results (past week)
await web_search({
  query: "TMBG interview",
  freshness: "pw"
});
```

<div id="web_fetch">
  ## web_fetch
</div>

URL から情報を取得し、読みやすいコンテンツを抽出します。

<div id="requirements">
  ### 要件
</div>

* `tools.web.fetch.enabled` は `false` であってはいけません（デフォルト: 有効）
* （任意）Firecrawl フォールバックを使用する場合は、`tools.web.fetch.firecrawl.apiKey` または `FIRECRAWL_API_KEY` を設定します。

<div id="config">
  ### 設定
</div>

```json5
{
  tools: {
    web: {
      fetch: {
        enabled: true,
        maxChars: 50000,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
        maxRedirects: 3,
        userAgent: "Mozilla/5.0 (Macintosh; Intel Mac OS X 14_7_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36",
        readability: true,
        firecrawl: {
          enabled: true,
          apiKey: "FIRECRAWL_API_KEY_HERE", // FIRECRAWL_API_KEY が設定されている場合は省略可能
          baseUrl: "https://api.firecrawl.dev",
          onlyMainContent: true,
          maxAgeMs: 86400000, // ms (1 day)
          timeoutSeconds: 60
        }
      }
    }
  }
}
```

### ツールパラメータ

* `url` (必須、http/https のみ)
* `extractMode` (`markdown` | `text`)
* `maxChars` (長いページを切り詰める)

注意:

* `web_fetch` はまず Readability（メインコンテンツ抽出）を使用し、次に Firecrawl（設定されている場合）を使用します。両方とも失敗した場合、ツールはエラーを返します。
* Firecrawl へのリクエストは、デフォルトでボット回避モードを使用し、結果をキャッシュします。
* `web_fetch` はデフォルトで Chrome に類似した User-Agent と `Accept-Language` を送信します。必要に応じて `userAgent` で上書きしてください。
* `web_fetch` はプライベート/内部ホスト名をブロックし、リダイレクトを再確認します（`maxRedirects` で制限可能）。
* `web_fetch` はベストエフォートの抽出であり、一部のサイトではブラウザツールが必要になります。
* キーのセットアップとサービスの詳細については [Firecrawl](/ja/tools/firecrawl) を参照してください。
* レスポンスはキャッシュされます（デフォルト 15 分）ので、同一ページの再取得を削減できます。
* ツールプロファイルや許可リストを使用する場合は、`web_search`/`web_fetch` または `group:web` を追加してください。
* Brave キーがない場合、`web_search` は簡単なセットアップ手順のヒントとドキュメントへのリンクを返します。