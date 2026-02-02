---
title: トーク
summary: "Talk モード: ElevenLabs TTS を使った連続音声会話"
read_when:
  - macOS/iOS/Android で Talk モードを実装するとき
  - 音声/TTS/割り込み動作を変更するとき
---

<div id="talk-mode">
  # トークモード
</div>

トークモードは、音声による連続的な会話ループです:

1) 音声を認識する
2) 書き起こしをモデルに送信する（メインセッション、chat.send）
3) 応答を待機する
4) ElevenLabs で読み上げる（ストリーミング再生）

<div id="behavior-macos">
  ## 動作 (macOS)
</div>

- Talk モードが有効な間、**常時表示のオーバーレイ** が表示されます。
- **Listening → Thinking → Speaking** の各フェーズ間を遷移します。
- **短いポーズ**（無音区間）があると、その時点までのトランスクリプトを送信します。
- 応答は **WebChat に書き込まれます**（手入力して送信する場合と同じ動作）。
- **発話による割り込み**（デフォルト有効）：アシスタントが話している最中にユーザーが話し始めた場合、再生を停止し、次回プロンプト用に割り込みが発生したタイムスタンプを記録します。

<div id="voice-directives-in-replies">
  ## 返信内での音声ディレクティブ
</div>

アシスタントは、音声を制御するために返信の先頭に **1 行の JSON** を付けることがあります。

```json
{"voice":"<voice-id>","once":true}
```

ルール:

* 最初の空でない行のみが対象です。
* 不明なキーは無視されます。
* `once: true` は現在のレスポンスのみに適用されます。
* `once` がない場合、そのボイスは Talk モードの新しいデフォルトになります。
* JSON 行は TTS 再生前に削除されます。

サポートされているキー:

* `voice` / `voice_id` / `voiceId`
* `model` / `model_id` / `modelId`
* `speed`, `rate` (WPM), `stability`, `similarity`, `style`, `speakerBoost`
* `seed`, `normalize`, `lang`, `output_format`, `latency_tier`
* `once`


<div id="config-openclawopenclawjson">
  ## 設定 (`~/.openclaw/openclaw.json`)
</div>

```json5
{
  "talk": {
    "voiceId": "elevenlabs_voice_id",
    "modelId": "eleven_v3",
    "outputFormat": "mp3_44100_128",
    "apiKey": "elevenlabs_api_key",
    "interruptOnSpeech": true
  }
}
```

デフォルト:

* `interruptOnSpeech`: true
* `voiceId`: `ELEVENLABS_VOICE_ID` / `SAG_VOICE_ID`（または API キーが利用可能な場合は ElevenLabs の最初の音声）にフォールバック
* `modelId`: 未設定の場合は `eleven_v3` をデフォルトとして使用
* `apiKey`: `ELEVENLABS_API_KEY`（または利用可能な場合は Gateway のシェルプロファイル）にフォールバック
* `outputFormat`: macOS/iOS では `pcm_44100`、Android では `pcm_24000` がデフォルト（MP3 ストリーミングを強制するには `mp3_*` を設定）


<div id="macos-ui">
  ## macOS UI
</div>

- メニューバーのトグルボタン: **Talk**
- 設定タブ: **Talk Mode** グループ（voice id ＋ 割り込みトグル）
- オーバーレイ:
  - **Listening**: マイク入力レベルに応じてクラウドがパルス表示
  - **Thinking**: 沈み込むアニメーション
  - **Speaking**: 放射状のリング
  - クラウドをクリック: 発話を停止
  - X をクリック: Talk モードを終了する

<div id="notes">
  ## 注意事項
</div>

- 音声認識とマイクへのアクセス権限が必要です。
- `chat.send` をセッションキー `main` に対して使用します。
- TTS は ElevenLabs のストリーミング API（`ELEVENLABS_API_KEY` を使用）を利用し、macOS/iOS/Android ではレイテンシ低減のための逐次再生を行います。
- `eleven_v3` 用の `stability` は `0.0`、`0.5`、`1.0` のいずれかのみ有効です。他のモデルでは `0..1` を指定できます。
- `latency_tier` は設定されている場合、`0..4` の範囲で検証されます。
- Android は低レイテンシの AudioTrack ストリーミング用に、`pcm_16000`、`pcm_22050`、`pcm_24000`、`pcm_44100` の出力フォーマットをサポートします。