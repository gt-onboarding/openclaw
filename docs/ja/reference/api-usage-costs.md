---
title: API 使用コスト
summary: "費用が発生しうる要素、使用されているキー、および利用状況の確認方法を把握・監査する"
read_when:
  - どの機能が有料 API を呼び出す可能性があるか理解したいとき
  - キー、コスト、および利用状況の可視性を監査する必要があるとき
  - /status や /usage のコストレポートについて説明しているとき
---

<div id="api-usage-costs">
  # API 使用量とコスト
</div>

このドキュメントでは、**API キーを使用する可能性のある機能**と、そのコストがどこに計上されるかを一覧します。主に、プロバイダーの使用量や有料 API コールを発生させる可能性がある OpenClaw の機能に焦点を当てます。

<div id="where-costs-show-up-chat-cli">
  ## コストがどこに表示されるか（チャット + CLI）
</div>

**セッション単位のコストスナップショット**

* `/status` は現在のセッションのモデル、コンテキスト使用量、直近のレスポンスのトークン数を表示します。
* モデルが **API-key 認証**を使用している場合、`/status` は直近の返信の**推定コスト**も表示します。

**メッセージごとのコストフッター**

* `/usage full` は、すべての返信に利用状況フッターを追加し、**推定コスト**（API-key 使用時のみ）を含めて表示します。
* `/usage tokens` はトークン数のみを表示し、OAuth 認証フローの場合は金額（ドル建て）は表示しません。

**CLI の使用ウィンドウ（プロバイダーのクオータ）**

* `openclaw status --usage` と `openclaw channels list` は、プロバイダーの**使用ウィンドウ**を表示します
  （メッセージ単位のコストではなく、クオータのスナップショットです）。

詳細と例については [Token use &amp; costs](/ja/token-use) を参照してください。

<div id="how-keys-are-discovered">
  ## キーの検出方法
</div>

OpenClaw は以下の場所から認証情報を取得できます：

* **認証プロファイル**（エージェントごとに `auth-profiles.json` に保存）。
* **環境変数**（例えば `OPENAI_API_KEY`、`BRAVE_API_KEY`、`FIRECRAWL_API_KEY`）。
* **設定**（`models.providers.*.apiKey`、`tools.web.search.*`、`tools.web.fetch.firecrawl.*`、
  `memorySearch.*`、`talk.apiKey`）。
* **スキル**（`skills.entries.<name>.apiKey`。スキルの実行プロセスの環境変数にキーをエクスポートする場合があります）。

<div id="features-that-can-spend-keys">
  ## キーを消費する機能
</div>

<div id="1-core-model-responses-chat-tools">
  ### 1) コアモデルのレスポンス（チャット + ツール）
</div>

すべての返信およびツール呼び出しは、**現在のモデルプロバイダー**（OpenAI、Anthropic など）を使用します。これは利用量およびコストの主な要因になります。

料金設定の構成については [Models](/ja/providers/models)、トークン使用量とコストの表示については [Token use &amp; costs](/ja/token-use) を参照してください。

<div id="2-media-understanding-audioimagevideo">
  ### 2) メディア理解（音声／画像／動画）
</div>

受信したメディアは、返信が実行される前に要約や文字起こしを行うことができます。これはモデル／プロバイダーの api を利用します。

* 音声: OpenAI / Groq / Deepgram（キーが存在する場合は、現在 **自動的に有効化** されます）。
* 画像: OpenAI / Anthropic / Google。
* 動画: Google。

[メディア理解](/ja/nodes/media-understanding) を参照してください。

<div id="3-memory-embeddings-semantic-search">
  ### 3) メモリ埋め込み + セマンティック検索
</div>

セマンティックメモリ検索は、リモートプロバイダーを使用するよう設定されている場合、**埋め込み用 API** を使用します:

* `memorySearch.provider = "openai"` → OpenAI 埋め込み
* `memorySearch.provider = "gemini"` → Gemini 埋め込み
* ローカル埋め込みが失敗した場合は、任意で OpenAI へのフォールバックを設定可能

`memorySearch.provider = "local"` にすることでローカルのみで動作させることができます（API は使用しません）。

[Memory](/ja/concepts/memory) を参照してください。

<div id="4-web-search-tool-brave-perplexity-via-openrouter">
  ### 4) Web search tool (Brave / Perplexity via OpenRouter)
</div>

`web_search` は API キーを使用し、利用に応じて料金が発生する場合があります:

* **Brave Search API**: `BRAVE_API_KEY` または `tools.web.search.apiKey`
* **Perplexity**（OpenRouter 経由）: `PERPLEXITY_API_KEY` または `OPENROUTER_API_KEY`

**Brave の無料枠（かなり余裕あり）:**

* **月あたり 2,000 リクエスト**
* **1 リクエスト/秒**
* 本人確認のために **クレジットカードが必要**（アップグレードしない限り請求は発生しません）

[Web tools](/ja/tools/web) を参照してください。

<div id="5-web-fetch-tool-firecrawl">
  ### 5) Web fetch ツール (Firecrawl)
</div>

`web_fetch` は API キーが設定されている場合に **Firecrawl** を呼び出します:

* `FIRECRAWL_API_KEY` または `tools.web.fetch.firecrawl.apiKey`

Firecrawl が構成されていない場合、このツールは直接の fetch と readability にフォールバックします（有料 API は使用しません）。

[Web tools](/ja/tools/web) を参照してください。

<div id="6-provider-usage-snapshots-statushealth">
  ### 6) プロバイダー利用状況スナップショット（ステータス／ヘルス）
</div>

一部のステータス系コマンドは、クォータ期間や認証状態（ヘルス）を表示するために **プロバイダー利用状況エンドポイント** を呼び出します。
これらは通常、呼び出し頻度は低いものの、プロバイダーの API にはアクセスします。

* `openclaw status --usage`
* `openclaw models status --json`

[Models CLI](/ja/cli/models) を参照してください。

<div id="7-compaction-safeguard-summarization">
  ### 7) 圧縮セーフガードによる要約
</div>

圧縮セーフガードは **現在のモデル** を使ってセッション履歴を要約でき、
その際にプロバイダーの API を呼び出します。

[セッション管理と圧縮](/ja/reference/session-management-compaction)を参照してください。

<div id="8-model-scan-probe">
  ### 8) モデルのスキャン / プローブ
</div>

`openclaw models scan` は OpenRouter のモデルに対してプローブを実行でき、プローブ機能が有効な場合は `OPENROUTER_API_KEY` を使用します。

[Models CLI](/ja/cli/models) を参照してください。

<div id="9-talk-speech">
  ### 9) Talk（音声）
</div>

Talk モードは、設定されていれば **ElevenLabs** を呼び出せます：

* `ELEVENLABS_API_KEY` または `talk.apiKey`

[Talk モード](/ja/nodes/talk) を参照してください。

<div id="10-skills-third-party-apis">
  ### 10) スキル（サードパーティ API）
</div>

スキルは `apiKey` を `skills.entries.<name>.apiKey` に保存できます。スキルがそのキーを外部
API に使用する場合、そのスキルのプロバイダーに応じて料金が発生する可能性があります。

[スキル](/ja/tools/skills) を参照してください。