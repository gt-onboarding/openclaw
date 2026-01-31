---
title: セッション
summary: "チャット用セッションの管理ルール、キー、永続化"
read_when:
  - セッションの取り扱いや保存方式を変更するとき
---

<div id="session-management">
  # セッション管理
</div>

OpenClaw は、**1 エージェントあたり 1 つのダイレクトチャットセッション**を主要なセッションとして扱います。ダイレクトチャットは `agent:<agentId>:<mainKey>`（デフォルトは `main`）に集約される一方、グループ/チャンネルチャットはそれぞれ独自のキーを持ちます。`session.mainKey` が有効になります。

**ダイレクトメッセージ**のグルーピング方法を制御するには `session.dmScope` を使用します:

* `main`（デフォルト）: すべての DM が継続性のためにメインセッションを共有します。
* `per-peer`: チャンネルをまたいで送信者 ID ごとに分離します。
* `per-channel-peer`: チャンネル + 送信者ごとに分離します（マルチユーザー受信箱向けに推奨）。
* `per-account-channel-peer`: アカウント + チャンネル + 送信者ごとに分離します（マルチアカウント受信箱向けに推奨）。

`session.identityLinks` を使用して、プロバイダーのプレフィックスが付いたピア ID を正規のアイデンティティにマッピングすることで、`per-peer`、`per-channel-peer`、`per-account-channel-peer` を使用している場合でも、同一人物であればチャンネルをまたいで同じ DM セッションを共有できるようにします。

<div id="gateway-is-the-source-of-truth">
  ## Gateway が唯一の信頼できる情報源
</div>

すべてのセッション状態は **Gateway が管理** します（「マスター」OpenClaw）。UI クライアント（macOS アプリ、WebChat など）は、ローカルファイルを読み取るのではなく、セッション一覧とトークン数を取得するために Gateway に問い合わせる必要があります。

* **remote モード**では、あなたが扱うべきセッションストアは Mac 上ではなく、リモートの Gateway ホスト上に存在します。
* UI に表示されるトークン数は、Gateway のストアフィールド（`inputTokens`、`outputTokens`、`totalTokens`、`contextTokens`）から取得されます。クライアントが JSONL 形式のトランスクリプトをパースして合計値を「修正」することはありません。

<div id="where-state-lives">
  ## 状態の保存場所
</div>

* **Gateway ホスト**上:
  * ストア用ファイル: `~/.openclaw/agents/<agentId>/sessions/sessions.json`（エージェントごと）。
* トランスクリプト: `~/.openclaw/agents/<agentId>/sessions/<SessionId>.jsonl`（Telegram のトピックセッションは `.../<SessionId>-topic-<threadId>.jsonl` を使用）。
* ストアは `sessionKey -> { sessionId, updatedAt, ... }` のマップです。エントリを削除しても安全で、必要に応じて再作成されます。
* グループエントリには、UI 上でセッションにラベルを付けるために `displayName`、`channel`、`subject`、`room`、`space` が含まれる場合があります。
* セッションエントリには `origin` メタデータ（ラベル + ルーティングヒント）が含まれており、UI がセッションの起点を説明できるようになっています。
* OpenClaw は旧来の Pi/Tau セッションフォルダを**読み取りません**。

<div id="session-pruning">
  ## セッションの剪定
</div>

OpenClaw はデフォルトで、LLM 呼び出しの直前にメモリ上のコンテキストから**古いツールの結果**を削除します。
これは JSONL の履歴を書き換えるものでは**ありません**。[/concepts/session-pruning](/ja/concepts/session-pruning) を参照してください。

<div id="pre-compaction-memory-flush">
  ## 圧縮前のメモリフラッシュ
</div>

セッションが自動圧縮に近づいたとき、OpenClaw はモデルに対して永続的なノートをディスクに書き出すようリマインドする、**サイレントメモリフラッシュ**のターンを実行できます。これは、ワークスペースが書き込み可能な場合にのみ実行されます。[Memory](/ja/concepts/memory) および
[Compaction](/ja/concepts/compaction) を参照してください。

<div id="mapping-transports-session-keys">
  ## トランスポート → セッションキーのマッピング
</div>

