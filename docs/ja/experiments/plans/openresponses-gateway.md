---
title: Openresponses Gateway
summary: "計画：OpenResponses の /v1/responses エンドポイントを追加し、chat completions をクリーンに非推奨化する"
owner: "openclaw"
status: "draft"
last_updated: "2026-01-19"
---

<div id="openresponses-gateway-integration-plan">
  # OpenResponses Gateway 統合計画
</div>

<div id="context">
  ## コンテキスト
</div>

OpenClaw Gateway は現在、最小限の OpenAI 互換 Chat Completions エンドポイントを
`/v1/chat/completions`（[OpenAI Chat Completions](/ja/gateway/openai-http-api) を参照）で公開しています。

Open Responses は、OpenAI Responses API に基づくオープンな推論標準仕様です。エージェント指向のワークフロー向けに設計されており、アイテムベースの入力とセマンティック・ストリーミングイベントを使用します。OpenResponses の仕様では `/v1/responses` が定義されており、`/v1/chat/completions` ではありません。

<div id="goals">
  ## 目標
</div>

* OpenResponses のセマンティクスに準拠した `/v1/responses` エンドポイントを追加する。
* Chat Completions を、無効化しやすく最終的には削除可能な互換レイヤーとして維持する。
* 分離された再利用可能なスキーマを用いて、検証とパースを標準化する。

<div id="non-goals">
  ## 非目標
</div>

* 初回実装で OpenResponses の機能を完全に同等にすること（画像、ファイル、ホストされたツール）。
* 内部のエージェント実行ロジックやツールオーケストレーションを置き換えること。
* 第1フェーズの間に、既存の `/v1/chat/completions` の動作を変更しないこと。

<div id="research-summary">
  ## 調査サマリー
</div>

出典: OpenResponses OpenAPI、OpenResponses 仕様サイト、Hugging Face のブログ記事。

抽出した主なポイント:

* `POST /v1/responses` は、`CreateResponseBody` のフィールドとして `model`、`input`（文字列または
  `ItemParam[]`）、`instructions`、`tools`、`tool_choice`、`stream`、`max_output_tokens`、`max_tool_calls` を受け付ける。
* `ItemParam` は次の判別共用体型となっている:
  * `system`、`developer`、`user`、`assistant` というロールを持つ `message` アイテム
  * `function_call` および `function_call_output`
  * `reasoning`
  * `item_reference`
* 成功したレスポンスは、`object: "response"`、`status`、および `output` アイテムを持つ `ResponseResource` を返す。
* ストリーミングでは、次のようなセマンティックイベントを使用する:
  * `response.created`、`response.in_progress`、`response.completed`、`response.failed`
  * `response.output_item.added`、`response.output_item.done`
  * `response.content_part.added`、`response.content_part.done`
  * `response.output_text.delta`、`response.output_text.done`
* 仕様上の要件:
  * `Content-Type: text/event-stream`
  * `event:` は JSON の `type` フィールドと一致していなければならない
  * 終端イベントはリテラルの `[DONE]` でなければならない
* `reasoning` アイテムには `content`、`encrypted_content`、`summary` が含まれる場合がある。
* Hugging Face の例では、リクエストに `OpenResponses-Version: latest`（任意ヘッダー）が含まれている。

<div id="proposed-architecture">
  ## 提案アーキテクチャ
</div>

* `src/gateway/open-responses.schema.ts` を追加し、Zod スキーマのみを含める（Gateway の import は行わない）。
* `/v1/responses` 用に `src/gateway/openresponses-http.ts`（または `open-responses-http.ts`）を追加する。
* レガシー互換性アダプタとして `src/gateway/openai-http.ts` はそのまま保持する。
* 設定項目 `gateway.http.endpoints.responses.enabled` を追加する（デフォルトは `false`）。
* `gateway.http.endpoints.chatCompletions.enabled` は独立させたままにし、両方のエンドポイントを
  個別に有効／無効を切り替えられるようにする。
* Chat Completions が有効化されている場合、レガシーであることを示す起動時の警告を出力する。

<div id="deprecation-path-for-chat-completions">
  ## Chat Completions の廃止までの移行パス
</div>

* 厳密なモジュール境界を維持し、responses と chat completions の間でスキーマ型を共有しない。
* 設定で Chat Completions をオプトイン方式にし、コードの変更なしで無効化できるようにする。
* `/v1/responses` が安定したら、ドキュメントを更新し、Chat Completions をレガシー機能として明記する。
* 将来的なオプションとして、削除パスを簡素化するために Chat Completions のリクエストを Responses ハンドラーにマッピングする。

<div id="phase-1-support-subset">
  ## フェーズ1でサポートするサブセット
</div>

* `input` を文字列、またはメッセージロールと `function_call_output` を含む `ItemParam[]` として受け付ける。
* system と developer メッセージを抽出して `extraSystemPrompt` に格納する。
* エージェント実行では、直近の `user` または `function_call_output` を現在のメッセージとして使用する。
* サポートされていないコンテンツパーツ（画像／ファイル）は `invalid_request_error` で拒否する。
* `output_text` コンテンツを持つ単一の assistant メッセージを返す。
* トークン計測が実装されるまでは、値がすべて 0 の `usage` を返す。

<div id="validation-strategy-no-sdk">
  ## 検証戦略（SDK なし）
</div>

* 次のサポート対象サブセット向けに Zod スキーマを実装する:
  * `CreateResponseBody`
  * `ItemParam` + メッセージコンテンツパートの union 型
  * `ResponseResource`
  * Gateway が使用するストリーミングイベントの構造
* スキーマは単一の独立したモジュールにまとめておき、乖離を防ぎつつ将来のコード生成を可能にする。

<div id="streaming-implementation-phase-1">
  ## ストリーミング実装（フェーズ 1）
</div>

* `event:` と `data:` の両方を含む SSE 行を使用する。
* 必須シーケンス（MVP としての最低限の要件）:
  * `response.created`
  * `response.output_item.added`
  * `response.content_part.added`
  * `response.output_text.delta`（必要に応じて繰り返し）
  * `response.output_text.done`
  * `response.content_part.done`
  * `response.completed`
  * `[DONE]`

<div id="tests-and-verification-plan">
  ## テストおよび検証計画
</div>

* `/v1/responses` に対する e2e テストカバレッジを追加する:
  * 認証が必須であること
  * ストリーミングしないレスポンスの構造
  * ストリームイベントの順序と `[DONE]`
  * ヘッダーおよび `user` を用いたセッションルーティング
* `src/gateway/openai-http.e2e.test.ts` は変更しない。
* 手動テスト: `stream: true` を指定して `/v1/responses` に対して curl を実行し、イベントの順序と終端の `[DONE]` を検証する。

<div id="doc-updates-follow-up">
  ## ドキュメントの更新（フォローアップ）
</div>

* `/v1/responses` の利用方法とサンプル例を示す新しいドキュメントページを追加する。
* `/gateway/openai-http-api` を、レガシーであることの注記と `/v1/responses` への誘導を含める形で更新する。