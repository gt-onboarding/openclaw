---
title: メモリ
summary: "`openclaw memory`（status/index/search）の CLI リファレンス"
read_when:
  - セマンティックメモリをインデックス化したり検索したいとき
  - メモリの利用状況やインデックス処理をデバッグしているとき
---

<div id="openclaw-memory">
  # `openclaw memory`
</div>

セマンティックメモリのインデックス作成と検索を管理します。
有効なメモリ用プラグインによって提供されます（デフォルトは `memory-core`。無効化するには `plugins.slots.memory = "none"` を設定します）。

関連:

* メモリの概念: [Memory](/ja/concepts/memory)
* プラグイン: [Plugins](/ja/plugins)

<div id="examples">
  ## 使用例
</div>

```bash
openclaw memory status
openclaw memory status --deep
openclaw memory status --deep --index
openclaw memory status --deep --index --verbose
openclaw memory index
openclaw memory index --verbose
openclaw memory search "release checklist"
openclaw memory status --agent main
openclaw memory index --agent main --verbose
```

<div id="options">
  ## オプション
</div>

共通:

* `--agent <id>`: 対象を単一のエージェントに限定する（デフォルト: 設定済みのすべてのエージェント群）。
* `--verbose`: プローブおよびインデックス作成中の詳細ログを出力する。

注意事項:

* `memory status --deep` はベクターおよび埋め込みの可用性をプローブする。
* `memory status --deep --index` はストアに未反映の変更がある場合に再インデックスを実行する。
* `memory index --verbose` はフェーズごとの詳細（プロバイダー、モデル、ソース、バッチ処理の状況）を出力する。
* `memory status` は `memorySearch.extraPaths` で設定された追加パスも含めて表示する。