* ダイレクトチャットは `session.dmScope` に従います（デフォルトは `main`）。
  * `main`: `agent:<agentId>:<mainKey>`（デバイス／チャネル間での継続性を提供）。
    * 複数の電話番号やチャネルを同一の agent main key にマッピングできます。その場合、それらは 1 つの会話へのトランスポートとして機能します。
  * `per-peer`: `agent:<agentId>:dm:<peerId>`。
  * `per-channel-peer`: `agent:<agentId>:<channel>:dm:<peerId>`。
  * `per-account-channel-peer`: `agent:<agentId>:<channel>:<accountId>:dm:<peerId>`（accountId のデフォルトは `default`）。
  * `session.identityLinks` がプロバイダー接頭辞付きの peer id（例: `telegram:123`）と一致する場合、正規キーが `<peerId>` を置き換えるため、同一人物はチャネルをまたいで 1 つのセッションを共有します。
* グループチャットは状態を分離します: `agent:<agentId>:<channel>:group:<id>`（ルーム／チャネルは `agent:<agentId>:<channel>:channel:<id>` を使用）。
  * Telegram のフォーラムトピックは、分離のためにグループ ID に `:topic:<threadId>` を付加します。
  * レガシーな `group:<id>` キーは、移行のために現在も認識されます。
* 受信コンテキストでは引き続き `group:<id>` が使われる場合があります。この場合、チャネルは `Provider` から推測され、正規形の `agent:<agentId>:<channel>:group:<id>` に正規化されます。
* その他のソース:
  * Cron ジョブ: `cron:<job.id>`
  * Webhook: `hook:<uuid>`（フック側で明示的に設定されていない限り）
  * ノード実行: `node-<nodeId>`

<div id="lifecycle">
  ## ライフサイクル
</div>

* リセットポリシー: セッションは有効期限切れになるまで再利用され、有効期限は次の受信メッセージが到着したタイミングで評価されます。
* 日次リセット: デフォルトは **Gateway ホストのローカル時間で午前 4:00** です。最後の更新時刻が直近の日次リセット時刻より前であれば、そのセッションは古い (stale) と見なされます。
* アイドルリセット (オプション): `idleMinutes` はスライディングウィンドウ形式のアイドル時間を追加します。日次リセットとアイドルリセットの両方が設定されている場合、**先に有効期限を迎えた方** が新しいセッションの開始を強制します。
* レガシーなアイドル専用モード: `session.reset` / `resetByType` 設定なしで `session.idleMinutes` を設定した場合、OpenClaw は後方互換性のためアイドル専用モードのまま動作します。
* タイプ別のオーバーライド (オプション): `resetByType` を使うと、`dm`、`group`、`thread` セッションごとにポリシーを上書きできます (thread = Slack/Discord のスレッド、Telegram トピック、コネクターが提供する場合の Matrix スレッド)。
* チャネル別のオーバーライド (オプション): `resetByChannel` はチャネルごとにリセットポリシーを上書きします (そのチャネル内のすべてのセッションタイプに適用され、`reset` / `resetByType` より優先されます)。
* リセットトリガー: `/new` または `/reset` と完全一致するメッセージ (および `resetTriggers` に含まれる追加トリガー) は、新しいセッション ID を開始し、残りのメッセージ部分は通常どおり処理されます。`/new <model>` はモデルエイリアス、`provider/model`、またはプロバイダー名 (あいまい一致) を受け取り、新しいセッションのモデルを設定します。`/new` または `/reset` だけが送信された場合、OpenClaw はリセット確認のための短い「hello」挨拶ターンを実行します。
* 手動リセット: ストアから特定のキーを削除するか、JSONL トランスクリプトを削除します。次のメッセージでそれらが再作成されます。
* 分離された cron ジョブは、実行ごとに必ず新しい `sessionId` を発行します (アイドル再利用なし)。

<div id="send-policy-optional">
  ## 送信ポリシー（オプション）
</div>

個々の ID を列挙せずに、特定のセッション種別に対して送信をブロックできます。

```json5
{
  session: {
    sendPolicy: {
      rules: [
        { action: "deny", match: { channel: "discord", chatType: "group" } },
        { action: "deny", match: { keyPrefix: "cron:" } }
      ],
      default: "allow"
    }
  }
}
```

Runtime override (owner only):

* `/send on` → このセッションでは許可する
* `/send off` → このセッションでは拒否する
* `/send inherit` → オーバーライドを解除して設定ルールを適用する
  これらは単独のメッセージとして送信して、設定が反映されるようにします。

