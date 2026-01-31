---
title: 投票
summary: "Gateway と CLI を使った投票送信"
read_when:
  - 投票機能を追加または変更するとき
  - CLI または Gateway 経由の投票送信をデバッグするとき
---

<div id="polls">
  # 投票
</div>

<div id="supported-channels">
  ## サポート対象チャネル
</div>

- WhatsApp（Web チャネル）
- Discord
- MS Teams（Adaptive Cards）

<div id="cli">
  ## CLI
</div>

```bash
# WhatsApp
openclaw message poll --target +15555550123 \
  --poll-question "Lunch today?" --poll-option "Yes" --poll-option "No" --poll-option "Maybe"
openclaw message poll --target 123456789@g.us \
  --poll-question "Meeting time?" --poll-option "10am" --poll-option "2pm" --poll-option "4pm" --poll-multi

# Discord
openclaw message poll --channel discord --target channel:123456789 \
  --poll-question "Snack?" --poll-option "Pizza" --poll-option "Sushi"
openclaw message poll --channel discord --target channel:123456789 \
  --poll-question "Plan?" --poll-option "A" --poll-option "B" --poll-duration-hours 48

# MS Teams
openclaw message poll --channel msteams --target conversation:19:abc@thread.tacv2 \
  --poll-question "Lunch?" --poll-option "Pizza" --poll-option "Sushi"
```

Options:

* `--channel`: `whatsapp`（デフォルト）、`discord`、または `msteams`
* `--poll-multi`: 複数の選択肢を選択可能にする
* `--poll-duration-hours`: Discord のみ有効（省略時は 24 時間がデフォルト）


<div id="gateway-rpc">
  ## Gateway RPC
</div>

メソッド: `poll`

パラメータ:

- `to` (string, 必須)
- `question` (string, 必須)
- `options` (string[], 必須)
- `maxSelections` (number, 省略可)
- `durationHours` (number, 省略可)
- `channel` (string, 省略可, デフォルト: `whatsapp`)
- `idempotencyKey` (string, 必須)

<div id="channel-differences">
  ## チャンネルごとの違い
</div>

- WhatsApp: 選択肢は 2～12 個、`maxSelections` は選択肢数の範囲内である必要があり、`durationHours` は無視されます。
- Discord: 選択肢は 2～10 個、`durationHours` は 1～768 時間の範囲に制限されます（デフォルト 24）。`maxSelections > 1` で複数選択が有効になりますが、Discord 側では厳密な選択数の固定はサポートされません。
- MS Teams: Adaptive Card による投票（OpenClaw 管理）。ネイティブな投票用 api は存在せず、`durationHours` は無視されます。

<div id="agent-tool-message">
  ## エージェントツール (Message)
</div>

`poll` アクションとともに `message` ツールを使用します（`to`、`pollQuestion`、`pollOption`、任意で `pollMulti`、`pollDurationHours`、`channel`）。

注意: Discord には「ちょうど N 個を選択」というモードはありません。`pollMulti` は複数選択に対応します。
Teams の投票は Adaptive Cards としてレンダリングされ、投票を `~/.openclaw/msteams-polls.json` に記録するために Gateway を稼働させ続けておく必要があります。