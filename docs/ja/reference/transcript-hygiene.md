---
title: トランスクリプトの健全性管理
summary: "リファレンス: プロバイダー固有のトランスクリプトのサニタイズと修復ルール"
read_when:
  - トランスクリプトの構造に起因するプロバイダーからのリクエスト拒否をデバッグしているとき
  - トランスクリプトのサニタイズ処理やツール呼び出しの修復ロジックを変更しているとき
  - プロバイダー間でのツール呼び出し ID の不一致を調査しているとき
---

<div id="transcript-hygiene-provider-fixups">
  # Transcript Hygiene (Provider Fixups)
</div>

このドキュメントでは、実行開始前（モデルコンテキストを構築する前）にトランスクリプトへ適用される、**プロバイダー固有の修正** について説明します。これらは厳密なプロバイダー要件を満たすために行う **インメモリでの調整** であり、ディスク上に保存された JSONL トランスクリプトを書き換えるものではありません。

対象範囲には次が含まれます:

* ツール呼び出し ID のサニタイズ
* ツール結果ペアリングの修復
* ターンの検証 / 並び順の調整
* 思考シグネチャのクリーンアップ
* 画像ペイロードのサニタイズ

トランスクリプトのストレージに関する詳細が必要な場合は、次を参照してください:

* [/reference/session-management-compaction](/ja/reference/session-management-compaction)

***

<div id="where-this-runs">
  ## これが実行される場所
</div>

すべてのトランスクリプト衛生処理は、埋め込みランナー内で一元管理されています：

* ポリシーの選択：`src/agents/transcript-policy.ts`
* サニタイズ／修復の適用：`src/agents/pi-embedded-runner/google.ts` 内の `sanitizeSessionHistory`

このポリシーは、`provider`、`modelApi`、`modelId` を使って、どの処理を適用するかを決定します。

***

<div id="global-rule-image-sanitization">
  ## グローバルルール: 画像のサニタイズ
</div>

画像ペイロードは常にサニタイズされ、サイズ制限超過（サイズオーバーした base64 画像のダウンスケール／再圧縮）が原因となるプロバイダー側での拒否を防ぎます。

実装:

* `src/agents/pi-embedded-helpers/images.ts` の `sanitizeSessionMessagesImages`
* `src/agents/tool-images.ts` の `sanitizeContentBlocksImages`

***

<div id="provider-matrix-current-behavior">
  ## プロバイダー・マトリクス（現在の動作）
</div>

**OpenAI / OpenAI Codex**

* 画像のサニタイズのみ。
* OpenAI Responses/Codex へのモデル切り替え時に、孤立した reasoning シグネチャ（後続の content ブロックを持たない単独の reasoning アイテム）を破棄。
* tool call id のサニタイズなし。
* tool result のペアリング修復なし。
* ターン検証や並べ替えなし。
* 合成（synthetic）tool result なし。
* thought signature の削除なし。

**Google (Generative AI / Gemini CLI / Antigravity)**

* tool call id のサニタイズ: 厳密な英数字のみ。
* tool result のペアリング修復および合成（synthetic）tool result を実施。
* ターン検証（Gemini スタイルのターン交互制約）。
* Google 向けターン順序の修正（履歴が assistant で始まる場合、先頭に短い user ブートストラップを付加）。
* Antigravity Claude: thinking シグネチャを正規化し、署名のない thinking ブロックを破棄。

**Anthropic / Minimax (Anthropic-compatible)**

* tool result のペアリング修復および合成（synthetic）tool result を実施。
* ターン検証（厳密な交互制約を満たすため、連続する user ターンを結合）。

**Mistral（model-id ベース検出を含む）**

* tool call id のサニタイズ: strict9（長さ 9 の英数字のみ）。

**OpenRouter Gemini**

* thought signature のクリーンアップ: base64 でない `thought_signature` 値を削除（base64 は保持）。

**その他すべて**

* 画像のサニタイズのみ。

***

<div id="historical-behavior-pre-2026122">
  ## 過去の挙動（2026.1.22 以前）
</div>

2026.1.22 リリース以前は、OpenClaw は複数レイヤーの transcript hygiene（対話履歴クリーンアップ処理）を適用していました:

* **transcript-sanitize extension** がすべてのコンテキスト構築時に実行され、次のことを行えました:
  * ツール呼び出しと結果のペアリングを修復。
  * ツール呼び出し ID をサニタイズ（`_` / `-` を保持する非厳格モードも含む）。
* ランナーもプロバイダー固有のサニタイズを実行しており、処理が重複していました。
* さらに、プロバイダー・ポリシーの外側でも追加の変更が行われていました。具体的には:
  * 永続化前に assistant テキストから `<final>` タグを削除。
  * 空の assistant のエラーターンを破棄。
  * ツール呼び出し後の assistant コンテンツをトリミング。

この複雑さによりプロバイダー間でリグレッションが発生しました（特に `openai-responses`
の `call_id|fc_id` ペアリング）。2026.1.22 のクリーンアップでは extension を削除し、ロジックをランナーに集中させ、OpenAI に対しては画像サニタイズ以外には一切手を加えない **no-touch** ポリシーとしました。