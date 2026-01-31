---
title: オンボード
summary: "`openclaw onboard`（対話型オンボーディングウィザード）の CLI リファレンス"
read_when:
  - Gateway、ワークスペース、認証、チャネル、スキルのセットアップをガイド付きで行いたいとき
---

<div id="openclaw-onboard">
  # `openclaw onboard`
</div>

対話型オンボーディングウィザード（ローカル／リモート Gateway のセットアップ用）。

関連:

* ウィザードガイド: [オンボーディング](/ja/start/onboarding)

<div id="examples">
  ## 使用例
</div>

```bash
openclaw onboard
openclaw onboard --flow quickstart
openclaw onboard --flow manual
openclaw onboard --mode remote --remote-url ws://gateway-host:18789
```

フローに関するメモ:

* `quickstart`: 最小限のプロンプトのみで Gateway トークンを自動生成します。
* `manual`: ポート / バインド / 認証をすべて対話的に設定します（`advanced` のエイリアス）。
* 最短で最初のチャットを始めるには: `openclaw dashboard`（Control UI。チャネルのセットアップは不要）。
