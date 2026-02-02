---
title: Tts
summary: "返信のテキスト読み上げ（TTS）"
read_when:
  - 返信のテキスト読み上げ（TTS）を有効化するとき
  - TTS プロバイダーや上限を設定するとき
  - /tts コマンドを使用するとき
---

<div id="text-to-speech-tts">
  # テキスト読み上げ（TTS）
</div>

OpenClaw は、ElevenLabs、OpenAI、または Edge TTS を使用して、送信する返信メッセージを音声に変換できます。
OpenClaw が音声を送信できる場所であればどこでも動作し、Telegram ではボイスメッセージの丸いバブルとして表示されます。

<div id="supported-services">
  ## 対応サービス
</div>

* **ElevenLabs**（プライマリまたはフォールバック用のプロバイダー）
* **OpenAI**（プライマリまたはフォールバック用のプロバイダー。要約にも使用）
* **Edge TTS**（プライマリまたはフォールバック用のプロバイダー。`node-edge-tts` を使用し、API キーが未設定の場合のデフォルト）

<div id="edge-tts-notes">
  ### Edge TTS に関する注意事項
</div>

Edge TTS は `node-edge-tts`
ライブラリ経由で、Microsoft Edge のオンライン ニューラル TTS サービスを使用します。これはクラウド型サービス（ローカル実行ではない）であり、Microsoft のエンドポイントを利用し、API キーは不要です。`node-edge-tts` は音声の設定オプションや出力フォーマットを提供しますが、すべてのオプションが Edge 側のサービスでサポートされているわけではありません。 citeturn2search0

Edge TTS は公開 Web サービスであり、公式な SLA やクォータが公開されていないため、ベストエフォートのサービスとして扱ってください。保証された上限値やサポートが必要な場合は、OpenAI または ElevenLabs の利用を検討してください。Microsoft の Speech REST API では、リクエストあたり 10 分の音声制限が文書化されていますが、Edge TTS にはそのような制限の公開情報がないため、同等またはそれ以下の制限があると想定してください。 citeturn0search3

<div id="optional-keys">
  ## オプションのキー
</div>

OpenAI または ElevenLabs を利用する場合は、次を設定します:

* `ELEVENLABS_API_KEY`（または `XI_API_KEY`）
* `OPENAI_API_KEY`

Edge TTS には API キーは**不要**です。API キーが見つからない場合、OpenClaw は
デフォルトで Edge TTS を使用します（`messages.tts.edge.enabled=false` で無効化しない限り）。

複数のプロバイダーが設定されている場合、選択されたプロバイダーが優先的に使用され、その他はフォールバックオプションとして使用されます。
自動要約は、設定された `summaryModel`（または `agents.defaults.model.primary`）を使用するため、
要約を有効にする場合は、そのプロバイダーについても認証が必要です。

<div id="service-links">
  ## サービスへのリンク
</div>

