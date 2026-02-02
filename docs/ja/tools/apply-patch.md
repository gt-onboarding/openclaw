---
title: パッチの適用
summary: "apply_patch ツールで複数ファイルにパッチを適用する"
read_when:
  - 複数ファイルにわたって構造化されたファイル編集を行う必要がある場合
  - パッチベースの編集内容を記録したりデバッグしたりしたい場合
---

<div id="apply_patch-tool">
  # apply_patch tool
</div>

構造化されたパッチ形式を使ってファイルの変更を適用します。これは、単一の `edit` 呼び出しでは脆くなってしまうような、複数ファイルまたは複数ハンクの編集に最適です。

このツールは、1 つ以上のファイル操作を含む単一の `input` 文字列を受け取ります。

```
*** Begin Patch
*** Add File: path/to/file.txt
+line 1
+line 2
*** Update File: src/app.ts
@@
-old line
+new line
*** Delete File: obsolete.txt
*** End Patch
```


<div id="parameters">
  ## パラメーター
</div>

- `input` (必須): `*** Begin Patch` および `*** End Patch` を含むパッチ内容全体。

<div id="notes">
  ## 注意事項
</div>

- パスはワークスペースのルートからの相対パスとして解決されます。
- ファイル名を変更するには、`*** Update File:` ハンク内で `*** Move to:` を使用します。
- 必要に応じて、`*** End of File` が EOF のみを挿入する位置を示します。
- 実験的機能であり、デフォルトでは無効です。`tools.exec.applyPatch.enabled` で有効化します。
- OpenAI 専用（OpenAI Codex を含む）です。必要に応じて
  `tools.exec.applyPatch.allowModels` でモデル単位で制限できます。
- 設定は `tools.exec` 配下にのみあります。

<div id="example">
  ## 例
</div>

```json
{
  "tool": "apply_patch",
  "input": "*** Begin Patch\n*** Update File: src/index.ts\n@@\n-const foo = 1\n+const foo = 2\n*** End Patch"
}
```
