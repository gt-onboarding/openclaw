---
title: セッションツール
summary: "セッションの一覧表示、履歴取得、セッション間メッセージ送信を行うエージェント用セッションツール"
read_when:
  - セッションツールを追加または変更するとき
---

<div id="session-tools">
  # セッションツール
</div>

目的: エージェントがセッションを一覧表示し、履歴を取得し、別のセッションへ送信できるようにする、小規模で誤用しにくいツールセットを提供すること。

<div id="tool-names">
  ## ツール名
</div>

- `sessions_list`
- `sessions_history`
- `sessions_send`
- `sessions_spawn`

<div id="key-model">
  ## キーのモデル
</div>

- メインのダイレクトチャット用バケットは、常にリテラルキー `"main"`（現在のエージェントのメインキーに解決される）です。
- グループチャットは `agent:<agentId>:<channel>:group:<id>` または `agent:<agentId>:<channel>:channel:<id>` を使用します（完全なキーを渡します）。
- Cron ジョブは `cron:<job.id>` を使用します。
- フックは明示的に設定されていない限り `hook:<uuid>` を使用します。
- ノードのセッションは明示的に設定されていない限り `node-<nodeId>` を使用します。

`global` と `unknown` は予約済みの値であり、決して一覧には含まれません。`session.scope = "global"` の場合、すべてのツールに対してそれを `main` のエイリアスとして扱うため、呼び出し側が `global` を目にすることはありません。

<div id="sessions_list">
  ## sessions_list
</div>

セッションを行の配列として一覧表示します。

Parameters:

- `kinds?: string[]` フィルタ: `"main" | "group" | "cron" | "hook" | "node" | "other"` のいずれか
- `limit?: number` 取得する最大行数（デフォルト: サーバーのデフォルト。例: 最大 200 行などに制限）
- `activeMinutes?: number` 直近 N 分以内に更新されたセッションのみ
- `messageLimit?: number` 0 = メッセージなし（デフォルト 0）、>0 = 直近 N 件のメッセージを含める

Behavior:

- `messageLimit > 0` の場合、セッションごとに `chat.history` を取得し、直近 N 件のメッセージを含めます。
- ツールの結果は一覧出力から除外されます。ツールメッセージには `sessions_history` を使用してください。
- **サンドボックス化された** エージェントセッションで実行している場合、セッションツールのデフォルトの可視範囲は **spawned-only visibility** になります（下記参照）。

Row shape (JSON):

- `key`: セッションキー（string）
- `kind`: `main | group | cron | hook | node | other`
- `channel`: `whatsapp | telegram | discord | signal | imessage | webchat | internal | unknown`
- `displayName`（存在する場合のグループ表示ラベル）
- `updatedAt`（ミリ秒）
- `sessionId`
- `model`, `contextTokens`, `totalTokens`
- `thinkingLevel`, `verboseLevel`, `systemSent`, `abortedLastRun`
- `sendPolicy`（セッションで上書きされている場合）
- `lastChannel`, `lastTo`
- `deliveryContext`（利用可能な場合の正規化された `{ channel, to, accountId }`）
- `transcriptPath`（ストアディレクトリ + sessionId から導出された、可能な範囲での推定パス）
- `messages?`（`messageLimit > 0` の場合のみ）

<div id="sessions_history">
  ## sessions_history
</div>

1つのセッションの会話ログ（トランスクリプト）を取得します。

Parameters:

- `sessionKey` (必須; セッションキーまたは `sessions_list` の `sessionId` を受け付けます)
- `limit?: number` メッセージの最大数（サーバー側でさらに上限が適用されます）
- `includeTools?: boolean` (デフォルト false)

Behavior:

- `includeTools=false` の場合、`role: "toolResult"` のメッセージを除外します。
- 生のトランスクリプト形式でメッセージ配列を返します。
- `sessionId` が指定された場合、OpenClaw はそれに対応するセッションキーへ解決します（存在しない id の場合はエラー）。

<div id="sessions_send">
  ## sessions_send
</div>

別のセッションにメッセージを送信します。

Parameters:

- `sessionKey` (必須; セッションキー、または `sessions_list` の `sessionId` を指定)
- `message` (必須)
- `timeoutSeconds?: number` (デフォルト > 0; 0 = fire-and-forget)

Behavior:

