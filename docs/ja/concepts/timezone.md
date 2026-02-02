---
title: タイムゾーン
summary: "エージェント、エンベロープ、プロンプトにおけるタイムゾーン処理"
read_when:
  - モデル用にタイムスタンプがどのように正規化されるかを理解する必要がある場合
  - システムプロンプト向けにユーザーのタイムゾーンを設定する場合
---

<div id="timezones">
  # タイムゾーン
</div>

OpenClaw はタイムスタンプを標準化し、モデルが **単一の基準時刻** のみを扱うようにします。

<div id="message-envelopes-local-by-default">
  ## メッセージエンベロープ（デフォルトではローカル）
</div>

受信メッセージは次のようなエンベロープにラップされます:

```
[Provider ... 2026-01-05 16:26 PST] message text
```

エンベロープ内のタイムスタンプは、デフォルトではホストローカル時刻で、分単位の精度です。

これを変更するには、次のようにします:

```json5
{
  agents: {
    defaults: {
      envelopeTimezone: "local", // "utc" | "local" | "user" | IANAタイムゾーン
      envelopeTimestamp: "on", // "on" | "off"
      envelopeElapsed: "on" // "on" | "off"
    }
  }
}
```

* `envelopeTimezone: "utc"` は UTC を使用します。
* `envelopeTimezone: "user"` は `agents.defaults.userTimezone` を使用します（未設定時はホストのタイムゾーンを使用）。
* 固定オフセットには、明示的な IANA タイムゾーン（例: `"Europe/Vienna"`）を使用します。
* `envelopeTimestamp: "off"` はエンベロープヘッダーから絶対タイムスタンプを削除します。
* `envelopeElapsed: "off"` は経過時間サフィックス（`+2m` 形式）を削除します。

<div id="examples">
  ### 例
</div>

**ローカル（デフォルト）:**

```
[Signal Alice +1555 2026-01-18 00:19 PST] hello
```

**固定されたタイムゾーン:**

```
[Signal Alice +1555 2026-01-18 06:19 GMT+1] hello
```

**経過時間：**

```
[Signal Alice +1555 +2m 2026-01-18T05:19Z] follow-up
```

<div id="tool-payloads-raw-provider-data-normalized-fields">
  ## ツールのペイロード（生のプロバイダーデータ + 正規化済みフィールド）
</div>

ツール呼び出し（`channels.discord.readMessages`、`channels.slack.readMessages` など）は、**生のプロバイダータイムスタンプ**を返します。
一貫性のために、正規化済みフィールドも併せて付与します:

* `timestampMs`（UTC エポックミリ秒）
* `timestampUtc`（ISO 8601 形式の UTC 文字列）

生のプロバイダーフィールドはそのまま保持されます。

<div id="user-timezone-for-the-system-prompt">
  ## システムプロンプト用のユーザータイムゾーン
</div>

`agents.defaults.userTimezone` を設定して、モデルにユーザーのローカルタイムゾーンを知らせます。これが未設定の場合、OpenClaw は **実行時にホストのタイムゾーンを取得** します（設定ファイルへの書き込みは行いません）。

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } }
}
```

システムプロンプトには次が含まれます：

* ローカル時刻とタイムゾーンを含む `Current Date & Time` セクション
* `Time format: 12-hour` または `24-hour`

`agents.defaults.timeFormat`（`auto` | `12` | `24`）でプロンプトの形式を制御できます。

挙動の詳細と例については [Date &amp; Time](/ja/date-time) を参照してください。
