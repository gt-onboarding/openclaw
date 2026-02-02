---
title: 日付と時刻
summary: "エンベロープ、プロンプト、ツール、コネクタ間での日付と時刻の扱い"
read_when:
  - タイムスタンプのモデルやユーザー向けの表示方法を変更しているとき
  - メッセージやシステムプロンプト出力の時刻フォーマットをデバッグしているとき
---

<div id="date-time">
  # 日付と時刻
</div>

OpenClaw はデフォルトで、**メッセージ伝送時のタイムスタンプにはホストローカル時間**を使用し、**システムプロンプト内でのみユーザーのタイムゾーン**を使用します。
プロバイダー側のタイムスタンプはそのまま保持されるため、ツールは本来のセマンティクスを維持します（現在時刻は `session_status` から取得できます）。

<div id="message-envelopes-local-by-default">
  ## メッセージエンベロープ（デフォルトはローカル時刻）
</div>

受信メッセージには、タイムスタンプ（分単位の精度）が付与されます：

```
[Provider ... 2026-01-05 16:26 PST] message text
```

このエンベロープのタイムスタンプは、プロバイダーのタイムゾーンに関係なく、**デフォルトではホストローカル時刻**になります。

この動作はオーバーライドできます。

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
* `envelopeTimezone: "local"` はホストのタイムゾーンを使用します。
* `envelopeTimezone: "user"` は `agents.defaults.userTimezone` を使用します（設定がない場合はホストのタイムゾーンにフォールバックします）。
* 固定したタイムゾーンを使いたい場合は、明示的な IANA タイムゾーン（例: `"America/Chicago"`）を指定します。
* `envelopeTimestamp: "off"` はエンベロープヘッダーから絶対タイムスタンプを削除します。
* `envelopeElapsed: "off"` は経過時間サフィックス（`+2m` 形式）を削除します。

<div id="examples">
  ### 使用例
</div>

**ローカル（デフォルト）：**

```
[WhatsApp +1555 2026-01-18 00:19 PST] hello
```

**ユーザーのタイムゾーン:**

```
[WhatsApp +1555 2026-01-18 00:19 CST] hello
```

**経過時間を有効化:**

```
[WhatsApp +1555 +30s 2026-01-18T05:19Z] follow-up
```

<div id="system-prompt-current-date-time">
  ## システムプロンプト: 現在日時
</div>

ユーザーのタイムゾーンが判明している場合、システムプロンプトには専用の
**Current Date &amp; Time** セクションが含まれ、その中にはプロンプトキャッシュを安定させるために
**タイムゾーンのみ**（時刻やそのフォーマットは含めない）を含めます。

```
Time zone: America/Chicago
```

エージェントが現在の時刻を取得する必要がある場合は、`session_status` ツールを使用してください。ステータス
カードにはタイムスタンプ行が含まれています。

<div id="system-event-lines-local-by-default">
  ## システムイベント行（デフォルトではローカル）
</div>

エージェントのコンテキストに挿入されるキューに入れられたシステムイベントには、メッセージエンベロープと同じタイムゾーン（デフォルト: ホストローカル）を使用して、タイムスタンプが先頭に付与されます。

```
System: [2026-01-12 12:19:17 PST] Model switched.
```

<div id="configure-user-timezone-format">
  ### ユーザーのタイムゾーンと時刻形式を設定する
</div>

```json5
{
  agents: {
    defaults: {
      userTimezone: "America/Chicago",
      timeFormat: "auto" // auto | 12 | 24
    }
  }
}
```

* `userTimezone` は、プロンプトコンテキスト用の**ユーザーのローカルタイムゾーン**を設定します。
* `timeFormat` は、プロンプト内の**12時間/24時間表示**を制御します。`auto` は OS の設定に従います。

<div id="time-format-detection-auto">
  ## 時刻形式の自動検出 (auto)
</div>

`timeFormat: "auto"` の場合、OpenClaw は OS の時刻表示設定 (macOS/Windows) を参照し、
ロケールに基づく書式にフォールバックします。検出された値は、
システムコールの繰り返しを避けるために **プロセスごとにキャッシュ** されます。

<div id="tool-payloads-connectors-raw-provider-time-normalized-fields">
  ## ツールペイロードとコネクター（生のプロバイダー時刻＋正規化済みフィールド）
</div>

チャネルツールは**プロバイダー固有のタイムスタンプ**を返し、一貫性のために正規化済みフィールドを追加します。

* `timestampMs`: エポックミリ秒（UTC）
* `timestampUtc`: ISO 8601 形式の UTC 文字列

プロバイダーからの生のフィールドもそのまま保持されるため、情報が失われることはありません。

* Slack: API から返されるエポック風の文字列
* Discord: UTC ISO タイムスタンプ
* Telegram/WhatsApp: プロバイダー固有の数値／ISO タイムスタンプ

ローカル時刻が必要な場合は、既知のタイムゾーン情報を用いて後段の処理で変換してください。

<div id="related-docs">
  ## 関連ドキュメント
</div>

* [システムプロンプト](/ja/concepts/system-prompt)
* [タイムゾーン](/ja/concepts/timezone)
* [メッセージ](/ja/concepts/messages)