---
title: 音声
summary: "受信した音声／ボイスメモがどのようにダウンロード・文字起こしされ、返信に組み込まれるか"
read_when:
  - 音声の文字起こしやメディア処理の設定を変更するとき
---

<div id="audio-voice-notes-2026-01-17">
  # 音声 / ボイスメモ — 2026-01-17
</div>

<div id="what-works">
  ## 動作する機能
</div>

* **メディア理解（音声）**: 音声理解が有効（または自動検出）になっている場合、OpenClaw は次を行います:
  1. 最初の音声添付ファイル（ローカルパスまたは URL）を特定し、必要に応じてダウンロードします。
  2. 各モデルエントリに送信する前に、`maxBytes` の上限を強制します。
  3. 順番に、最初に利用可能なモデルエントリ（プロバイダーまたは CLI）を実行します。
  4. 失敗したり（サイズ/タイムアウト）スキップされた場合、次のエントリを試行します。
  5. 成功すると、`Body` を `[Audio]` ブロックに置き換え、`{{Transcript}}` を設定します。
* **コマンド解析**: 書き起こしに成功すると、スラッシュコマンドが引き続き動作するように、`CommandBody` / `RawBody` にトランスクリプトを設定します。
* **詳細ログ出力**: `--verbose` モードでは、書き起こしの実行タイミングと本文が置き換えられたタイミングをログに記録します。

<div id="auto-detection-default">
  ## 自動検出（デフォルト）
</div>

**モデルを設定しておらず**、かつ `tools.media.audio.enabled` が `false` に **なっていない** 場合、
OpenClaw は次の順序で自動検出を行い、最初に利用可能だったオプションで処理を停止します:

1. **ローカル CLI**（インストール済みの場合）
   * `sherpa-onnx-offline`（encoder/decoder/joiner/tokens を含む `SHERPA_ONNX_MODEL_DIR` が必要）
   * `whisper-cli`（`whisper-cpp` 由来；`WHISPER_CPP_MODEL` または同梱の tiny モデルを使用）
   * `whisper`（Python CLI；モデルを自動ダウンロード）
2. `read_many_files` を用いた **Gemini CLI**（`gemini`）
3. **プロバイダーキー**（OpenAI → Groq → Deepgram → Google）

自動検出を無効化するには、`tools.media.audio.enabled: false` を設定します。
カスタマイズする場合は、`tools.media.audio.models` を設定します。
注意: バイナリの検出は macOS/Linux/Windows 間でベストエフォートです。CLI が `PATH` 上に存在すること（`~` は展開されます）を確認するか、フルパスを含むコマンドで明示的に CLI モデルを指定してください。

<div id="config-examples">
  ## 設定例
</div>

<div id="provider-cli-fallback-openai-whisper-cli">
  ### プロバイダー + CLI フォールバック（OpenAI + Whisper CLI）
</div>

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        maxBytes: 20971520,
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          {
            type: "cli",
            command: "whisper",
            args: ["--model", "base", "{{MediaPath}}"],
            timeoutSeconds: 45
          }
        ]
      }
    }
  }
}
```

<div id="provider-only-with-scope-gating">
  ### スコープ制御付きのプロバイダー専用
</div>

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        scope: {
          default: "allow",
          rules: [
            { action: "deny", match: { chatType: "group" } }
          ]
        },
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" }
        ]
      }
    }
  }
}
```

<div id="provider-only-deepgram">
  ### プロバイダー限定（Deepgram）
</div>

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "deepgram", model: "nova-3" }]
      }
    }
  }
}
```

<div id="notes-limits">
  ## 注意点と制限事項
</div>

* プロバイダー認証は標準のモデル認証順序（認証プロファイル、環境変数、`models.providers.*.apiKey`）に従います。
* `provider: "deepgram"` が使用されている場合、Deepgram は `DEEPGRAM_API_KEY` を自動的に取得します。
* Deepgram のセットアップの詳細: [Deepgram (audio transcription)](/ja/providers/deepgram)。
* 音声プロバイダーは、`tools.media.audio` 経由で `baseUrl`、`headers`、`providerOptions` を上書きできます。
* デフォルトのサイズ上限は 20MB（`tools.media.audio.maxBytes`）です。上限を超える音声はそのモデルではスキップされ、次のエントリが試行されます。
* 音声のデフォルトの `maxChars` は **未設定**（全文文字起こし）です。出力をトリミングするには、`tools.media.audio.maxChars` またはエントリごとの `maxChars` を設定してください。
* OpenAI のデフォルトは `gpt-4o-mini-transcribe` です。より高精度にするには `model: "gpt-4o-transcribe"` を設定してください。
* 複数のボイスノートを処理するには、`tools.media.audio.attachments` を使用します（`mode: "all"` + `maxAttachments`）。
* テンプレートからは文字起こし結果に `{{Transcript}}` としてアクセスできます。
* CLI の標準出力は 5MB に制限されます。CLI の出力は簡潔に保ってください。

<div id="gotchas">
  ## ハマりどころ / 注意点
</div>

* スコープのルールは、先にマッチしたものを優先する「first match wins」方式です。`chatType` は `direct`、`group`、`room` のいずれかに正規化されます。
* CLI は終了コード 0 で終了し、プレーンテキストを出力するようにしてください。JSON からは `jq -r .text` を使ってプレーンテキストを取り出してください。
* タイムアウト値（`timeoutSeconds`、デフォルト 60 秒）は、返信キューをブロックしない程度に妥当な範囲で設定してください。