* [OpenAI Text-to-Speech ガイド](https://platform.openai.com/docs/guides/text-to-speech)
* [OpenAI Audio API リファレンス](https://platform.openai.com/docs/api-reference/audio)
* [ElevenLabs Text to Speech](https://elevenlabs.io/docs/api-reference/text-to-speech)
* [ElevenLabs 認証](https://elevenlabs.io/docs/api-reference/authentication)
* [node-edge-tts](https://github.com/SchneeHertz/node-edge-tts)
* [Microsoft Speech の出力フォーマット](https://learn.microsoft.com/azure/ai-services/speech-service/rest-text-to-speech#audio-outputs)

<div id="is-it-enabled-by-default">
  ## デフォルトで有効になっていますか？
</div>

いいえ。Auto‑TTS はデフォルトでは**オフ**です。config で
`messages.tts.auto` を有効にするか、セッションごとに `/tts always`（エイリアス: `/tts on`）で有効化してください。

TTS がオンになっている場合、Edge TTS はデフォルトで**有効**になっており、
OpenAI または ElevenLabs の API キーが利用できないときに自動的に使用されます。

<div id="config">
  ## 設定
</div>

TTS 設定は `openclaw.json` の `messages.tts` セクションにあります。
完全なスキーマは [Gateway configuration](/ja/gateway/configuration) を参照してください。

<div id="minimal-config-enable-provider">
  ### 最小限の設定（有効化 + プロバイダー）
</div>

```json5
{
  messages: {
    tts: {
      auto: "always",
      provider: "elevenlabs"
    }
  }
}
```

<div id="openai-primary-with-elevenlabs-fallback">
  ### OpenAI をプライマリにし、ElevenLabs をフォールバックにする
</div>

```json5
{
  messages: {
    tts: {
      auto: "always",
      provider: "openai",
      summaryModel: "openai/gpt-4.1-mini",
      modelOverrides: {
        enabled: true
      },
      openai: {
        apiKey: "openai_api_key",
        model: "gpt-4o-mini-tts",
        voice: "alloy"
      },
      elevenlabs: {
        apiKey: "elevenlabs_api_key",
        baseUrl: "https://api.elevenlabs.io",
        voiceId: "voice_id",
        modelId: "eleven_multilingual_v2",
        seed: 42,
        applyTextNormalization: "auto",
        languageCode: "en",
        voiceSettings: {
          stability: 0.5,
          similarityBoost: 0.75,
          style: 0.0,
          useSpeakerBoost: true,
          speed: 1.0
        }
      }
    }
  }
}
```

<div id="edge-tts-primary-no-api-key">
  ### Edge TTS プライマリ（API キー不要）
</div>

```json5
{
  messages: {
    tts: {
      auto: "always",
      provider: "edge",
      edge: {
        enabled: true,
        voice: "en-US-MichelleNeural",
        lang: "en-US",
        outputFormat: "audio-24khz-48kbitrate-mono-mp3",
        rate: "+10%",
        pitch: "-5%"
      }
    }
  }
}
```

<div id="disable-edge-tts">
  ### Edge TTS を無効にする
</div>

```json5
{
  messages: {
    tts: {
      edge: {
        enabled: false
      }
    }
  }
}
```

<div id="custom-limits-prefs-path">
  ### カスタム制限と設定ファイルパス
</div>

```json5
{
  messages: {
    tts: {
      auto: "always",
      maxTextLength: 4000,
      timeoutMs: 30000,
      prefsPath: "~/.openclaw/settings/tts.json"
    }
  }
}
```

<div id="only-reply-with-audio-after-an-inbound-voice-note">
  ### 受信した音声メッセージにのみ音声で返信
</div>

```json5
{
  messages: {
    tts: {
      auto: "inbound"
    }
  }
}
```

<div id="disable-auto-summary-for-long-replies">
  ### 長い返信に対する自動要約を無効にする
</div>

```json5
{
  messages: {
    tts: {
      auto: "always"
    }
  }
}
```

次を実行してください:

```
/tts summary off
```

<div id="notes-on-fields">
  ### フィールドに関する注意
</div>

* `auto`: 自動 TTS モード（`off`、`always`、`inbound`、`tagged`）。
  * `inbound` は、受信したボイスメモの後のみ音声を送信します。
  * `tagged` は、返信に `[[tts]]` タグが含まれている場合にのみ音声を送信します。
* `enabled`: レガシー用トグル（`doctor` がこれを `auto` に移行します）。
* `mode`: `"final"`（デフォルト）または `"all"`（ツール/ブロックの返信も含む）。
* `provider`: `"elevenlabs"`、`"openai"`、または `"edge"`（フォールバックは自動）。
* `provider` が **未設定** の場合、OpenClaw は `openai`（キーがあれば）、次に `elevenlabs`（キーがあれば）を優先し、
  それ以外は `edge` を使用します。
* `summaryModel`: 自動要約用のオプションの低コストモデル。デフォルトは `agents.defaults.model.primary`。
  * `provider/model` 形式または設定済みのモデルエイリアスを受け付けます。
* `modelOverrides`: モデルが TTS ディレクティブを出力できるようにします（デフォルトで有効）。
* `maxTextLength`: TTS 入力のハード上限（文字数）。超過すると `/tts audio` は失敗します。
* `timeoutMs`: リクエストのタイムアウト（ミリ秒）。
* `prefsPath`: ローカルの prefs JSON パス（provider/limit/summary）を上書きします。
* `apiKey` の値は、未設定の場合に環境変数（`ELEVENLABS_API_KEY`/`XI_API_KEY`、`OPENAI_API_KEY`）がフォールバックとして使用されます。
* `elevenlabs.baseUrl`: ElevenLabs API のベース URL を上書きします。
* `elevenlabs.voiceSettings`:
  * `stability`, `similarityBoost`, `style`: `0..1`
  * `useSpeakerBoost`: `true|false`
  * `speed`: `0.5..2.0`（1.0 = 通常）
* `elevenlabs.applyTextNormalization`: `auto|on|off`
* `elevenlabs.languageCode`: 2 文字の ISO 639-1 コード（例: `en`, `de`）
* `elevenlabs.seed`: 整数 `0..4294967295`（決定性向上のベストエフォート）
* `edge.enabled`: Edge TTS の利用を許可（デフォルト `true`。API キー不要）。
* `edge.voice`: Edge ニューラルボイス名（例: `en-US-MichelleNeural`）。
* `edge.lang`: 言語コード（例: `en-US`）。
* `edge.outputFormat`: Edge の出力フォーマット（例: `audio-24khz-48kbitrate-mono-mp3`）。
  * 有効な値については Microsoft Speech の出力フォーマットを参照してください。すべてのフォーマットが Edge でサポートされているわけではありません。
* `edge.rate` / `edge.pitch` / `edge.volume`: パーセント文字列（例: `+10%`, `-5%`）。
* `edge.saveSubtitles`: 音声ファイルと併せて JSON 形式の字幕を書き出します。
* `edge.proxy`: Edge TTS リクエスト用のプロキシ URL。
* `edge.timeoutMs`: リクエストタイムアウトを上書きする値（ミリ秒）。

<div id="model-driven-overrides-default-on">
  ## モデル駆動のオーバーライド（デフォルトで有効）
</div>

デフォルトでは、モデルは単一の返信に対して TTS ディレクティブを出力でき**ます**。
`messages.tts.auto` が `tagged` の場合、音声をトリガーするにはこれらのディレクティブが必須です。

有効な場合、モデルは単一の返信で使用する音声を上書きするために `[[tts:...]]` ディレクティブを出力でき、さらに任意で、
笑い声や歌い始めなどの表現タグを含み、音声にのみ反映される
`[[tts:text]]...[[/tts:text]]` ブロックを追加できます。

返信ペイロードの例:

```
Here you go.

[[tts:provider=elevenlabs voiceId=pMsXgVXv3BLzUgSXRplE model=eleven_v3 speed=1.1]]
[[tts:text]](laughs) Read the song once more.[[/tts:text]]
```

有効化されている場合に使用できるディレクティブキー:

* `provider` (`openai` | `elevenlabs` | `edge`)
* `voice` (OpenAI の音声) または `voiceId` (ElevenLabs)
* `model` (OpenAI の TTS モデルまたは ElevenLabs のモデル ID)
* `stability`, `similarityBoost`, `style`, `speed`, `useSpeakerBoost`
* `applyTextNormalization` (`auto|on|off`)
* `languageCode` (ISO 639-1)
* `seed`

すべてのモデル設定の上書きを無効化:

```json5
{
  messages: {
    tts: {
      modelOverrides: {
        enabled: false
      }
    }
  }
}
```

オプションの許可リスト（タグを有効にしたまま特定のオーバーライドを無効化）:

```json5
{
  messages: {
    tts: {
      modelOverrides: {
        enabled: true,
        allowProvider: false,
        allowSeed: false
      }
    }
  }
}
```

<div id="per-user-preferences">
  ## ユーザーごとの設定
</div>

スラッシュコマンドはローカルのオーバーライド設定を `prefsPath` に書き込みます（デフォルト:
`~/.openclaw/settings/tts.json`。`OPENCLAW_TTS_PREFS` または
`messages.tts.prefsPath` で上書き可能）。

保存されるフィールドは次のとおりです:

* `enabled`
* `provider`
* `maxLength`（要約のしきい値。デフォルトは 1500 文字）
* `summarize`（デフォルトは `true`）

これらはそのホストに対する `messages.tts.*` を上書きします。

<div id="output-formats-fixed">
  ## 出力フォーマット（固定）
</div>

* **Telegram**: Opus ボイスメモ（ElevenLabs の `opus_48000_64`、OpenAI の `opus`）。
  * 48kHz / 64kbps はボイスメモ用途として良好なトレードオフであり、丸い吹き出し表示にも必須です。
* **その他のチャネル**: MP3（ElevenLabs の `mp3_44100_128`、OpenAI の `mp3`）。
  * 44.1kHz / 128kbps は音声の明瞭さとビットレートのバランスが取れたデフォルトです。
* **Edge TTS**: `edge.outputFormat` を使用（デフォルトは `audio-24khz-48kbitrate-mono-mp3`）。
  * `node-edge-tts` は `outputFormat` を受け取りますが、すべてのフォーマットが
    Edge サービス側で利用できるわけではありません。 citeturn2search0
  * 出力フォーマット値は Microsoft Speech の出力フォーマット（Ogg/WebM Opus を含む）に従います。 citeturn1search0
  * Telegram の `sendVoice` は OGG/MP3/M4A を受け付けます。Opus のボイスメモを確実に使いたい場合は OpenAI/ElevenLabs を使用してください。 citeturn1search1
  * 設定した Edge の出力フォーマットで失敗した場合、OpenClaw は MP3 でリトライします。

OpenAI/ElevenLabs のフォーマットは固定であり、Telegram はボイスメモの UX のために Opus を前提としています。

<div id="auto-tts-behavior">
  ## 自動 TTS の動作
</div>

有効化すると、OpenClaw は次のように動作します:

* 返信にすでにメディアが含まれているか、`MEDIA:` ディレクティブが含まれている場合は TTS をスキップします。
* 非常に短い返信（10 文字未満）はスキップします。
* 有効化されている場合、長い返信は `agents.defaults.model.primary`（または `summaryModel`）を使って要約します。
* 生成した音声を返信に添付します。

返信が `maxLength` を超えており、要約がオフ（または要約モデル用の API キーがない）になっている場合は、
音声生成はスキップされ、通常のテキスト返信のみが送信されます。

<div id="flow-diagram">
  ## フロー図
</div>

```
Reply -> TTS enabled?
  no  -> send text
  yes -> has media / MEDIA: / short?
          yes -> send text
          no  -> length > limit?
                   no  -> TTS -> attach audio
                   yes -> summary enabled?
                            no  -> send text
                            yes -> summarize (summaryModel or agents.defaults.model.primary)
                                      -> TTS -> attach audio
```

<div id="slash-command-usage">
  ## スラッシュコマンドの使用方法
</div>

利用できるコマンドは 1 つだけです: `/tts`。
有効化方法の詳細は [Slash commands](/ja/tools/slash-commands) を参照してください。

Discord に関する注意: `/tts` は Discord に組み込まれているコマンドのため、OpenClaw は
ネイティブコマンドとして `/voice` を登録します。テキストで `/tts ...` と入力する方法も引き続き利用できます。

```
/tts off
/tts always
/tts inbound
/tts tagged
/tts status
/tts provider openai
/tts limit 2000
/tts summary off
/tts audio Hello from OpenClaw
```

Notes:

* コマンドには認可された送信者が必要です（許可リスト／owner ルールは引き続き適用されます）。
* `commands.text` またはネイティブなコマンド登録を有効にする必要があります。
* `off|always|inbound|tagged` はセッションごとの切り替え設定です（`/tts on` は `/tts always` のエイリアスです）。
* `limit` と `summary` はメインの設定ではなくローカルの環境設定に保存されます。
* `/tts audio` はその一回限りの音声返信を生成します（TTS のオン／オフを切り替えるものではありません）。

<div id="agent-tool">
  ## エージェントツール
</div>

`tts` ツールはテキストを音声に変換し、`MEDIA:` パスを返します。結果が Telegram 互換の形式であれば、ツールは `[[audio_as_voice]]` を含めるため、Telegram 上ではボイスメッセージの吹き出しとして表示されます。

<div id="gateway-rpc">
  ## Gateway RPC
</div>

Gateway のメソッド:

* `tts.status`
* `tts.enable`
* `tts.disable`
* `tts.convert`
* `tts.setProvider`
* `tts.providers`