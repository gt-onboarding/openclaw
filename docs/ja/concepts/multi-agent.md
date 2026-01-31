---
summary: "マルチエージェント・ルーティング：分離されたエージェント、チャネルアカウント、バインディング"
title: マルチエージェント・ルーティング
read_when: "1つの Gateway プロセス内で、ワークスペースと認証が分離された複数のエージェントが必要な場合。"
status: active
---

<div id="multi-agent-routing">
  # マルチエージェントルーティング
</div>

目的：1 つの稼働中の Gateway 上で、複数の *分離された* エージェント（個別のワークスペース + `agentDir` + セッション）と、複数のチャネルアカウント（例：2つの WhatsApp）を扱えるようにすること。受信メッセージはバインディングによってエージェントにルーティングされる。

<div id="what-is-one-agent">
  ## 「1つのエージェント」とは？
</div>

**エージェント**は、それぞれが完全にスコープされた「頭脳」であり、次のものをそれぞれ個別に持ちます：

* **ワークスペース**（ファイル、AGENTS.md/SOUL.md/USER.md、ローカルノート、ペルソナ・ルール）。
* 認証プロファイル、モデルレジストリ、エージェント単位の設定のための **状態ディレクトリ**（`agentDir`）。
* `~/.openclaw/agents/<agentId>/sessions` 配下にある **セッションストア**（チャット履歴＋ルーティング状態）。

認証プロファイルは **エージェント単位** です。各エージェントは自分専用の次のものから読み取ります：

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

メインエージェントの認証情報は自動的には共有されません。`agentDir` をエージェント間で再利用しないでください（認証／セッションの衝突を引き起こします）。認証情報を共有したい場合は、`auth-profiles.json` を別のエージェントの `agentDir` にコピーしてください。

