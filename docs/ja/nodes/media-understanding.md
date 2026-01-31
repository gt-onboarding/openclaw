---
title: メディア理解
summary: "プロバイダーとCLIによるフォールバック付きの、受信画像／音声／動画の理解（オプション）"
read_when:
  - メディア理解を設計またはリファクタリングするとき
  - 受信画像／音声／動画の前処理を調整するとき
---

<div id="media-understanding-inbound-2026-01-17">
  # メディア理解（インバウンド） — 2026-01-17
</div>

OpenClaw は、応答パイプラインが実行される前に、受信メディア（画像／音声／動画）を**要約**できます。ローカルツールやプロバイダーキーの有無を自動検出し、機能を無効化したりカスタマイズしたりできます。メディア理解がオフの場合でも、モデルは通常どおり元のファイルや URL を受け取ります。

<div id="goals">
  ## 目的
</div>

* （任意）受信メディアを事前に短いテキストへ要約し、ルーティングの高速化とコマンド解析の精度向上を行えるようにする。
* 元のメディアを常にモデルにそのまま渡せるようにしておく。
* **プロバイダー API** と **CLI フォールバック** をサポートする。
* 複数モデルを設定し、エラー／サイズ／タイムアウト時に優先順位付きでフォールバックできるようにする。

<div id="highlevel-behavior">
  ## 高レベルな動作
</div>

1. 受信した添付ファイル（`MediaPaths`, `MediaUrls`, `MediaTypes`）を収集する。
2. 有効化されている各ケイパビリティ（画像 / 音声 / 動画）ごとに、ポリシーに従って添付ファイルを選択する（デフォルト: **先頭**）。
3. 最初の適格なモデルエントリ（サイズ + ケイパビリティ + 認証）を選択する。
4. モデルの実行が失敗するかメディアが大きすぎる場合、**次のエントリにフォールバックする**。
5. 成功時:
   * `Body` は `[Image]`、`[Audio]`、または `[Video]` ブロックになる。
   * 音声の場合は `{{Transcript}}` を設定し、コマンド解析ではキャプションテキストがあればそれを使用し、
     なければトランスクリプトを使用する。
   * キャプションはブロック内の `User text:` として保持される。

メディア理解に失敗するか、または無効化されている場合でも、**応答フローは続行され**、元の本文 + 添付ファイルが使用される。

<div id="config-overview">
  ## コンフィグ概要
</div>

`tools.media` は、**共有モデル** と機能ごとのオーバーライドをサポートします:

* `tools.media.models`: 共有モデル一覧（`capabilities` での制御に利用）。
* `tools.media.image` / `tools.media.audio` / `tools.media.video`:
  * デフォルト値（`prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language`）
  * プロバイダーごとのオーバーライド（`baseUrl`, `headers`, `providerOptions`）
  * `tools.media.audio.providerOptions.deepgram` 経由の Deepgram 向け音声オプション
  * 任意の **機能ごとの `models` リスト**（共有モデルより優先）
  * `attachments` ポリシー（`mode`, `maxAttachments`, `prefer`）
  * `scope`（channel/chatType/セッションキーによる任意のゲーティング）
* `tools.media.concurrency`: 機能の同時実行数の上限（デフォルト **2**）。

```json5
{
  tools: {
    media: {
      models: [ /* 共有リスト */ ],
      image: { /* optional overrides */ },
      audio: { /* optional overrides */ },
      video: { /* optional overrides */ }
    }
  }
}
```

<div id="model-entries">
  ### モデルエントリ
</div>

各 `models[]` エントリには、**プロバイダー** または **CLI** を指定できます。

```json5
{
  type: "provider",        // default if omitted
  provider: "openai",
  model: "gpt-5.2",
  prompt: "Describe the image in <= 500 chars.",
  maxChars: 500,
  maxBytes: 10485760,
  timeoutSeconds: 60,
  capabilities: ["image"], // オプション、マルチモーダルエントリで使用
  profile: "vision-profile",
  preferredProfile: "vision-fallback"
}
```

```json5
{
  type: "cli",
  command: "gemini",
  args: [
    "-m",
    "gemini-3-flash",
    "--allowed-tools",
    "read_file",
    "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters."
  ],
  maxChars: 500,
  maxBytes: 52428800,
  timeoutSeconds: 120,
  capabilities: ["video", "image"]
}
```

