---
title: Minimax
summary: "OpenClaw で MiniMax M2.1 を利用する"
read_when:
  - OpenClaw で MiniMax モデルを利用したい
  - MiniMax のセットアップ方法が必要
---

<div id="minimax">
  # MiniMax
</div>

MiniMax は **M2/M2.1** モデルファミリーを開発している AI 企業です。現在の
コード関連機能に特化したリリースは **MiniMax M2.1**（2025 年 12 月 23 日リリース）で、
実世界の複雑なタスク向けに設計されています。

出典: [MiniMax M2.1 release note](https://www.minimax.io/news/minimax-m21)

<div id="model-overview-m21">
  ## モデル概要（M2.1）
</div>

MiniMax は M2.1 の改善点として次の項目を挙げています:

* **多言語でのコーディング**（Rust、Java、Go、C++、Kotlin、Objective-C、TS/JS）の性能が向上。
* **Web/アプリ開発**と UI/デザイン面の出力品質（ネイティブモバイルを含む）が向上。
* インタリーブ思考と制約付き実行の統合を土台に、オフィス系ワークフロー向けの **複合指示** の処理が改善。
* **より簡潔な応答** により、トークン使用量が抑えられ、反復ループが高速化。
* **ツール/エージェントフレームワーク** との互換性とコンテキスト管理が強化（Claude Code、Droid/Factory AI、Cline、Kilo Code、Roo Code、BlackBox）。
* **対話および技術文書** の生成品質が向上。

<div id="minimax-m21-vs-minimax-m21-lightning">
  ## MiniMax M2.1 と MiniMax M2.1 Lightning の比較
</div>

* **速度:** Lightning は、MiniMax の料金表では「高速」版として扱われています。
* **コスト:** 料金表上では入力コストは同じですが、Lightning の出力コストはより高く設定されています。
* **Coding プランでのルーティング:** Lightning のバックエンドは、MiniMax の Coding プランからは直接利用できません。MiniMax はほとんどのリクエストを自動的に Lightning にルーティングしますが、トラフィックが急増した場合には通常の M2.1 バックエンドにフォールバックします。

<div id="choose-a-setup">
  ## セットアップ方法を選ぶ
</div>

<div id="minimax-m21-recommended">
  ### MiniMax M2.1 — 推奨
</div>

**最適な用途:** Anthropic 互換 API を使用するホスト型 MiniMax。

CLI から設定します:

* `openclaw configure` を実行
* **Model/auth** を選択
* **MiniMax M2.1** を選択

```json5
{
  env: { MINIMAX_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "minimax/MiniMax-M2.1" } } },
  models: {
    mode: "merge",
    providers: {
      minimax: {
        baseUrl: "https://api.minimax.io/anthropic",
        apiKey: "${MINIMAX_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 15, output: 60, cacheRead: 2, cacheWrite: 10 },
            contextWindow: 200000,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

<div id="minimax-m21-as-fallback-opus-primary">
  ### MiniMax M2.1 をフォールバック先に設定（Opus をプライマリ）
</div>

**最適な用途:** Opus 4.5 をプライマリとして使用しつつ、MiniMax M2.1 へフェイルオーバーする構成。

```json5
{
  env: { MINIMAX_API_KEY: "sk-..." },
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-5": { alias: "opus" },
        "minimax/MiniMax-M2.1": { alias: "minimax" }
      },
      model: {
        primary: "anthropic/claude-opus-4-5",
        fallbacks: ["minimax/MiniMax-M2.1"]
      }
    }
  }
}
```

<div id="optional-local-via-lm-studio-manual">
  ### オプション: LM Studio を使ったローカル実行（手動）
</div>

**最適:** LM Studio を使ったローカル推論。
LM Studio のローカルサーバーを使用し、十分に高性能なハードウェア（例: デスクトップ/サーバー）上で MiniMax M2.1 を利用したところ、良好な結果が得られています。

`openclaw.json` を手動で設定します:

```json5
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.1-gs32" },
      models: { "lmstudio/minimax-m2.1-gs32": { alias: "Minimax" } }
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

<div id="configure-via-openclaw-configure">
  ## `openclaw configure` で設定する
</div>

インタラクティブな設定ウィザードを使って、JSON ファイルを直接編集せずに MiniMax を設定します:

1. `openclaw configure` を実行します。
2. **Model/auth** を選択します。
3. **MiniMax M2.1** を選択します。
4. プロンプトが表示されたら、使用するデフォルトモデルを選択します。

<div id="configuration-options">
  ## 設定オプション
</div>

* `models.providers.minimax.baseUrl`: `https://api.minimax.io/anthropic`（Anthropic 互換）を推奨します。`https://api.minimax.io/v1` は、OpenAI 互換ペイロード用のオプションです。
* `models.providers.minimax.api`: `anthropic-messages` を推奨します。`openai-completions` は、OpenAI 互換ペイロード用のオプションです。
* `models.providers.minimax.apiKey`: MiniMax の API キー（`MINIMAX_API_KEY`）。
* `models.providers.minimax.models`: `id`、`name`、`reasoning`、`contextWindow`、`maxTokens`、`cost` を定義します。
* `agents.defaults.models`: 許可リストに含めたいモデルのエイリアスを定義します。
* `models.mode`: MiniMax をビルトインモデルと併用したい場合は、`merge` のままにしておきます。

<div id="notes">
  ## 注意事項
</div>

* モデルの参照名は `minimax/<model>` です。
* Coding Plan の使用量 API: `https://api.minimaxi.com/v1/api/openplatform/coding_plan/remains`（Coding Plan キーが必要）。
* 正確なコストを追跡する必要がある場合は、`models.json` 内の料金の値を更新してください。
* MiniMax Coding Plan のリファラルリンク（10% オフ）: https://platform.minimax.io/subscribe/coding-plan?code=DbXJTRClnb&amp;source=link
* プロバイダーのルールについては [/concepts/model-providers](/ja/concepts/model-providers) を参照してください。
* `openclaw models list` と `openclaw models set minimax/MiniMax-M2.1` を使用してモデルを切り替えてください。

<div id="troubleshooting">
  ## トラブルシューティング
</div>

<div id="unknown-model-minimaxminimax-m21">
  ### “Unknown model: minimax/MiniMax-M2.1”
</div>

これは通常、**MiniMax プロバイダーが未設定**（プロバイダーエントリがなく、MiniMax の認証プロファイル／環境変数キーも見つからない）であることを意味します。この問題の検出ロジックに対する修正は **2026.1.12**（執筆時点では未リリース）で行われています。次のいずれかで対処してください:

* **2026.1.12** にアップグレード（またはソースの `main` から実行）し、その後 Gateway を再起動する。
* `openclaw configure` を実行して **MiniMax M2.1** を選択する、または
* `models.providers.minimax` ブロックを手動で追加する、または
* `MINIMAX_API_KEY`（または MiniMax の認証プロファイル）を設定して、プロバイダーが注入されるようにする。

モデル ID は **大文字小文字が区別される** ことを確認してください:

* `minimax/MiniMax-M2.1`
* `minimax/MiniMax-M2.1-lightning`

その後、次のコマンドで再確認してください:

```bash
openclaw models list
```
