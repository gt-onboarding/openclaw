---
title: 位置情報
summary: "インバウンドチャネルの位置情報解析（Telegram と WhatsApp）およびコンテキストフィールド"
read_when:
  - チャネルの位置情報解析を追加または変更するとき
  - エージェントのプロンプトやツールで位置情報のコンテキストフィールドを使用するとき
---

<div id="channel-location-parsing">
  # チャンネル位置情報のパース
</div>

OpenClaw はチャットチャンネルから共有された位置情報を、次の 2 つの形に正規化します。

- 受信メッセージ本文の末尾に付加される、人間が読める形式のテキスト
- 自動返信コンテキストのペイロード内に含まれる構造化フィールド

現在サポートしているのは次のとおりです。

- **Telegram**（位置ピン + 会場情報 + ライブ位置情報）
- **WhatsApp**（`locationMessage` + `liveLocationMessage`）
- **Matrix**（`geo_uri` を含む `m.location`）

<div id="text-formatting">
  ## テキスト形式
</div>

位置情報は、角括弧などを使わない読みやすい1行のテキストとして表示されます:

* ピン:
  * `📍 48.858844, 2.294351 ±12m`
* 名前付きの場所:
  * `📍 Eiffel Tower — Champ de Mars, Paris (48.858844, 2.294351 ±12m)`
* ライブ位置共有:
  * `🛰 Live location: 48.858844, 2.294351 ±12m`

チャネルにキャプション／コメントが含まれている場合は、その次の行に追記されます:

```
📍 48.858844, 2.294351 ±12m
ここで会おう
```


<div id="context-fields">
  ## コンテキストフィールド
</div>

位置情報が利用可能な場合、これらのフィールドが `ctx` に追加されます：

- `LocationLat` (数値)
- `LocationLon` (数値)
- `LocationAccuracy` (数値、メートル単位、省略可)
- `LocationName` (文字列、省略可)
- `LocationAddress` (文字列、省略可)
- `LocationSource` (`pin | place | live`)
- `LocationIsLive` (真偽値)

<div id="channel-notes">
  ## チャンネルに関する注意事項
</div>

- **Telegram**: venue は `LocationName/LocationAddress` に対応します。ライブ位置情報では `live_period` が使用されます。
- **WhatsApp**: `locationMessage.comment` と `liveLocationMessage.caption` はキャプション行として末尾に追加されます。
- **Matrix**: `geo_uri` はピン位置として解釈されます。高度は無視され、`LocationIsLive` は常に false です。