CLI テンプレートでは次のものも利用できます:

* `{{MediaDir}}` (メディアファイルを含むディレクトリ)
* `{{OutputDir}}` (この実行用に作成された一時ディレクトリ)
* `{{OutputBase}}` (拡張子なしの一時ファイルのベースパス)

<div id="defaults-and-limits">
  ## 既定値と制限
</div>

推奨される既定値:

* `maxChars`: 画像/動画には **500**（短く、コマンド用途向き）
* `maxChars`: 音声には **未設定**（制限を設けない限り全文を書き起こし）
* `maxBytes`:
  * 画像: **10MB**
  * 音声: **20MB**
  * 動画: **50MB**

ルール:

* メディアが `maxBytes` を超えた場合、そのモデルはスキップされ、**次のモデルが試行される**。
* モデルが `maxChars` を超える出力を返した場合、出力はトリムされる。
* `prompt` の既定値は、シンプルな「{media} を説明してください。」に `maxChars` のガイダンスを加えたもの（画像/動画のみ）。
* `<capability>.enabled: true` だがモデルが一つも設定されていない場合、プロバイダーがその機能をサポートしていれば、
  OpenClaw は **アクティブな返信用モデル** を利用しようとする。

<div id="auto-detect-media-understanding-default">
  ### メディア理解の自動検出（デフォルト）
</div>

`tools.media.<capability>.enabled` が `false` に設定されておらず、かつモデルを
設定していない場合、OpenClaw は次の順番で自動検出を行い、**最初に正常に動作したオプションで停止します**:

1. **ローカル CLI**（音声のみ・インストール済みの場合）
   * `sherpa-onnx-offline`（encoder/decoder/joiner/tokens を含む `SHERPA_ONNX_MODEL_DIR` が必要）
   * `whisper-cli`（`whisper-cpp`; `WHISPER_CPP_MODEL` または同梱の tiny モデルを使用）
   * `whisper`（Python CLI; モデルを自動ダウンロード）
2. **Gemini CLI**（`gemini`）で `read_many_files` を使用
3. **プロバイダーキー**
   * 音声: OpenAI → Groq → Deepgram → Google
   * 画像: OpenAI → Anthropic → Google → MiniMax
   * 動画: Google

自動検出を無効化するには、次のように設定します:

```json5
{
  tools: {
    media: {
      audio: {
        enabled: false
      }
    }
  }
}
```

注記: バイナリの検出は macOS/Linux/Windows 間でベストエフォートです。CLI が `PATH` に含まれていること（`~` は展開されます）を確認するか、コマンドのフルパスを指定した明示的な CLI モデルを設定してください。

<div id="capabilities-optional">
  ## 機能 (任意)
</div>

`capabilities` を設定すると、そのエントリは指定したメディアタイプに対してのみ実行されます。共有リストの場合、OpenClaw は既定値を自動的に推定します:

* `openai`, `anthropic`, `minimax`: **image**
* `google` (Gemini API): **image + audio + video**
* `groq`: **audio**
* `deepgram`: **audio**

CLI エントリでは、予期しないマッチを防ぐために、**必ず `capabilities` を明示的に設定**してください。`capabilities` を省略した場合、そのエントリは記載されているリストの候補として扱われます。

<div id="provider-support-matrix-openclaw-integrations">
  ## プロバイダー対応マトリクス（OpenClaw 連携）
</div>

| 機能 | プロバイダー連携 | 備考 |
|------------|----------------------|-------|
| 画像 | OpenAI / Anthropic / Google / その他（`pi-ai` 経由） | レジストリに登録された画像対応モデルはすべて利用できます。 |
| 音声 | OpenAI, Groq, Deepgram, Google | プロバイダー提供の文字起こし機能（Whisper / Deepgram / Gemini）。 |
| 動画 | Google (Gemini API) | プロバイダー提供の動画理解機能。 |

<div id="recommended-providers">
  ## 推奨プロバイダー
</div>

**画像**

* 画像をサポートしている場合は、通常利用しているモデルを優先して使用する。
* 推奨デフォルト: `openai/gpt-5.2`, `anthropic/claude-opus-4-5`, `google/gemini-3-pro-preview`.

**音声**

* `openai/gpt-4o-mini-transcribe`, `groq/whisper-large-v3-turbo`, または `deepgram/nova-3`。
* CLI フォールバック: `whisper-cli` (whisper-cpp) または `whisper`。
* Deepgram のセットアップ: [Deepgram (audio transcription)](/ja/providers/deepgram)。