<div id="configuration-optional-rename-example">
  ## 設定（任意の名前変更例）
</div>

```json5
// ~/.openclaw/openclaw.json
{
  session: {
    scope: "per-sender",      // グループキーを個別に保持
    dmScope: "main",          // DMの継続性(共有受信箱の場合はper-channel-peer/per-account-channel-peerを設定)
    identityLinks: {
      alice: ["telegram:123456789", "discord:987654321012345678"]
    },
    reset: {
      // デフォルト: mode=daily、atHour=4(Gatewayホストのローカル時刻)
      // idleMinutesも設定した場合、先に期限が切れた方が優先されます
      mode: "daily",
      atHour: 4,
      idleMinutes: 120
    },
    resetByType: {
      thread: { mode: "daily", atHour: 4 },
      dm: { mode: "idle", idleMinutes: 240 },
      group: { mode: "idle", idleMinutes: 120 }
    },
    resetByChannel: {
      discord: { mode: "idle", idleMinutes: 10080 }
    },
    resetTriggers: ["/new", "/reset"],
    store: "~/.openclaw/agents/{agentId}/sessions/sessions.json",
    mainKey: "main",
  }
}
```

<div id="inspecting">
  ## 確認
</div>

* `openclaw status` — ストアパスと最近のセッションを表示します。
* `openclaw sessions --json` — すべてのエントリを出力します（`--active <minutes>` でフィルタ可能）。
* `openclaw gateway call sessions.list --params '{}'` — 実行中の Gateway からセッションを取得します（リモート Gateway にアクセスする場合は `--url` / `--token` を使用）。
* チャットで単独メッセージとして `/status` を送信して、エージェントに到達可能かどうか、セッションコンテキストの使用量、現在の思考モード／詳細モードのオン／オフ設定、および WhatsApp Web の認証情報が最後に更新されたタイミングを確認します（再リンクが必要かどうかを判断するのに役立ちます）。
* `/context list` または `/context detail` を単独メッセージとして送信して、システムプロンプトとインジェクトされているワークスペースファイル（およびコンテキストに最も寄与している要素）の内容を確認します。
* `/stop` を単独メッセージとして送信して、現在の実行を中断し、そのセッションのキューにあるフォローアップをクリアし、そこから生成されたサブエージェントの実行を停止します（返信には停止された件数が含まれます）。
* `/compact`（任意の指示を付与可能）を単独メッセージとして送信して、古いコンテキストを要約し、コンテキストウィンドウの空き容量を確保します。[/concepts/compaction](/ja/concepts/compaction) を参照してください。
* JSONL 形式のトランスクリプトを直接開いて、すべてのターンを確認できます。

<div id="tips">
  ## ヒント
</div>

* プライマリキーは 1:1 チャットのトラフィック専用にし、グループはそれぞれ独自のキーを持たせてください。
* クリーンアップ処理を自動化する場合は、他の箇所のコンテキストを維持するために、ストア全体ではなく個々のキーを削除するようにしてください。

<div id="session-origin-metadata">
  ## セッションの発生元メタデータ
</div>

各セッションエントリは、そのセッションがどこから来たかを（可能な範囲で）`origin` に記録します:

* `label`: 人間向けのラベル（会話ラベル＋グループの件名/チャンネルから決定される）
* `provider`: 正規化されたチャネル ID（拡張を含む）
* `from`/`to`: 受信エンベロープからの生のルーティング ID
* `accountId`: プロバイダーアカウント ID（マルチアカウントの場合）
* `threadId`: チャンネルがサポートしている場合のスレッド/トピック ID
  `origin` フィールドはダイレクトメッセージ、チャンネル、およびグループに対して設定されます。
  コネクタが配信ルーティングのみを更新する場合（たとえば DM のメインセッションを最新状態に保つためなど）でも、セッションが説明用メタデータを保持できるように、受信コンテキストは必ず提供する必要があります。
  拡張機能は、受信コンテキストで `ConversationLabel`、`GroupSubject`、`GroupChannel`、`GroupSpace`、`SenderName` を送信し、`recordSessionMetaFromInbound` を呼び出す（または同じコンテキストを `updateLastRoute` に渡す）ことでこれを行えます。