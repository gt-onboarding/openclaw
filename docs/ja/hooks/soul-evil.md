---
title: ソウル・イービル
summary: "SOUL Evil フック（SOUL.md を SOUL_EVIL.md に差し替える）"
read_when:
  - SOUL Evil フックを有効化または調整したいとき
  - パージウィンドウやランダム確率でのペルソナの切り替えを行いたいとき
---

<div id="soul-evil-hook">
  # SOUL Evil Hook
</div>

SOUL Evil フックは、パージウィンドウ中、またはランダムなタイミングで、**注入された** `SOUL.md` の内容を `SOUL_EVIL.md` の内容と入れ替えます。ディスク上のファイルは**一切**変更しません。

<div id="how-it-works">
  ## 動作の仕組み
</div>

`agent:bootstrap` が実行されると、システムプロンプトが組み立てられる前に、
フックによってメモリ上の `SOUL.md` の内容を差し替えることができます。
`SOUL_EVIL.md` が存在しないか空の場合、OpenClaw は警告をログに出力し、
通常の `SOUL.md` をそのまま使用します。

サブエージェントの実行では、ブートストラップ用ファイルに `SOUL.md` が
含まれないため、このフックはサブエージェントには影響しません。

<div id="enable">
  ## 有効化
</div>

```bash
openclaw hooks enable soul-evil
```

次に、以下のように設定します：

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "soul-evil": {
          "enabled": true,
          "file": "SOUL_EVIL.md",
          "chance": 0.1,
          "purge": { "at": "21:00", "duration": "15m" }
        }
      }
    }
  }
}
```

エージェントのワークスペースのルート（`SOUL.md` の隣）に `SOUL_EVIL.md` を作成します。

<div id="options">
  ## オプション
</div>

* `file` (string): 代替 SOUL ファイル名（デフォルト: `SOUL_EVIL.md`）
* `chance` (number 0–1): 実行ごとに `SOUL_EVIL.md` を使用する確率（0〜1 の数値）
* `purge.at` (HH:mm): 毎日のパージ開始時刻（24時間表記）
* `purge.duration` (duration): 時間枠の長さ（例: `30s`, `10m`, `1h`）

**優先順位:** パージ時間枠が `chance` より優先されます。

**タイムゾーン:** 設定されている場合は `agents.defaults.userTimezone` を使用し、未設定の場合はホストのタイムゾーンを使用します。

<div id="notes">
  ## 注記
</div>

* ディスク上のファイルは一切書き込まれず、変更も行われません。
* `SOUL.md` がブートストラップリストに含まれていない場合、このフックは何も行いません。

<div id="see-also">
  ## 関連情報
</div>

* [Hooks](/ja/hooks)