---
title: 入力中インジケーター
summary: "OpenClaw が入力中インジケーターを表示するタイミングと、その調整方法"
read_when:
  - 入力中インジケーターの動作やデフォルトを変更するとき
---

<div id="typing-indicators">
  # 入力中インジケーター
</div>

入力中インジケーターは、run の実行中にチャットチャンネルへ送信されます。
`agents.defaults.typingMode` を使って入力を開始する**タイミング**を制御し、`typingIntervalSeconds`
でインジケーターの更新**頻度**を制御します。

<div id="defaults">
  ## デフォルト
</div>

`agents.defaults.typingMode` が**未設定**の場合、OpenClaw は従来の動作を維持します：

- **ダイレクトチャット**：モデルループの実行が開始されると同時にタイピングが開始されます。
- **メンションありのグループチャット**：すぐにタイピングが開始されます。
- **メンションなしのグループチャット**：メッセージ本文のストリーミングが始まった時点でタイピングが開始されます。
- **ハートビート実行時**：タイピングは無効化されます。

<div id="modes">
  ## モード
</div>

`agents.defaults.typingMode` を次のいずれかに設定します。

- `never` — タイピングインジケーターを一切表示しません。
- `instant` — 実行が最終的にサイレント返信トークンのみを返す場合でも、**モデルループが開始された瞬間から** タイピングを開始します。
- `thinking` — **最初の reasoning delta** を受信したタイミングでタイピングを開始します（この実行には
  `reasoningLevel: "stream"` が必要です）。
- `message` — **最初のサイレントでないテキスト delta** を受信したタイミングでタイピングを開始します（
  `NO_REPLY` サイレントトークンは無視します）。

「どれだけ早く発火するか」の順序:
`never` → `message` → `thinking` → `instant`

<div id="configuration">
  ## 設定
</div>

```json5
{
  agent: {
    typingMode: "thinking",
    typingIntervalSeconds: 6
  }
}
```

セッション単位でモードや実行間隔を上書きできます。

```json5
{
  session: {
    typingMode: "message",
    typingIntervalSeconds: 4
  }
}
```


<div id="notes">
  ## 注記
</div>

- `message` モードでは、サイレント専用の応答（出力を抑制するために使用される `NO_REPLY`
  トークンなど）については入力中インジケーターを表示しません。
- `thinking` は、実行が推論をストリーミングする場合（`reasoningLevel: "stream"`）にのみ発生します。
  モデルが推論デルタを出力しない場合、入力中インジケーターは開始されません。
- ハートビートでは、モードに関係なく入力中インジケーターは表示されません。
- `typingIntervalSeconds` は、**更新の間隔** を制御するものであり、開始タイミングを制御するものではありません。
  デフォルトは 6 秒です。