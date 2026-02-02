---
title: Venice
summary: "OpenClaw で Venice AI のプライバシー重視モデルを利用する"
read_when:
  - OpenClaw でプライバシー重視の推論を行いたい場合
  - Venice AI のセットアップガイドが必要な場合
---

<div id="venice-ai-venice-highlight">
  # Venice AI (Venice ハイライト)
</div>

**Venice** は、プライバシーを最優先した推論を行うための、プロプライエタリモデルへの匿名アクセスも可能な代表的な Venice 構成です。

Venice AI は、検閲されていないモデルをサポートしつつ、主要なプロプライエタリモデルへ匿名プロキシ経由でアクセスできる、プライバシー重視の AI 推論を提供します。すべての推論はデフォルトでプライベートに処理され、あなたのデータで学習に利用されることも、ログが記録されることもありません。

<div id="why-venice-in-openclaw">
  ## OpenClaw で Venice を使う理由
</div>

- オープンソースモデル向けの**プライベート推論**（ログ記録なし）。
- 必要なときに使える**非検閲モデル**。
- 品質を重視したいときの、プロプライエタリモデル（Opus/GPT/Gemini）への**匿名化されたアクセス**。
- OpenAI 互換の `/v1` エンドポイント。

<div id="privacy-modes">
  ## プライバシーモード
</div>

Venice では 2 つのプライバシーレベルを提供しています — これを理解することが、どのモデルを選ぶかの鍵になります:

| モード | 説明 | モデル |
|------|-------------|--------|
| **Private** | 完全なプライベート。プロンプト／レスポンスは **一切保存もログ記録もされません**。エフェメラル（一時的にのみ保持されます）。 | Llama, Qwen, DeepSeek, Venice Uncensored など |
| **Anonymized** | Venice を経由してプロキシされ、その際にメタデータが削除されます。背後のプロバイダー（OpenAI, Anthropic）は匿名化されたリクエストのみを受け取ります。 | Claude, GPT, Gemini, Grok, Kimi, MiniMax |

<div id="features">
  ## 機能
</div>

- **プライバシー重視**: 「private」（完全非公開）と「anonymized」（匿名化プロキシ経由）のモードから選択可能
- **検閲なしモデル**: コンテンツ制限のないモデルにアクセス可能
- **主要モデルへのアクセス**: Claude、GPT-5.2、Gemini、Grok を Venice の匿名化プロキシ経由で利用可能
- **OpenAI 互換 API**: 簡単に統合できる標準の `/v1` エンドポイント
- **ストリーミング**: ✅ すべてのモデルで利用可能
- **Function calling**: ✅ 一部のモデルで利用可能（モデルの対応状況を参照）
- **ビジョン**: ✅ ビジョン対応モデルで利用可能
- **厳格なレート制限なし**: 極端な利用の場合はフェアユースに基づくスロットリングが適用される可能性あり

<div id="setup">
  ## セットアップ
</div>

<div id="1-get-api-key">
  ### 1. APIキーを取得する
</div>

