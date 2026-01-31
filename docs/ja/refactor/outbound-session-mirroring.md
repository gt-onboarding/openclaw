---
title: アウトバウンド・セッションミラーリングのリファクタリング（Issue #1520）
description: アウトバウンド・セッションミラーリングのリファクタリングに関するメモ、決定事項、テスト、および未対応項目を記録・管理する。
---

<div id="outbound-session-mirroring-refactor-issue-1520">
  # アウトバウンドセッション・ミラーリングのリファクタリング（Issue #1520）
</div>

<div id="status">
  ## ステータス
</div>

- 対応中。
- アウトバウンドミラーリング向けに Core とプラグインチャネルのルーティングを更新済み。
- Gateway の send は、sessionKey が省略された場合に宛先セッションを自動的に決定するようになった。

<div id="context">
  ## コンテキスト
</div>

アウトバウンド送信は、ターゲットのチャネルセッションではなく、*現在の* エージェントセッション（ツール用セッションキー）にミラーされていました。インバウンドのルーティングはチャネル／ピアのセッションキーを使用するため、アウトバウンドレスポンスが誤ったセッションに記録され、初回コンタクト先にはセッションエントリが存在しないことがしばしばありました。

<div id="goals">
  ## 目標
</div>

- 送信メッセージを対象チャネルのセッションキーにミラーリングする。
- 送信時にセッションエントリが存在しない場合は作成する。
- スレッド／トピックのスコープを受信側のセッションキーと整合させて保つ。
- コアチャネルおよびバンドルされた拡張機能をカバーする。

<div id="implementation-summary">
  ## 実装サマリー
</div>

- 新しいアウトバウンドセッションルーティング用ヘルパー:
  - `src/infra/outbound/outbound-session.ts`
  - `resolveOutboundSessionRoute` は `buildAgentSessionKey`（dmScope + identityLinks）を使用してターゲットの sessionKey を構築する。
  - `ensureOutboundSessionEntry` は `recordSessionMetaFromInbound` を介して最小限の `MsgContext` を書き込む。
- `runMessageAction`（送信）はターゲットの sessionKey を導出し、ミラーリングのためにそれを `executeSendAction` に渡す。
- `message-tool` はもはや直接ミラーリングせず、現在の sessionKey から agentId を解決するだけにする。
- プラグインの送信パスは、導出された sessionKey を使用して `appendAssistantMessageToSessionTranscript` 経由でミラーリングする。
- Gateway による送信処理は、セッションキーが指定されていない場合（デフォルトエージェント）にターゲットの sessionKey を導出し、セッションエントリを確保する。

<div id="threadtopic-handling">
  ## スレッド／トピック処理
</div>

- Slack: replyTo/threadId -> `resolveThreadSessionKeys`（サフィックスを使用）。
- Discord: threadId/replyTo -> `resolveThreadSessionKeys` に `useSuffix=false` を指定して受信側と一致させる（スレッドチャンネル ID がすでにセッションのスコープとして機能している）。
- Telegram: トピック ID は `buildTelegramGroupPeerId` を通じて `chatId:topic:<id>` にマップする。

<div id="extensions-covered">
  ## 対応拡張機能
</div>

- Matrix、MS Teams、Mattermost、BlueBubbles、Nextcloud Talk、Zalo、Zalo Personal、Nostr、Tlon。
- 補足:
  - Mattermost のターゲットは、DM セッションキーのルーティングのために `@` を削除します。
  - Zalo Personal は、1:1 ターゲットに DM peer kind を使用します（`group:` が存在する場合にのみグループとして扱います）。
  - BlueBubbles のグループターゲットは、受信セッションキーに一致させるために `chat_*` プレフィックスを削除します。
  - Slack の自動スレッドミラーリングは、チャネル ID を大文字小文字を区別せずに照合します。
  - Gateway send は、ミラーリング前に指定されたセッションキーを小文字に変換します。

<div id="decisions">
  ## 決定事項
</div>

- **Gateway 送信セッション導出**: `sessionKey` が指定されている場合はそれを使用する。省略されている場合は、ターゲット + デフォルトエージェントから `sessionKey` を導出し、そのセッションにミラーリングする。
- **セッションエントリの作成**: 常に `recordSessionMetaFromInbound` を使用し、`Provider/From/To/ChatType/AccountId/Originating*` をインバウンドの形式に合わせる。
- **ターゲットの正規化**: アウトバウンドルーティングでは、利用可能な場合は解決済みターゲット（`resolveChannelTarget` 実行後）を使用する。
- **セッションキーの大文字小文字**: セッションキーは書き込み時およびマイグレーション中に小文字へ正規化する。

<div id="tests-addedupdated">
  ## 追加／更新されたテスト
</div>

- `src/infra/outbound/outbound-session.test.ts`
  - Slack スレッド用セッションキー。
  - Telegram トピック用セッションキー。
  - Discord との dmScope identityLinks。
- `src/agents/tools/message-tool.test.ts`
  - セッションキーから agentId を導出する（sessionKey が渡されない場合）。
- `src/gateway/server-methods/send.test.ts`
  - セッションキーが省略された場合にセッションキーを導出し、セッションエントリを作成する。

<div id="open-items-follow-ups">
  ## 未完了項目 / フォローアップ
</div>

- Voice-call プラグインはカスタムの `voice:<phone>` セッションキーを使用している。送信側マッピングはここでは標準化されていないため、message-tool で音声通話の送信をサポートする必要がある場合は、明示的なマッピングを追加すること。
- 同梱されているセット以外で、外部プラグインが非標準的な `From/To` 形式を使用していないか確認すること。

<div id="files-touched">
  ## 変更対象ファイル
</div>

- `src/infra/outbound/outbound-session.ts`
- `src/infra/outbound/outbound-send-service.ts`
- `src/infra/outbound/message-action-runner.ts`
- `src/agents/tools/message-tool.ts`
- `src/gateway/server-methods/send.ts`
- テスト:
  - `src/infra/outbound/outbound-session.test.ts`
  - `src/agents/tools/message-tool.test.ts`
  - `src/gateway/server-methods/send.test.ts`