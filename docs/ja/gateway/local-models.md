---
title: ローカルモデル
summary: "ローカル LLM 上で OpenClaw を実行する（LM Studio、vLLM、LiteLLM、カスタム OpenAI エンドポイント）"
read_when:
  - 自前の GPU マシンからモデルを提供したいとき
  - LM Studio または OpenAI 互換プロキシと連携したいとき
  - もっとも安全なローカルモデル運用に関するガイドが必要なとき
---

<div id="local-models">
  # ローカルモデル
</div>

ローカル実行も可能だが、OpenClaw は大きなコンテキストとプロンプトインジェクションに対する強力な防御を前提としている。小容量の GPU カードではコンテキストが切り捨てられ、安全対策が弱まりやすい。目標は高く設定すること：**最大構成の Mac Studio 2 台以上、または同等の GPU リグ（約 3 万ドル超）**。単一の **24 GB** GPU が現実的なのは、軽めのプロンプトで、かつ高い待ち時間を許容できる場合のみ。**実行可能な範囲で最大／フルサイズのモデルバリアント**を使用すること。強く量子化されたチェックポイントや「小さい」チェックポイントはプロンプトインジェクションのリスクを高める（[Security](/ja/gateway/security) を参照）。

<div id="recommended-lm-studio-minimax-m21-responses-api-full-size">
  ## 推奨: LM Studio + MiniMax M2.1（Responses API、フルサイズ）
</div>

現時点で最良のローカルスタック構成です。LM Studio で MiniMax M2.1 をロードし、ローカルサーバー（デフォルト `http://127.0.0.1:1234`）を有効にして、推論と最終的なテキスト生成を分離するために Responses API を使用します。

```json5
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.1-gs32" },
      models: {
        "anthropic/claude-opus-4-5": { alias: "Opus" },
        "lmstudio/minimax-m2.1-gs32": { alias: "Minimax" }
      }
    }
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.1-gs32",
            name: "MiniMax M2.1 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

**セットアップチェックリスト**

* LM Studio をインストール: https://lmstudio.ai
* LM Studio で、利用可能な **最も大きな MiniMax M2.1 ビルド** をダウンロードし（“small” や強く量子化されたバリアントは避ける）、サーバーを起動して、`http://127.0.0.1:1234/v1/models` にそのモデルが表示されることを確認する。
* モデルはメモリに載せたままにしておく。コールドロードは起動レイテンシを増加させる。
* 使用している LM Studio ビルドが異なる場合は、`contextWindow` / `maxTokens` を調整する。
* WhatsApp 用には Responses API の利用に絞り、最終テキストのみが送信されるようにする。

ローカル実行時でもホスト型モデルは有効にしておき、フォールバックを維持するために `models.mode: "merge"` を使用すること。

<div id="hybrid-config-hosted-primary-local-fallback">
  ### ハイブリッド構成：ホストを優先し、ローカルをフォールバックとして利用
</div>

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-sonnet-4-5",
        fallbacks: ["lmstudio/minimax-m2.1-gs32", "anthropic/claude-opus-4-5"]
      },
      models: {
        "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
        "lmstudio/minimax-m2.1-gs32": { alias: "MiniMax Local" },
        "anthropic/claude-opus-4-5": { alias: "Opus" }
      }
    }
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.1-gs32",
            name: "MiniMax M2.1 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

<div id="local-first-with-hosted-safety-net">
  ### ホスト側をセーフティネットにしたローカル優先
</div>

primary と fallback の順序を入れ替えます。同じ providers ブロックを使い、`models.mode: "merge"` も維持することで、ローカルマシンがダウンしているときに Sonnet または Opus へフォールバックできます。

<div id="regional-hosting-data-routing">
  ### リージョンホスティング / データルーティング
</div>

* MiniMax/Kimi/GLM のホスト版も、OpenRouter 上にリージョン固定エンドポイント（例: 米国ホスト）として存在します。そこでリージョンごとのバリアントを選択すれば、`models.mode: "merge"` を使った Anthropic / OpenAI フォールバックを維持しつつ、トラフィックを希望する法域内にとどめることができます。
* ローカルのみの運用が、依然として最も強力なプライバシー保護の手段です。ホスト型のリージョンルーティングは、プロバイダーの機能が必要でありつつも、データフローを制御したい場合の中間的な選択肢となります。

<div id="other-openai-compatible-local-proxies">
  ## その他の OpenAI 互換ローカルプロキシ
</div>

vLLM、LiteLLM、OAI-proxy、またはカスタムゲートウェイは、OpenAI 互換の `/v1` エンドポイントを公開していれば利用できます。上記の `provider` ブロックを、自分のエンドポイントとモデル ID に置き換えてください。

```json5
{
  models: {
    mode: "merge",
    providers: {
      local: {
        baseUrl: "http://127.0.0.1:8000/v1",
        apiKey: "sk-local",
        api: "openai-responses",
        models: [
          {
            id: "my-local-model",
            name: "Local Model",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 120000,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

ホスト型モデルをフォールバックとして引き続き利用できるように、`models.mode: "merge"` のままにしておいてください。

<div id="troubleshooting">
  ## トラブルシューティング
</div>

* Gateway からプロキシに接続できるか？`curl http://127.0.0.1:1234/v1/models` を実行して確認します。
* LM Studio のモデルがアンロードされていないか？アンロードされている場合は再読み込みしてください。コールドスタートは「ハング」状態のよくある原因です。
* コンテキスト関連のエラーが発生しているか？`contextWindow` を小さくするか、サーバー側の上限を増やしてください。
* セキュリティ: ローカルモデルはプロバイダー側フィルターをスキップします。プロンプトインジェクションの影響範囲を抑えるため、エージェントの役割を絞り、コンパクションを有効にしておいてください。