---
title: ボイスウェイク
summary: "グローバルな音声ウェイクワード（Gateway 管理）と、それらがノード間でどのように同期されるか"
read_when:
  - 音声ウェイクワードの動作やデフォルト設定を変更するとき
  - ウェイクワード同期が必要な新しいノードプラットフォームを追加するとき
---

<div id="voice-wake-global-wake-words">
  # Voice Wake（グローバルウェイクワード）
</div>

OpenClaw は、**ウェイクワードを Gateway が所有する単一のグローバルリスト**として扱います。

- **ノードごとのカスタムウェイクワードはありません**。
- **任意のノード／アプリの UI からこのリストを編集できます**。変更は Gateway によって永続化され、すべてのノード／クライアントにブロードキャストされます。
- 各デバイスは引き続き、自身の **Voice Wake の有効／無効トグル**を個別に保持します（ローカルの UX や権限はデバイスごとに異なります）。

<div id="storage-gateway-host">
  ## ストレージ（Gateway ホスト）
</div>

ウェイクワードは、Gateway が動作しているマシン上の次の場所に保存されます。

* `~/.openclaw/settings/voicewake.json`

構造:

```json
{ "triggers": ["openclaw", "claude", "computer"], "updatedAtMs": 1730000000000 }
```


<div id="protocol">
  ## プロトコル
</div>

<div id="methods">
  ### Methods
</div>

- `voicewake.get` → `{ triggers: string[] }`
- `voicewake.set` with params `{ triggers: string[] }` → `{ triggers: string[] }`

Notes:

- トリガーは正規化されます（前後の空白を削除し、空要素は破棄されます）。空リストの場合はデフォルト値にフォールバックします。
- 安全性のために制限（件数／文字数の上限）が適用されます。

<div id="events">
  ### Events
</div>

- `voicewake.changed` payload `{ triggers: string[] }`

受信対象:

- すべての WebSocket クライアント（macOS アプリ、WebChat など）
- 接続中のすべてのノード（iOS/Android）。また、ノード接続時には、初期の「現在の状態」を通知するプッシュとしても送信されます。

<div id="client-behavior">
  ## クライアントの挙動
</div>

<div id="macos-app">
  ### macOS アプリ
</div>

- グローバルリストを使用して、`VoiceWakeRuntime` トリガーの発火を制御します。
- Voice Wake 設定で「トリガーワード」を編集すると、`voicewake.set` が呼び出され、その後はブロードキャストを利用して他のクライアントとの同期を保ちます。

<div id="ios-node">
  ### iOSノード
</div>

- `VoiceWakeManager` のトリガー検出のためにグローバルリストを使用します。
- 設定でウェイクワードを編集すると、Gateway の WS 経由で `voicewake.set` を呼び出すと同時に、ローカルのウェイクワード検出も引き続き高い応答性を維持します。

<div id="android-node">
  ### Android ノード
</div>

- 設定にウェイクワードエディタを表示します。
- 編集内容が全クライアントで同期されるよう、Gateway の WS 経由で `voicewake.set` を呼び出します。