- `timeoutSeconds = 0`: キューに積んで `{ runId, status: "accepted" }` を返します。
- `timeoutSeconds > 0`: 最大 N 秒間、完了を待機し、その後 `{ runId, status: "ok", reply }` を返します。
- 待機がタイムアウトした場合: `{ runId, status: "timeout", error }`。実行は継続します。後で `sessions_history` を呼び出してください。
- 実行が失敗した場合: `{ runId, status: "error", error }`。
- プライマリ実行完了後に配信アナウンス用の実行を行いますが、これはベストエフォートであり、`status: "ok"` はアナウンスが確実に配信されたことを保証しません。
- Gateway の `agent.wait`（サーバー側）を使って待機するため、再接続しても待機が失われません。
- プライマリ実行に対しては、エージェント間メッセージコンテキストが注入されます。
- プライマリ実行完了後、OpenClaw は **返信ループ（reply-back loop）** を実行します:
  - 2 ターン目以降は、リクエスタ側エージェントとターゲットエージェントの間で交互に実行されます。
  - ピンポンを止めるには、正確に `REPLY_SKIP` と返信します。
  - 最大ターン数は `session.agentToAgent.maxPingPongTurns`（0–5、デフォルト 5）です。
- ループ終了後、OpenClaw は **エージェント間アナウンスステップ**（ターゲットエージェントのみ）を実行します:
  - 反応せずに沈黙を保つには、正確に `ANNOUNCE_SKIP` と返信します。
  - それ以外の返信はターゲットチャネルへ送信されます。
  - アナウンスステップには、元のリクエスト + 1 ターン目の返信 + 直近のピンポン返信が含まれます。

<div id="channel-field">
  ## Channel フィールド
</div>

- グループの場合、`channel` はセッションエントリに記録されているチャネルです。
- ダイレクトチャットの場合、`channel` には `lastChannel` の値がマッピングされます。
- cron/hook/node の場合、`channel` は `internal` です。
- 指定されていない場合、`channel` は `unknown` になります。

<div id="security-send-policy">
  ## セキュリティ / 送信ポリシー
</div>

チャネル／チャット種別単位のポリシーベースのブロック（セッション ID 単位ではない）。

```json
{
  "session": {
    "sendPolicy": {
      "rules": [
        {
          "match": { "channel": "discord", "chatType": "group" },
          "action": "deny"
        }
      ],
      "default": "allow"
    }
  }
}
```

セッションエントリごとのランタイムオーバーライド:

* `sendPolicy: "allow" | "deny"`（未設定の場合は設定を継承）
* `sessions.patch` または所有者専用の `/send on|off|inherit`（単独メッセージとして）で設定可能。

強制適用ポイント:

* `chat.send` / `agent`（Gateway）
* 自動返信の配信ロジック


<div id="sessions_spawn">
  ## sessions_spawn
</div>

分離されたセッション内でサブエージェントの実行を起動し、その結果をリクエスト元のチャットチャネルに通知します。

Parameters:

- `task` (必須)
- `label?` (任意; ログや UI 用)
- `agentId?` (任意; 許可されている場合は別のエージェント ID の下で起動)
- `model?` (任意; サブエージェントのモデルを上書き; 不正な値はエラー)
- `runTimeoutSeconds?` (デフォルト 0; 設定すると、サブエージェントの実行を N 秒後に中断)
- `cleanup?` (`delete|keep`, デフォルト `keep`)

Allowlist:

- `agents.list[].subagents.allowAgents`: `agentId` 経由で許可されるエージェント ID のリスト（任意のエージェントを許可するには `["*"]`）。デフォルト: リクエスト元エージェントのみ。

Discovery:

- `sessions_spawn` で許可されているエージェント ID を確認するには `agents_list` を使用します。

Behavior:

- `deliver: false` で新しい `agent:<agentId>:subagent:<uuid>` セッションを開始します。
- サブエージェントはデフォルトで **セッションツールを除く** 全ツールセットを使用します（`tools.subagents.tools` で設定可能）。
- サブエージェントは `sessions_spawn` を呼び出すことはできません（サブエージェント → サブエージェントの生成は禁止）。
- 常にノンブロッキング: 即座に `{ status: "accepted", runId, childSessionKey }` を返します。
- 完了後、OpenClaw はサブエージェントの**アナウンスステップ**を実行し、その結果をリクエスト元のチャットチャネルに投稿します。
- アナウンスステップ中に `ANNOUNCE_SKIP` と**完全一致で**返信すると、通知を行いません。
- アナウンス返信は `Status`/`Result`/`Notes` に正規化されます。`Status` は実行時の結果に基づいて決まり（モデルテキストからではありません）。
- サブエージェントのセッションは `agents.defaults.subagents.archiveAfterMinutes` 経過後に自動アーカイブされます（デフォルト: 60）。
- アナウンス返信には統計行（実行時間、トークン数、sessionKey/sessionId、トランスクリプトパス、および任意のコスト）が含まれます。

<div id="sandbox-session-visibility">
  ## サンドボックス・セッションの可視性
</div>

サンドボックス化されたセッションはセッションツールを利用できますが、デフォルトでは `sessions_spawn` で自分自身が生成したセッションしか参照できません。

設定:

```json5
{
  agents: {
    defaults: {
      sandbox: {
        // デフォルト: "spawned"
        sessionToolsVisibility: "spawned" // or "all"
      }
    }
  }
}
```
