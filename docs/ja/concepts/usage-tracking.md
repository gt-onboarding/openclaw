---
title: Usage トラッキング
summary: "Usage トラッキングの表示と認証情報要件"
read_when:
  - プロバイダーの Usage やクォータの表示を組み込んでいるとき
  - Usage トラッキングの挙動や認証情報要件について説明する必要があるとき
---

<div id="usage-tracking">
  # 利用状況トラッキング
</div>

<div id="what-it-is">
  ## 概要
</div>

- プロバイダーの usage 用エンドポイントから、利用状況とクォータを直接取得します。
- コストの推定は行わず、プロバイダーが報告する利用ウィンドウだけを使用します。

<div id="where-it-shows-up">
  ## 表示される場所
</div>

- チャット内の `/status`：絵文字付きのステータスカードに、セッションのトークン数と推定コスト（API キー使用時のみ）を表示します。プロバイダーの使用状況は、利用可能な場合は**現在のモデルプロバイダー**について表示されます。
- チャット内の `/usage off|tokens|full`：レスポンスごとの使用状況フッターを表示します（OAuth の場合はトークン数のみ）。
- チャット内の `/usage cost`：OpenClaw のセッションログから集計したローカルのコストサマリーを表示します。
- CLI: `openclaw status --usage` は、プロバイダーごとの詳細な使用状況の内訳を出力します。
- CLI: `openclaw channels list` は、プロバイダーの設定情報と並べて同じ使用状況のスナップショットを出力します（スキップするには `--no-usage` を使用します）。
- macOS メニューバー：Context 配下の「Usage」セクション（利用可能な場合のみ）。

<div id="providers-credentials">
  ## プロバイダー + 資格情報
</div>

- **Anthropic (Claude)**: 認証プロファイル内の OAuth トークン。
- **GitHub Copilot**: 認証プロファイル内の OAuth トークン。
- **Gemini CLI**: 認証プロファイル内の OAuth トークン。
- **Antigravity**: 認証プロファイル内の OAuth トークン。
- **OpenAI Codex**: 認証プロファイル内の OAuth トークン（存在する場合は accountId を使用）。
- **MiniMax**: API キー（コーディングプラン用キー; `MINIMAX_CODE_PLAN_KEY` または `MINIMAX_API_KEY`）。5 時間のコーディングプランウィンドウを使用。
- **z.ai**: env/config/auth store 経由の API キー。

対応する OAuth/API 資格情報が存在しない場合、利用状況は非表示になります。