**動画**

* `google/gemini-3-flash-preview` (高速), `google/gemini-3-pro-preview` (より高機能)。
* CLI フォールバック: `gemini` CLI（動画/音声に対する `read_file` をサポート）。

<div id="attachment-policy">
  ## 添付ファイルポリシー
</div>

機能単位の `attachments` 設定で、処理対象とする添付ファイルを制御します:

* `mode`: `first`（デフォルト）または `all`
* `maxAttachments`: 処理する添付ファイル数の上限（デフォルト **1**）
* `prefer`: `first`, `last`, `path`, `url`

`mode: "all"` の場合、出力には `[Image 1/2]`、`[Audio 2/2]` などのラベルが付きます。

<div id="config-examples">
  ## 設定例
</div>

<div id="1-shared-models-list-overrides">
  ### 1) 共通モデルリストとオーバーライド
</div>

```json5
{
  tools: {
    media: {
      models: [
        { provider: "openai", model: "gpt-5.2", capabilities: ["image"] },
        { provider: "google", model: "gemini-3-flash-preview", capabilities: ["image", "audio", "video"] },
        {
          type: "cli",
          command: "gemini",
          args: [
            "-m",
            "gemini-3-flash",
            "--allowed-tools",
            "read_file",
            "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters."
          ],
          capabilities: ["image", "video"]
        }
      ],
      audio: {
        attachments: { mode: "all", maxAttachments: 2 }
      },
      video: {
        maxChars: 500
      }
    }
  }
}
```

<div id="2-audio-video-only-image-off">
  ### 2) 音声＋映像のみ（画像オフ）
</div>

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          {
            type: "cli",
            command: "whisper",
            args: ["--model", "base", "{{MediaPath}}"]
          }
        ]
      },
      video: {
        enabled: true,
        maxChars: 500,
        models: [
          { provider: "google", model: "gemini-3-flash-preview" },
          {
            type: "cli",
            command: "gemini",
            args: [
              "-m",
              "gemini-3-flash",
              "--allowed-tools",
              "read_file",
              "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters."
            ]
          }
        ]
      }
    }
  }
}
```

<div id="3-optional-image-understanding">
  ### 3) オプションの画像理解
</div>

```json5
{
  tools: {
    media: {
      image: {
        enabled: true,
        maxBytes: 10485760,
        maxChars: 500,
        models: [
          { provider: "openai", model: "gpt-5.2" },
          { provider: "anthropic", model: "claude-opus-4-5" },
          {
            type: "cli",
            command: "gemini",
            args: [
              "-m",
              "gemini-3-flash",
              "--allowed-tools",
              "read_file",
              "{{MediaPath}}のメディアを読み取り、{{MaxChars}}文字以内で説明してください。"
            ]
          }
        ]
      }
    }
  }
}
```

<div id="4-multimodal-single-entry-explicit-capabilities">
  ### 4) マルチモーダル単一エントリポイント（ケイパビリティの明示）
</div>

```json5
{
  tools: {
    media: {
      image: { models: [{ provider: "google", model: "gemini-3-pro-preview", capabilities: ["image", "video", "audio"] }] },
      audio: { models: [{ provider: "google", model: "gemini-3-pro-preview", capabilities: ["image", "video", "audio"] }] },
      video: { models: [{ provider: "google", model: "gemini-3-pro-preview", capabilities: ["image", "video", "audio"] }] }
    }
  }
}
```

<div id="status-output">
  ## ステータス出力
</div>

メディア理解機能が実行中の場合、`/status` に短い要約行が含まれます。

```
📎 Media: image ok (openai/gpt-5.2) · audio skipped (maxBytes)
```

これは、各機能ごとの結果と、該当する場合に選択されたプロバイダーやモデルを示します。

<div id="notes">
  ## 注意事項
</div>

* 理解は**ベストエフォート**です。エラーが発生しても返信は中断されません。
* 理解が無効な場合でも、添付ファイルはモデルに渡されます。
* `scope` を使用して、理解を実行する範囲を制限します（例: DM のみに限定）。

<div id="related-docs">
  ## 関連ドキュメント
</div>

* [設定](/ja/gateway/configuration)
* [画像・メディア対応](/ja/nodes/images)