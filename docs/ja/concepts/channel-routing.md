---
title: チャンネルルーティング
summary: "チャンネルごとのルーティングルール（WhatsApp、Telegram、Discord、Slack）と共有コンテキスト"
read_when:
  - チャンネルのルーティングまたは受信トレイの動作を変更するとき
---

<div id="channels-routing">
  # チャンネルとルーティング
</div>

OpenClaw は返信を**メッセージが届いた元のチャンネルに**ルーティングします。
モデルがチャンネルを選択することはなく、ルーティングは決定論的で、ホスト構成によって制御されます。

<div id="key-terms">
  ## 重要な用語
</div>

* **Channel**: `whatsapp`, `telegram`, `discord`, `slack`, `signal`, `imessage`, `webchat`。
* **AccountId**: チャネルごとのアカウントインスタンス（サポートされている場合）。
* **AgentId**: 分離されたワークスペース + セッションストア（「脳」に相当）。
* **SessionKey**: コンテキストの保存と並行実行の制御に使用されるバケットキー。

<div id="session-key-shapes-examples">
  ## セッションキーの形式（例）
</div>

ダイレクトメッセージはエージェントの**メイン**セッションに集約されます：

* `agent:<agentId>:<mainKey>`（デフォルト: `agent:main:main`）

グループとチャンネルは、チャンネル単位で分離されたまま維持されます：

* グループ: `agent:<agentId>:<channel>:group:<id>`
* チャンネル/ルーム: `agent:<agentId>:<channel>:channel:<id>`

スレッド:

* Slack/Discord のスレッドは、ベースキーの末尾に `:thread:<threadId>` を追加します。
* Telegram のフォーラムトピックは、グループキー内に `:topic:<topicId>` を埋め込みます。

例:

* `agent:main:telegram:group:-1001234567890:topic:42`
* `agent:main:discord:channel:123456:thread:987654`

<div id="routing-rules-how-an-agent-is-chosen">
  ## ルーティングルール（どのエージェントが選ばれるか）
</div>

ルーティングは、受信メッセージごとに**1つのエージェント**だけを選択します。

1. **ピアの完全一致**（`peer.kind` と `peer.id` を指定した `bindings`）。
2. **Guild 一致**（Discord）`guildId` 経由。
3. **Team 一致**（Slack）`teamId` 経由。
4. **アカウント一致**（チャネル上の `accountId`）。
5. **チャネル一致**（そのチャネル上の任意のアカウント）。
6. **デフォルトエージェント**（`agents.list[].default`、なければリストの先頭エントリ、最終的に `main` にフォールバック）。

マッチしたエージェントによって、どのワークスペースおよびセッションストアを使用するかが決まります。

<div id="broadcast-groups-run-multiple-agents">
  ## ブロードキャストグループ（複数エージェントを実行）
</div>

ブロードキャストグループを使うと、OpenClaw が通常返信を行うタイミングで、同じ相手（ピア）に対して **複数エージェント** を同時に実行できます（例：WhatsApp グループでメンションされた／アクティベーション条件を満たした後など）。

設定:

```json5
{
  broadcast: {
    strategy: "parallel",
    "120363403215116621@g.us": ["alfred", "baerbel"],
    "+15555550123": ["support", "logger"]
  }
}
```

詳細は [Broadcast Groups](/ja/broadcast-groups) を参照してください。

<div id="config-overview">
  ## 設定の概要
</div>

* `agents.list`: 名前付きエージェント定義（ワークスペース、モデルなど）。
* `bindings`: 受信チャネル／アカウント／ピアをエージェントに対応付ける。

例:

```json5
{
  agents: {
    list: [
      { id: "support", name: "Support", workspace: "~/.openclaw/workspace-support" }
    ]
  },
  bindings: [
    { match: { channel: "slack", teamId: "T123" }, agentId: "support" },
    { match: { channel: "telegram", peer: { kind: "group", id: "-100123" } }, agentId: "support" }
  ]
}
```

<div id="session-storage">
  ## セッションストレージ
</div>

セッションストアは state ディレクトリ配下に保存されます（デフォルトは `~/.openclaw`）:

* `~/.openclaw/agents/<agentId>/sessions/sessions.json`
* JSONL のトランスクリプトはストアと同じ場所に保存されます

`session.store` と `{agentId}` テンプレートを使って、ストアパスを変更できます。

<div id="webchat-behavior">
  ## WebChat の動作
</div>

WebChat は **選択されたエージェント** に紐づき、既定ではそのエージェントのメインセッションを使用します。このため WebChat では、そのエージェントに対する複数チャネルにまたがるコンテキストを 1 つの場所で確認できます。

<div id="reply-context">
  ## 返信コンテキスト
</div>

受信した返信には、次が含まれます：

* 利用可能な場合は、`ReplyToId`、`ReplyToBody`、`ReplyToSender`。
* 引用されたコンテキストは、`Body` に `[Replying to ...]` ブロックとして追加されます。

これはすべてのチャネルで共通です。