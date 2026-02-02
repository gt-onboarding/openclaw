---
title: メニューバー
summary: "メニューバーのステータスロジックとユーザーに表示される内容"
read_when:
  - Mac のメニューバー UI やステータスロジックを調整しているとき
---

<div id="menu-bar-status-logic">
  # メニューバーのステータス表示ロジック
</div>

<div id="what-is-shown">
  ## 表示内容
</div>

- メニューバーのアイコンとメニューの最初のステータス行に、現在のエージェントの作業状態が表示されます。
- 作業中はヘルスステータスは非表示になり、すべてのセッションがアイドル状態になると再び表示されます。
- メニュー内の「Nodes」ブロックには、クライアント／プレゼンスエントリではなく、**デバイス**（`node.list` 経由でペアリングされたノード）のみが一覧表示されます。
- プロバイダーの使用状況スナップショットが利用可能な場合は、「Context」の下に「Usage」セクションが表示されます。

<div id="state-model">
  ## 状態モデル
</div>

- セッション: イベントは（実行ごとの）`runId` と、ペイロード内の `sessionKey` を伴って届きます。「main」セッションはキー `main` です。これが存在しない場合は、最後に更新されたセッションにフォールバックします。
- 優先順位: main が常に優先されます。main がアクティブであれば、その状態を即座に表示します。main がアイドル状態であれば、直近でアクティブだった non‑main セッションを表示します。アクティビティの途中で表示対象を切り替えることはせず、現在のセッションがアイドルになるか、main がアクティブになるタイミングでのみ切り替えます。
- アクティビティ種別:
  - `job`: 高水準のコマンド実行（`state: started|streaming|done|error`）。
  - `tool`: `phase: start|result` と `toolName`、`meta/args` を伴います。

<div id="iconstate-enum-swift">
  ## IconState enum (Swift)
</div>

- `idle`
- `workingMain(ActivityKind)`
- `workingOther(ActivityKind)`
- `overridden(ActivityKind)`（デバッグ用のオーバーライド）

<div id="activitykind-glyph">
  ### ActivityKind → グリフ
</div>

- `exec` → 💻
- `read` → 📄
- `write` → ✍️
- `edit` → 📝
- `attach` → 📎
- デフォルト → 🛠️

<div id="visual-mapping">
  ### 表示のマッピング
</div>

- `idle`: 通常状態の critter。
- `workingMain`: バッジにグリフ、全面的な着色、脚の「作業中」アニメーション。
- `workingOther`: バッジにグリフ、控えめな着色、`scurry` なし。
- `overridden`: 動作状態に関係なく、選択したグリフ／色を使用。

<div id="status-row-text-menu">
  ## ステータス行のテキスト（メニュー）
</div>

- 処理が実行中のとき: `<Session role> · <activity label>`
  - 例: `Main · exec: pnpm test`, `Other · read: apps/macos/Sources/OpenClaw/AppState.swift`
- アイドル状態のとき: ヘルスサマリーが表示されます。

<div id="event-ingestion">
  ## イベントの取り込み
</div>

- ソース: コントロールチャネルの `agent` イベント（`ControlChannel.handleAgentEvent`）。
- 解析されるフィールド:
  - 開始/停止を示す `data.state` を持つ `stream: "job"`。
  - `data.phase`、`name`、任意の `meta`/`args` を持つ `stream: "tool"`。
- ラベル:
  - `exec`: `args.command` の先頭行。
  - `read`/`write`: 短縮されたパス。
  - `edit`: パスと、`meta`/差分数から推論された変更種別。
  - フォールバック時: ツール名。

<div id="debug-override">
  ## デバッグ用オーバーライド
</div>

- Settings ▸ Debug ▸ 「Icon override」ピッカー:
  - `System (auto)`（デフォルト）
  - `Working: main`（ツール種別ごと）
  - `Working: other`（ツール種別ごと）
  - `Idle`
- `@AppStorage("iconOverride")` に保存され、`IconState.overridden` にマッピングされる。

<div id="testing-checklist">
  ## テストチェックリスト
</div>

- メイン セッションのジョブをトリガーする：アイコンが即座に切り替わり、ステータス行にメインのラベルが表示されることを確認すること。
- メイン セッションがアイドル状態のときに非メイン セッションのジョブをトリガーする：アイコン／ステータスが非メインを示し、ジョブが完了するまでその状態が変わらないことを確認すること。
- 別のセッションがアクティブな状態でメイン セッションを開始する：アイコンが即座にメインへ切り替わることを確認すること。
- ツールが連続してバースト実行される場合：バッジが点滅しないことを確認すること（ツール結果に TTL のグレース期間が効いていること）。
- すべてのセッションがアイドル状態になると、ヘルス行が再び表示されることを確認すること。