スキルは各ワークスペースの `skills/` フォルダを通じてエージェントごとに管理され、共有スキルは `~/.openclaw/skills` から利用できます。[Skills: per-agent vs shared](/ja/tools/skills#per-agent-vs-shared-skills) を参照してください。

Gateway は **1 つのエージェント**（デフォルト）だけでなく、**多数のエージェント**を並列でホストできます。

**ワークスペースに関する注意:** 各エージェントのワークスペースは **デフォルトのカレントディレクトリ（cwd）** であり、厳密なサンドボックスではありません。相対パスはワークスペース内で解決されますが、絶対パスはサンドボックス化（sandboxing）が有効になっていない限り、ホスト上の他の場所にも到達できます。[Sandboxing](/ja/gateway/sandboxing) を参照してください。

<div id="paths-quick-map">
  ## パス（クイックマップ）
</div>

* Config（設定）: `~/.openclaw/openclaw.json`（または `OPENCLAW_CONFIG_PATH`）
* State dir（状態ディレクトリ）: `~/.openclaw`（または `OPENCLAW_STATE_DIR`）
* Workspace（ワークスペース）: `~/.openclaw/workspace`（または `~/.openclaw/workspace-<agentId>`）
* Agent dir（エージェントディレクトリ）: `~/.openclaw/agents/<agentId>/agent`（または `agents.list[].agentDir`）
* Sessions（セッション）: `~/.openclaw/agents/<agentId>/sessions`

<div id="single-agent-mode-default">
  ### シングルエージェントモード（デフォルト）
</div>

何も設定しない場合、OpenClaw は単一のエージェントとして動作します:

* `agentId` のデフォルトは **`main`** です。
* セッションは `agent:main:<mainKey>` というキー形式になります。
* ワークスペースのデフォルトは `~/.openclaw/workspace`（`OPENCLAW_PROFILE` が設定されている場合は `~/.openclaw/workspace-<profile>`）です。
* 状態の保存先のデフォルトは `~/.openclaw/agents/main/agent` です。

<div id="agent-helper">
  ## エージェントヘルパー
</div>

新しい分離エージェントを追加するには、エージェントウィザードを使用してください。

```bash
openclaw agents add work
```

次に、受信メッセージをルーティングするための `bindings` を追加します（またはウィザードに任せます）。

次の方法で確認します：

```bash
openclaw agents list --bindings
```

<div id="multiple-agents-multiple-people-multiple-personalities">
  ## 複数エージェント = 複数の人間、複数の人格
</div>

**複数のエージェント**を使うと、それぞれの `agentId` が**完全に分離されたペルソナ**として扱われます。

* **異なる電話番号/アカウント**（チャネルごとの `accountId` 単位）。
* **異なる人格**（エージェントごとのワークスペース内にある `AGENTS.md` や `SOUL.md` などのファイル）。
* **個別の認証 + セッション**（明示的に有効化しない限り相互干渉なし）。

これにより、**複数の人**が 1 つの Gateway サーバーを共有しつつ、それぞれの AI の「頭脳」とデータを分離したまま利用できます。

<div id="one-whatsapp-number-multiple-people-dm-split">
  ## WhatsApp番号1つで複数人に対応（DM分岐）
</div>

**1つのWhatsAppアカウント**上で、**異なるWhatsAppのDM**を別々のエージェントにルーティングできます。送信者のE.164番号（`+15551234567`のような形式）と `peer.kind: "dm"` を条件にマッチさせます。返信はすべて同じWhatsApp番号から送信されるため、エージェントごとの送信元識別子は付きません。

重要なポイント：DM（1対1チャット）はエージェントの**メインセッションキー**に集約されるため、厳密に分離したい場合は**1人あたり1エージェント**が必要です。

例：

```json5
{
  agents: {
    list: [
      { id: "alex", workspace: "~/.openclaw/workspace-alex" },
      { id: "mia", workspace: "~/.openclaw/workspace-mia" }
    ]
  },
  bindings: [
    { agentId: "alex", match: { channel: "whatsapp", peer: { kind: "dm", id: "+15551230001" } } },
    { agentId: "mia",  match: { channel: "whatsapp", peer: { kind: "dm", id: "+15551230002" } } }
  ],
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551230001", "+15551230002"]
    }
  }
}
```

注意：

* DM アクセス制御は **WhatsApp アカウント単位でのグローバル設定**（ペアリング／許可リスト）であり、エージェント単位ではありません。
* 共有グループの場合は、そのグループを1つのエージェントに紐付けるか、[Broadcast グループ](/ja/broadcast-groups) を使用してください。

<div id="routing-rules-how-messages-pick-an-agent">
  ## ルーティングルール（メッセージがどのエージェントに送られるか）
</div>

バインディングは**決定的**で、かつ**より条件が厳密なものが優先**されます:

1. `peer` の一致（DM / グループ / チャンネル ID の完全一致）
2. `guildId`（Discord）
3. `teamId`（Slack）
4. チャンネルの `accountId` の一致
5. チャンネルレベルの一致（`accountId: "*"`）
6. デフォルトエージェントへのフォールバック（`agents.list[].default`、なければリストの先頭エントリ、既定: `main`）

<div id="multiple-accounts-phone-numbers">
  ## 複数アカウント / 複数電話番号
</div>

**複数アカウント** をサポートするチャネル（例: WhatsApp）は、各ログインを識別するために `accountId` を使用します。各 `accountId` は別々のエージェントにルーティングできるため、1台のサーバーで複数の電話番号を扱ってもセッションが混在することはありません。

<div id="concepts">
  ## 概念
</div>

* `agentId`: 1 つの「脳」（ワークスペース、エージェントごとの認証、エージェントごとのセッションストア）。
* `accountId`: 1 つのチャネルアカウントインスタンス（例: WhatsApp アカウント `"personal"` と `"biz"`）。
* `binding`: 受信メッセージを `(channel, accountId, peer)` と、必要に応じてギルド/チーム ID によって `agentId` へルーティングするもの。
* ダイレクトチャットは `agent:<agentId>:<mainKey>`（エージェントごとの「メイン」（`session.mainKey`））に集約される。

<div id="example-two-whatsapps-two-agents">
  ## 例：2つのWhatsApp → 2つのエージェント
</div>

`~/.openclaw/openclaw.json` (JSON5):

```js
{
  agents: {
    list: [
      {
        id: "home",
        default: true,
        name: "Home",
        workspace: "~/.openclaw/workspace-home",
        agentDir: "~/.openclaw/agents/home/agent",
      },
      {
        id: "work",
        name: "Work",
        workspace: "~/.openclaw/workspace-work",
        agentDir: "~/.openclaw/agents/work/agent",
      },
    ],
  },

  // Deterministic routing: first match wins (most-specific first).
  bindings: [
    { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },

    // Optional per-peer override (example: send a specific group to work agent).
    {
      agentId: "work",
      match: {
        channel: "whatsapp",
        accountId: "personal",
        peer: { kind: "group", id: "1203630...@g.us" },
      },
    },
  ],

  // Off by default: agent-to-agent messaging must be explicitly enabled + allowlisted.
  tools: {
    agentToAgent: {
      enabled: false,
      allow: ["home", "work"],
    },
  },

  channels: {
    whatsapp: {
      accounts: {
        personal: {
          // オプションのオーバーライド。デフォルト: ~/.openclaw/credentials/whatsapp/personal
          // authDir: "~/.openclaw/credentials/whatsapp/personal",
        },
        biz: {
          // Optional override. Default: ~/.openclaw/credentials/whatsapp/biz
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        },
      },
    },
  },
}
```

<div id="example-whatsapp-daily-chat-telegram-deep-work">
  ## 例：WhatsApp での日常チャット + Telegram でのディープワーク
</div>

チャネルごとに振り分けます：WhatsApp は高速な日常用エージェントに、Telegram は Opus エージェントにルーティングします。

```json5
{
  agents: {
    list: [
      {
        id: "chat",
        name: "Everyday",
        workspace: "~/.openclaw/workspace-chat",
        model: "anthropic/claude-sonnet-4-5"
      },
      {
        id: "opus",
        name: "Deep Work",
        workspace: "~/.openclaw/workspace-opus",
        model: "anthropic/claude-opus-4-5"
      }
    ]
  },
  bindings: [
    { agentId: "chat", match: { channel: "whatsapp" } },
    { agentId: "opus", match: { channel: "telegram" } }
  ]
}
```

注意事項:

* あるチャネルに複数のアカウントがある場合は、バインディングに `accountId` を追加します（例: `{ channel: "whatsapp", accountId: "personal" }`）。
* 特定の DM/グループだけを Opus にルーティングし、それ以外を chat に残したい場合は、その peer に対して `match.peer` バインディングを追加します。peer のマッチは、常にチャネル全体のルールより優先されます。

<div id="example-same-channel-one-peer-to-opus">
  ## 例: 同じチャネルで、特定の1人だけを Opus にルーティングする
</div>

WhatsApp は高速なエージェントに任せたまま、特定の1件の DM だけを Opus にルーティングします:

```json5
{
  agents: {
    list: [
      { id: "chat", name: "Everyday", workspace: "~/.openclaw/workspace-chat", model: "anthropic/claude-sonnet-4-5" },
      { id: "opus", name: "Deep Work", workspace: "~/.openclaw/workspace-opus", model: "anthropic/claude-opus-4-5" }
    ]
  },
  bindings: [
    { agentId: "opus", match: { channel: "whatsapp", peer: { kind: "dm", id: "+15551234567" } } },
    { agentId: "chat", match: { channel: "whatsapp" } }
  ]
}
```

ピアバインディングが常に優先されるため、チャネル全体に適用されるルールよりも上に配置してください。

<div id="family-agent-bound-to-a-whatsapp-group">
  ## WhatsApp グループに紐付けられたファミリーエージェント
</div>

専用のファミリーエージェントを 1 つの WhatsApp グループに紐付け、
メンションを条件とするゲーティングと、より厳格なツールポリシーを適用します。

```json5
{
  agents: {
    list: [
      {
        id: "family",
        name: "Family",
        workspace: "~/.openclaw/workspace-family",
        identity: { name: "Family Bot" },
        groupChat: {
          mentionPatterns: ["@family", "@familybot", "@Family Bot"]
        },
        sandbox: {
          mode: "all",
          scope: "agent"
        },
        tools: {
          allow: ["exec", "read", "sessions_list", "sessions_history", "sessions_send", "sessions_spawn", "session_status"],
          deny: ["write", "edit", "apply_patch", "browser", "canvas", "nodes", "cron"]
        }
      }
    ]
  },
  bindings: [
    {
      agentId: "family",
      match: {
        channel: "whatsapp",
        peer: { kind: "group", id: "120363999999999999@g.us" }
      }
    }
  ]
}
```

Notes:

* ツールの allow/deny リストは **ツール** に対するものであり、スキル向けではありません。スキルがバイナリを実行する必要がある場合は、`exec` が許可されており、そのバイナリがサンドボックス内に存在することを確認してください。
* より厳格に制御したい場合は、`agents.list[].groupChat.mentionPatterns` を設定し、チャネルのグループ許可リストを有効にしたままにしてください。

<div id="per-agent-sandbox-and-tool-configuration">
  ## エージェントごとのサンドボックスおよびツール設定
</div>

v2026.1.6 以降、各エージェントごとにサンドボックスとツール制限を個別に設定できます。

```js
{
  agents: {
    list: [
      {
        id: "personal",
        workspace: "~/.openclaw/workspace-personal",
        sandbox: {
          mode: "off",  // No sandbox for personal agent
        },
        // No tool restrictions - all tools available
      },
      {
        id: "family",
        workspace: "~/.openclaw/workspace-family",
        sandbox: {
          mode: "all",     // Always sandboxed
          scope: "agent",  // One container per agent
          docker: {
            // Optional one-time setup after container creation
            setupCommand: "apt-get update && apt-get install -y git curl",
          },
        },
        tools: {
          allow: ["read"],                    // Only read tool
          deny: ["exec", "write", "edit", "apply_patch"],    // その他を拒否
        },
      },
    ],
  },
}
```

注意: `setupCommand` は `sandbox.docker` 配下にあり、コンテナ作成時に一度だけ実行されます。
最終的な scope が `"shared"` の場合、エージェントごとの `sandbox.docker.*` オーバーライドは無視されます。

**利点:**

* **セキュリティ分離**: 信頼できないエージェント向けにツールを制限する
* **リソース制御**: 特定のエージェントだけをサンドボックス化し、他はホスト上で動かす
* **柔軟なポリシー**: エージェントごとに異なる権限を設定できる

注意: `tools.elevated` は **グローバル** かつ送信者ベースであり、エージェント単位では設定できません。
エージェントごとの境界が必要な場合は、`agents.list[].tools` を使って `exec` を拒否してください。
グループターゲティングには、`agents.list[].groupChat.mentionPatterns` を使用し、@メンションが意図したエージェントに正確にマッピングされるようにします。

詳細な例については [Multi-Agent Sandbox &amp; Tools](/ja/multi-agent-sandbox-tools) を参照してください。
