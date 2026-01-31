---
title: Ollama
summary: "Ollama（ローカル LLM ランタイム）と組み合わせて OpenClaw を実行する"
read_when:
  - Ollama 経由でローカルモデルを使って OpenClaw を実行したい
  - Ollama のセットアップと構成方法について知りたい
---

<div id="ollama">
  # Ollama
</div>

Ollama はローカルで動作する LLM ランタイムであり、マシン上でオープンソースモデルを簡単に実行できるようにします。OpenClaw は Ollama の OpenAI 互換 API と統合されており、`OLLAMA_API_KEY`（または認証プロファイル）でオプトインし、明示的な `models.providers.ollama` エントリを定義しない場合に、**ツール対応モデルを自動検出**できます。

<div id="quick-start">
  ## クイックスタート
</div>

1. Ollama をインストールします: https://ollama.ai

2. モデルを取得します:

```bash
ollama pull llama3.3
# または
ollama pull qwen2.5-coder:32b
# または
ollama pull deepseek-r1:32b
```

3. OpenClaw で Ollama を有効化します（任意の値で構いません。Ollama では実際のキーは不要です）:

```bash
# 環境変数を設定する
export OLLAMA_API_KEY="ollama-local"

# または設定ファイルで設定する
openclaw config set models.providers.ollama.apiKey "ollama-local"
```

4. Ollama モデルを使用する：

```json5
{
  agents: {
    defaults: {
      model: { primary: "ollama/llama3.3" }
    }
  }
}
```

<div id="model-discovery-implicit-provider">
  ## モデル検出（暗黙のプロバイダー）
</div>

`OLLAMA_API_KEY`（または認証プロファイル）を設定し、かつ `models.providers.ollama` を**定義しない**場合、OpenClaw はローカルの Ollama インスタンス（`http://127.0.0.1:11434`）からモデルを自動検出します:

* `/api/tags` と `/api/show` にクエリを送信する
* `tools` 機能を報告するモデルのみを保持する
* モデルが `thinking` を報告する場合、`reasoning` としてマークする
* 利用可能な場合、`model_info["<arch>.context_length"]` から `contextWindow` を読み取る
* `maxTokens` をコンテキストウィンドウの 10 倍に設定する
* すべてのコストを `0` に設定する

これにより、Ollama の機能にカタログを合わせつつ、モデルエントリーを手動で追加する必要がなくなります。

利用可能なモデルを確認するには:

```bash
ollama list
openclaw models list
```

新しいモデルを追加するには、Ollama を使ってプルするだけです。

```bash
ollama pull mistral
```

新しいモデルは自動的に検出され、利用できるようになります。

`models.providers.ollama` を明示的に設定した場合、自動検出はスキップされ、モデルを手動で定義する必要があります（下記参照）。

<div id="configuration">
  ## 設定
</div>

<div id="basic-setup-implicit-discovery">
  ### 基本設定（自動検出）
</div>

Ollama を有効にする最も簡単な方法は、環境変数を設定することです。

```bash
export OLLAMA_API_KEY="ollama-local"
```

<div id="explicit-setup-manual-models">
  ### 明示的セットアップ（手動モデル）
</div>

次のような場合には明示的な設定を使用します：

* Ollama が別のホスト／ポートで動作している場合
* 特定のコンテキストウィンドウやモデルリストを固定したい場合
* ツール対応を報告しないモデルも含めたい場合

```json5
{
  models: {
    providers: {
      ollama: {
        // OpenAI互換APIには /v1 を含むホストを使用
        baseUrl: "http://ollama-host:11434/v1",
        apiKey: "ollama-local",
        api: "openai-completions",
        models: [
          {
            id: "llama3.3",
            name: "Llama 3.3",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 8192,
            maxTokens: 8192 * 10
          }
        ]
      }
    }
  }
}
```

`OLLAMA_API_KEY` が設定されている場合、プロバイダーエントリで `apiKey` を省略できます。OpenClaw が可用性チェック用にその値を自動設定します。

<div id="custom-base-url-explicit-config">
  ### カスタムベース URL（明示的な設定）
</div>

Ollama が別のホストやポートで動作している場合（明示的な設定にすると自動検出が無効になるため、モデルを手動で定義する必要があります）:

```json5
{
  models: {
    providers: {
      ollama: {
        apiKey: "ollama-local",
        baseUrl: "http://ollama-host:11434/v1"
      }
    }
  }
}
```

<div id="model-selection">
  ### モデル選択
</div>

設定が完了すると、Ollama の全モデルが利用できるようになります。

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "ollama/llama3.3",
        fallback: ["ollama/qwen2.5-coder:32b"]
      }
    }
  }
}
```

<div id="advanced">
  ## 高度な機能
</div>

<div id="reasoning-models">
  ### 推論モデル
</div>

OpenClaw は、Ollama が `/api/show` で `thinking` を返した場合、そのモデルを推論対応モデルとしてマークします。

```bash
ollama pull deepseek-r1:32b
```

<div id="model-costs">
  ### モデルのコスト
</div>

Ollama は無料でローカルで動作するため、すべてのモデルのコストは $0 に設定されています。

<div id="context-windows">
  ### コンテキストウィンドウ
</div>

自動検出されたモデルに対しては、利用可能な場合は Ollama が報告するコンテキストウィンドウを OpenClaw が使用し、利用できない場合はデフォルトで `8192` を使用します。明示的なプロバイダー設定で `contextWindow` と `maxTokens` をオーバーライドできます。

<div id="troubleshooting">
  ## トラブルシューティング
</div>

<div id="ollama-not-detected">
  ### Ollama が検出されません
</div>

Ollama が起動しており、`OLLAMA_API_KEY`（または認証プロファイル）が設定されていて、かつ明示的な `models.providers.ollama` エントリを **定義していない** ことを確認してください：

```bash
ollama serve
```

また、api にアクセスできることを確認してください：

```bash
curl http://localhost:11434/api/tags
```

<div id="no-models-available">
  ### 利用可能なモデルがありません
</div>

OpenClaw は、ツールサポートを報告するモデルのみを自動検出します。モデルが一覧に表示されない場合は、次のいずれかを行ってください:

* ツール対応のモデルを pull して取得するか、または
* `models.providers.ollama` でモデルを明示的に定義します。

モデルを追加するには:

```bash
ollama list  # インストール済みのものを確認
ollama pull llama3.3  # モデルをプル
```

<div id="connection-refused">
  ### 接続が拒否されました
</div>

Ollama が正しいポートで起動していることを確認してください：

```bash
# Ollamaが実行中かどうかを確認
ps aux | grep ollama

# またはOllamaを再起動
ollama serve
```

<div id="see-also">
  ## 関連項目
</div>

* [モデルプロバイダー](/ja/concepts/model-providers) - すべてのプロバイダーの概要
* [モデル選択](/ja/concepts/models) - モデルの選び方
* [設定](/ja/gateway/configuration) - 設定の完全なリファレンス