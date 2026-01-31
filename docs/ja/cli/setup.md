---
title: セットアップ
summary: "`openclaw setup` の CLI リファレンス（設定とワークスペースの初期化）"
read_when:
  - 完全なオンボーディングウィザードを使わずに初回セットアップを行う場合
  - デフォルトのワークスペースのパスを設定したい場合
---

<div id="openclaw-setup">
  # `openclaw setup`
</div>

`~/.openclaw/openclaw.json` とエージェント用ワークスペースを初期化します。

関連:

* はじめに: [Getting started](/ja/start/getting-started)
* セットアップウィザード: [Onboarding](/ja/start/onboarding)

<div id="examples">
  ## 使用例
</div>

```bash
openclaw setup
openclaw setup --workspace ~/.openclaw/workspace
```

セットアップからウィザードを実行するには次のようにします:

```bash
openclaw setup --wizard
```
