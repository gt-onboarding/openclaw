---
title: Deepgram
summary: "Deepgram による受信ボイスメモの文字起こし"
read_when:
  - 音声添付ファイルに Deepgram の音声文字起こし（speech-to-text）を使いたいとき
  - すぐに使える Deepgram の設定例が必要なとき
---

<div id="deepgram-audio-transcription">
  # Deepgram（音声文字起こし）
</div>

Deepgram は音声をテキストに変換する API です。OpenClaw では、`tools.media.audio` を通じて **受信音声／ボイスメモの文字起こし** に使用されます。

有効化されている場合、OpenClaw は音声ファイルを Deepgram にアップロードし、書き起こされたテキストを応答パイプライン（`{{Transcript}}` + `[Audio]` ブロック）に挿入します。これは **ストリーミングではなく**、あらかじめ録音された音声向けの文字起こしエンドポイントを使用します。

Website: https://deepgram.com  
Docs: https://developers.deepgram.com

<div id="quick-start">
  ## クイックスタート
</div>

1. API キーを設定する：

```
DEEPGRAM_API_KEY=dg_...
```

2. プロバイダーを有効にします：

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


<div id="options">
  ## オプション
</div>

* `model`: Deepgram のモデル ID（デフォルト: `nova-3`）
* `language`: 言語のヒント（任意）
* `tools.media.audio.providerOptions.deepgram.detect_language`: 言語検出を有効化（任意）
* `tools.media.audio.providerOptions.deepgram.punctuate`: 句読点の付与を有効化（任意）
* `tools.media.audio.providerOptions.deepgram.smart_format`: スマートフォーマット機能を有効化（任意）

言語を指定する例:

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [
          { provider: "deepgram", model: "nova-3", language: "en" }
        ]
      }
    }
  }
}
```

Deepgram オプションの使用例:

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        providerOptions: {
          deepgram: {
            detect_language: true,
            punctuate: true,
            smart_format: true
          }
        },
        models: [{ provider: "deepgram", model: "nova-3" }]
      }
    }
  }
}
```


<div id="notes">
  ## 補足
</div>

- 認証は標準的なプロバイダーの認証順序に従います。最も簡単な方法は `DEEPGRAM_API_KEY` を利用することです。
- プロキシを使用する場合は、`tools.media.audio.baseUrl` と `tools.media.audio.headers` でエンドポイントやヘッダーをオーバーライドします。
- 出力も、他のプロバイダーと同じ音声関連ルール（サイズ上限、タイムアウト、文字起こしのインジェクション）に従います。