1. [venice.ai](https://venice.ai) にアクセスしてサインアップする
2. **Settings → API Keys → Create new key** を開く
3. APIキー（形式: `vapi_xxxxxxxxxxxx`）をコピーする

<div id="2-configure-openclaw">
  ### 2. OpenClaw を設定する
</div>

**方法 A：環境変数**

```bash
export VENICE_API_KEY="vapi_xxxxxxxxxxxx"
```

**オプション B：インタラクティブセットアップ（推奨）**

```bash
openclaw onboard --auth-choice venice-api-key
```

これにより、次のことが行われます:

1. API キーの入力を求められます（既存の `VENICE_API_KEY` があればそれを使用します）
2. 利用可能なすべての Venice モデルが表示されます
3. デフォルトのモデルを選択できるようになります
4. プロバイダーが自動的に構成されます

**オプション C: 非対話型**

```bash
openclaw onboard --non-interactive \
  --auth-choice venice-api-key \
  --venice-api-key "vapi_xxxxxxxxxxxx"
```


<div id="3-verify-setup">
  ### 3. セットアップの確認
</div>

```bash
openclaw chat --model venice/llama-3.3-70b "Hello, are you working?"
```


<div id="model-selection">
  ## モデルの選択
</div>

セットアップ後、OpenClaw は利用可能な Venice モデルをすべて表示します。用途に応じて選択してください:

* **デフォルト（推奨）**: プライベートかつバランスの良い性能を備えた `venice/llama-3.3-70b`。
* **総合的な品質が最良**: 難易度の高いタスク向けの `venice/claude-opus-45`（Opus が依然として最強です）。
* **プライバシー重視**: 完全にプライベートな推論のために「private」モデルを選択。
* **機能性重視**: Venice のプロキシ経由で Claude、GPT、Gemini にアクセスするには「anonymized」モデルを選択。

デフォルトモデルはいつでも変更できます:

```bash
openclaw models set venice/claude-opus-45
openclaw models set venice/llama-3.3-70b
```

利用可能なモデルをすべて一覧表示：

```bash
openclaw models list | grep venice
```


<div id="configure-via-openclaw-configure">
  ## `openclaw configure` で設定する
</div>

1. `openclaw configure` を実行する
2. **Model/auth** を選択する
3. **Venice AI** を選択する

<div id="which-model-should-i-use">
  ## どのモデルを使うべきか？
</div>

| ユースケース | 推奨モデル | 理由 |
|----------|-------------------|-----|
| **一般的なチャット** | `llama-3.3-70b` | 万能で高性能、完全にプライベート |
| **総合的な品質重視** | `claude-opus-45` | 難しいタスクに対して依然として最強 |
| **プライバシー + Claude 品質** | `claude-opus-45` | 匿名化プロキシ経由で最高レベルの推論性能 |
| **コーディング** | `qwen3-coder-480b-a35b-instruct` | コード向けに最適化、262k トークンコンテキスト |
| **Vision タスク** | `qwen3-vl-235b-a22b` | 最高のプライベート Vision モデル |
| **検閲なし** | `venice-uncensored` | コンテンツ制限なし |
| **高速かつ低コスト** | `qwen3-4b` | 軽量だが十分な性能 |
| **複雑な推論** | `deepseek-v3.2` | 推論能力が高く、プライベート |

<div id="available-models-25-total">
  ## 利用可能なモデル（全25モデル）
</div>

<div id="private-models-15-fully-private-no-logging">
  ### プライベートモデル (15) — 完全プライベート、ログ記録なし
</div>

| Model ID | Name | Context (tokens) | Features |
|----------|------|------------------|----------|
| `llama-3.3-70b` | Llama 3.3 70B | 131k | 汎用 |
| `llama-3.2-3b` | Llama 3.2 3B | 131k | 高速・軽量 |
| `hermes-3-llama-3.1-405b` | Hermes 3 Llama 3.1 405B | 131k | 複雑なタスク向け |
| `qwen3-235b-a22b-thinking-2507` | Qwen3 235B Thinking | 131k | 推論 |
| `qwen3-235b-a22b-instruct-2507` | Qwen3 235B Instruct | 131k | 汎用 |
| `qwen3-coder-480b-a35b-instruct` | Qwen3 Coder 480B | 262k | コード |
| `qwen3-next-80b` | Qwen3 Next 80B | 262k | 汎用 |
| `qwen3-vl-235b-a22b` | Qwen3 VL 235B | 262k | ビジョン |
| `qwen3-4b` | Venice Small (Qwen3 4B) | 32k | 高速・推論向け |
| `deepseek-v3.2` | DeepSeek V3.2 | 163k | 推論 |
| `venice-uncensored` | Venice Uncensored | 32k | 検閲なし |
| `mistral-31-24b` | Venice Medium (Mistral) | 131k | ビジョン |
| `google-gemma-3-27b-it` | Gemma 3 27B Instruct | 202k | ビジョン |
| `openai-gpt-oss-120b` | OpenAI GPT OSS 120B | 131k | 汎用 |
| `zai-org-glm-4.7` | GLM 4.7 | 202k | 推論・多言語対応 |

<div id="anonymized-models-10-via-venice-proxy">
  ### 匿名化モデル (10) — Venice プロキシ経由
</div>

| Model ID | オリジナル | コンテキスト（トークン数） | 機能 |
|----------|------------|----------------------------|------|
| `claude-opus-45` | Claude Opus 4.5 | 202k | 推論、ビジョン |
| `claude-sonnet-45` | Claude Sonnet 4.5 | 202k | 推論、ビジョン |
| `openai-gpt-52` | GPT-5.2 | 262k | 推論 |
| `openai-gpt-52-codex` | GPT-5.2 Codex | 262k | 推論、ビジョン |
| `gemini-3-pro-preview` | Gemini 3 Pro | 202k | 推論、ビジョン |
| `gemini-3-flash-preview` | Gemini 3 Flash | 262k | 推論、ビジョン |
| `grok-41-fast` | Grok 4.1 Fast | 262k | 推論、ビジョン |
| `grok-code-fast-1` | Grok Code Fast 1 | 262k | 推論、コード |
| `kimi-k2-thinking` | Kimi K2 Thinking | 262k | 推論 |
| `minimax-m21` | MiniMax M2.1 | 202k | 推論 |

<div id="model-discovery">
  ## モデルの検出
</div>

OpenClaw は、`VENICE_API_KEY` が設定されている場合、Venice API からモデルを自動検出します。API に到達できない場合は、静的なカタログにフォールバックします。

`/models` エンドポイントはパブリックです（一覧取得に認証は不要）ですが、推論には有効な API キーが必要です。

<div id="streaming-tool-support">
  ## ストリーミングとツールのサポート
</div>

| 機能 | 対応状況 |
|---------|---------|
| **Streaming** | ✅ すべてのモデル |
| **Function calling** | ✅ ほとんどのモデル（api の `supportsFunctionCalling` を確認） |
| **Vision/Images** | ✅ 「Vision」機能付きのモデル |
| **JSON モード** | ✅ `response_format` でサポート |

<div id="pricing">
  ## 料金
</div>

Venice はクレジット制を採用しています。最新の料金は [venice.ai/pricing](https://venice.ai/pricing) を確認してください。

- **プライベートモデル**: 一般的にコストが低い
- **匿名化モデル**: 直接の api 料金とほぼ同じ + Venice の少額の手数料

<div id="comparison-venice-vs-direct-api">
  ## 比較: Venice と直接 API
</div>

| 比較項目 | Venice（匿名化） | 直接 API |
|--------|---------------------|------------|
| **プライバシー** | メタデータを削除し匿名化 | あなたのアカウントに紐付け |
| **レイテンシ** | +10〜50ms（プロキシ経由） | 直接 |
| **機能** | ほとんどの機能をサポート | すべての機能を利用可能 |
| **課金** | Venice クレジット | プロバイダー側で課金 |

<div id="usage-examples">
  ## 使用例
</div>

```bash
# Use default private model
openclaw chat --model venice/llama-3.3-70b

# Venice経由でClaudeを使用（匿名化）
openclaw chat --model venice/claude-opus-45

# Use uncensored model
openclaw chat --model venice/venice-uncensored

# Use vision model with image
openclaw chat --model venice/qwen3-vl-235b-a22b

# Use coding model
openclaw chat --model venice/qwen3-coder-480b-a35b-instruct
```


<div id="troubleshooting">
  ## トラブルシューティング
</div>

<div id="api-key-not-recognized">
  ### APIキーが認識されない
</div>

```bash
echo $VENICE_API_KEY
openclaw models list | grep venice
```

キーが `vapi_` で始まっていることを確認してください。


<div id="model-not-available">
  ### モデルが利用できません
</div>

Venice モデルカタログは動的に更新されます。現在利用可能なモデルを確認するには、`openclaw models list` を実行してください。一部のモデルは一時的にオフラインになっている場合があります。

<div id="connection-issues">
  ### 接続の問題
</div>

Venice API は `https://api.venice.ai/api/v1` です。ネットワークが HTTPS 接続を許可していることを確認してください。

<div id="config-file-example">
  ## 設定ファイルの例
</div>

```json5
{
  env: { VENICE_API_KEY: "vapi_..." },
  agents: { defaults: { model: { primary: "venice/llama-3.3-70b" } } },
  models: {
    mode: "merge",
    providers: {
      venice: {
        baseUrl: "https://api.venice.ai/api/v1",
        apiKey: "${VENICE_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "llama-3.3-70b",
            name: "Llama 3.3 70B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 131072,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```


<div id="links">
  ## リンク
</div>

- [Venice AI](https://venice.ai)
- [API ドキュメント](https://docs.venice.ai)
- [料金](https://venice.ai/pricing)
- [ステータス](https://status.venice.ai)