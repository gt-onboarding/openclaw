---
title: グループ
summary: "各種プラットフォーム間でのグループチャットの挙動（WhatsApp/Telegram/Discord/Slack/Signal/iMessage/Microsoft Teams）"
read_when:
  - グループチャットの挙動やメンション制御を変更するとき
---

<div id="groups">
  # グループ
</div>

OpenClaw は、WhatsApp、Telegram、Discord、Slack、Signal、iMessage、Microsoft Teams など、すべてのプラットフォームでグループチャットを一貫して扱います。

<div id="beginner-intro-2-minutes">
  ## ビギナー向けイントロ（2分）
</div>

OpenClaw はあなた自身のメッセージングアカウント上で「動作」します。別個の WhatsApp ボットユーザーは存在しません。
**あなた** がグループに参加していれば、OpenClaw はそのグループを認識し、そこで応答できます。

デフォルトの挙動:

* グループは制限されています（`groupPolicy: "allowlist"`）。
* メンションゲーティングを明示的に無効化しない限り、返信にはメンションが必要です。

訳: 許可リストに登録された送信者は、OpenClaw をメンションすることでトリガーできます。

> TL;DR
>
> * **DM アクセス** は `*.allowFrom` で制御されます。
> * **グループアクセス** は `*.groupPolicy` と許可リスト（`*.groups`, `*.groupAllowFrom`）で制御されます。
> * **返信トリガー** はメンションゲーティング（`requireMention`, `/activation`）で制御されます。

クイックフロー（グループメッセージに対して何が起こるか）:

```
groupPolicy? disabled -> drop
groupPolicy? allowlist -> group allowed? no -> drop
requireMention? yes -> mentioned? no -> store for context only
otherwise -> reply
```

![Group message flow](/images/groups-flow.svg)

次のようにしたい場合:

| 目的                          | 設定する内容                                                     |
| --------------------------- | ---------------------------------------------------------- |
| すべてのグループを許可し、@メンション時のみ返信したい | `groups: { "*": { requireMention: true } }`                |
| すべてのグループへの返信を無効化したい         | `groupPolicy: "disabled"`                                  |
| 特定のグループだけ許可したい              | `groups: { "<group-id>": { ... } }` (`"*"` キーは指定しない)       |
| グループでトリガーできるのを自分だけに限定したい    | `groupPolicy: "allowlist"`, `groupAllowFrom: ["+1555..."]` |

<div id="session-keys">
  ## セッションキー
</div>

* グループセッションは `agent:<agentId>:<channel>:group:<id>` セッションキーを使用します（ルーム/チャンネルは `agent:<agentId>:<channel>:channel:<id>` を使用します）。
* Telegram のフォーラムトピックでは、グループ ID の末尾に `:topic:<threadId>` が追加され、トピックごとに独立したセッションになります。
* ダイレクトチャットはメインのセッション（または設定されていれば送信者ごとのセッション）を使用します。
* グループセッションではハートビートはスキップされます。

<div id="pattern-personal-dms-public-groups-single-agent">
  ## パターン: 個人DM + 公開グループ（単一エージェント）
</div>

はい — 「個人」のトラフィックが **DM**、そして「公開」のトラフィックが **グループ**であれば、この構成は有効です。

理由: 単一エージェントモードでは、DM は通常 **main** セッションキー（`agent:main:main`）に入り、一方でグループは常に **non-main** セッションキー（`agent:main:<channel>:group:<id>`）を使用します。`mode: "non-main"` でサンドボックスを有効化すると、これらのグループセッションは Docker 内で実行され、メインの DM セッションはホスト上に残ります。

これにより、エージェントの「脳」（共有ワークスペース + メモリ）は 1 つのまま、実行パターンを 2 つに分けられます:

* **DM**: すべてのツール（ホスト）
* **グループ**: サンドボックス + 制限付きツール（Docker）

> 本当に分離されたワークスペース／ペルソナ（「個人」と「公開」を絶対に混在させたくない）を必要とする場合は、2 つ目のエージェントとバインディングを使用してください。[Multi-Agent Routing](/ja/concepts/multi-agent) を参照してください。

例（DM はホスト上、グループはサンドボックス + メッセージング専用ツール）:

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // groups/channels are non-main -> sandboxed
        scope: "session", // 最も強力な分離(グループ/チャンネルごとに1コンテナ)
        workspaceAccess: "none"
      }
    }
  },
  tools: {
    sandbox: {
      tools: {
        // If allow is non-empty, everything else is blocked (deny still wins).
        allow: ["group:messaging", "group:sessions"],
        deny: ["group:runtime", "group:fs", "group:ui", "nodes", "cron", "gateway"]
      }
    }
  }
}
```

「ホストへは一切アクセスさせない」のではなく「グループはフォルダXだけ見られる」にしたいですか？ `workspaceAccess: "none"` はそのままにして、サンドボックス内には許可リストに登録したパスだけをマウントしてください。

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        scope: "session",
        workspaceAccess: "none",
        docker: {
          binds: [
            // ホストパス:コンテナパス:モード
            "~/FriendsShared:/data:ro"
          ]
        }
      }
    }
  }
}
```

関連項目:

* 設定キーとデフォルト値: [Gateway 設定](/ja/gateway/configuration#agentsdefaultssandbox)
* ツールがブロックされている理由のデバッグ: [サンドボックス vs ツールポリシー vs Elevated](/ja/gateway/sandbox-vs-tool-policy-vs-elevated)
* バインドマウントの詳細: [サンドボックス化](/ja/gateway/sandboxing#custom-bind-mounts)

<div id="display-labels">
  ## 表示ラベル
</div>

* UI ラベルには、利用可能な場合は `displayName` を使用し、`<channel>:<token>` という形式で表示します。
* `#room` はルーム／チャンネル用として予約されており、グループチャットには `g-<slug>` を使用します（すべて小文字にし、スペースは `-` に変換し、`#@+._-` はそのまま保持します）。

<div id="group-policy">
  ## グループポリシー
</div>

各チャンネルごとに、グループ／ルーム内メッセージの処理方法を制御します：

```json5
{
  channels: {
    whatsapp: {
      groupPolicy: "disabled", // "open" | "disabled" | "allowlist" (open: すべてのユーザーからのメッセージ受信を無制限に許可)
      groupAllowFrom: ["+15551234567"]
    },
    telegram: {
      groupPolicy: "disabled",
      groupAllowFrom: ["123456789", "@username"]
    },
    signal: {
      groupPolicy: "disabled",
      groupAllowFrom: ["+15551234567"]
    },
    imessage: {
      groupPolicy: "disabled",
      groupAllowFrom: ["chat_id:123"]
    },
    msteams: {
      groupPolicy: "disabled",
      groupAllowFrom: ["user@org.com"]
    },
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        "GUILD_ID": { channels: { help: { allow: true } } }
      }
    },
    slack: {
      groupPolicy: "allowlist",
      channels: { "#general": { allow: true } }
    },
    matrix: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["@owner:example.org"],
      groups: {
        "!roomId:example.org": { allow: true },
        "#alias:example.org": { allow: true }
      }
    }
  }
}
```

| Policy        | 動作                                        |
| ------------- | ----------------------------------------- |
| `"open"`      | グループは許可リストをバイパスするが、メンションゲーティングは引き続き適用される。 |
| `"disabled"`  | すべてのグループメッセージを完全にブロックする。                  |
| `"allowlist"` | 設定された許可リストに一致するグループ／ルームのみを許可する。           |

Notes:

* `groupPolicy` はメンションゲーティング（@メンションが必要）とは別の設定。
* WhatsApp/Telegram/Signal/iMessage/Microsoft Teams: `groupAllowFrom` を使用（フォールバック: 明示的な `allowFrom`）。
* Discord: 許可リストは `channels.discord.guilds.<id>.channels` を使用。
* Slack: 許可リストは `channels.slack.channels` を使用。
* Matrix: 許可リストは `channels.matrix.groups`（ルームID、エイリアス、または名前）を使用。送信者を制限するには `channels.matrix.groupAllowFrom` を使用し、ルームごとの `users` 許可リストにも対応。
* グループDMは別で制御される（`channels.discord.dm.*`, `channels.slack.dm.*`）。
* Telegram の許可リストはユーザーID（`"123456789"`, `"telegram:123456789"`, `"tg:123456789"`）またはユーザー名（`"@alice"` または `"alice"`）にマッチ可能。プレフィックスは大文字小文字を区別しない。
* デフォルトは `groupPolicy: "allowlist"`。グループ許可リストが空の場合、グループメッセージはブロックされる。

Quick mental model（グループメッセージの評価順）:

1. `groupPolicy` (open/disabled/allowlist)
2. グループ許可リスト（`*.groups`, `*.groupAllowFrom`, チャンネル固有の許可リスト）
3. メンションゲーティング（`requireMention`, `/activation`）

<div id="mention-gating-default">
  ## メンション制御（デフォルト）
</div>

グループメッセージでは、グループ単位で別途設定されていない限りメンションが必須です。デフォルト値はサブシステムごとに `*.groups."*"` 以下に定義されています。

ボットメッセージへの返信は、（チャンネルが返信メタデータをサポートしている場合）暗黙のメンションとして扱われます。これは Telegram、WhatsApp、Slack、Discord、Microsoft Teams に適用されます。

```json5
{
  channels: {
    whatsapp: {
      groups: {
        "*": { requireMention: true },
        "123@g.us": { requireMention: false }
      }
    },
    telegram: {
      groups: {
        "*": { requireMention: true },
        "123456789": { requireMention: false }
      }
    },
    imessage: {
      groups: {
        "*": { requireMention: true },
        "123": { requireMention: false }
      }
    }
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          mentionPatterns: ["@openclaw", "openclaw", "\\+15555550123"],
          historyLimit: 50
        }
      }
    ]
  }
}
```

注意事項:

* `mentionPatterns` は大文字小文字を区別しない（case-insensitive）正規表現です。
* メンションを明示的に扱えるサーフェスはそのまま通過し、`mentionPatterns` はフォールバックとしてのみ使用されます。
* エージェント単位での上書き設定: `agents.list[].groupChat.mentionPatterns`（複数のエージェントが同じグループを共有している場合に有用）。
* メンション検出が可能な場合（ネイティブメンションがある、または `mentionPatterns` が設定されている）にのみ、メンションによるゲート制御が適用されます。
* Discord のデフォルト設定は `channels.discord.guilds."*"` にあり、ギルド／チャンネルごとに上書き可能です。
* グループの履歴コンテキストは、すべてのチャネルで一様にラップされ、**pending のみ**（メンションによるゲート制御でスキップされたメッセージ）を対象とします。グローバルデフォルトには `messages.groupChat.historyLimit` を使用し、上書きには `channels.<channel>.historyLimit`（または `channels.<channel>.accounts.*.historyLimit`）を使用します。無効化するには `0` を設定します。

<div id="groupchannel-tool-restrictions-optional">
  ## グループ／チャネルごとのツール制限（任意）
</div>

一部のチャネル設定では、**特定のグループ／ルーム／チャネル内**で利用可能なツールを制限できます。

* `tools`: グループ全体に対して許可／拒否するツール。
* `toolsBySender`: グループ内の送信者ごとの上書き設定（キーはチャネル種別に応じて sender ID／ユーザー名／メールアドレス／電話番号など）。ワイルドカードとして `"*"` を使用します。

適用順序（より詳細に指定されたものが優先）:

1. グループ／チャネルの `toolsBySender` に一致
2. グループ／チャネルの `tools`
3. デフォルト（`"*"`）の `toolsBySender` に一致
4. デフォルト（`"*"`）の `tools`

例（Telegram）:

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { tools: { deny: ["exec"] } },
        "-1001234567890": {
          tools: { deny: ["exec", "read", "write"] },
          toolsBySender: {
            "123456789": { alsoAllow: ["exec"] }
          }
        }
      }
    }
  }
}
```

補足:

* グループ／チャネル単位のツール制限は、グローバル／エージェントのツールポリシーに加えて適用されます（どの場合も `deny` が優先されます）。
* 一部のチャネルでは、ルーム／チャネルの階層（ネスト）構造が異なります（例: Discord `guilds.*.channels.*`、Slack `channels.*`、MS Teams `teams.*.channels.*`）。

<div id="group-allowlists">
  ## グループ許可リスト
</div>

`channels.whatsapp.groups`、`channels.telegram.groups`、`channels.imessage.groups` が設定されている場合、それぞれのキーはグループ許可リストとして機能します。デフォルトのメンション動作を設定しつつ、すべてのグループを許可するには `"*"` を使用します。

よくある用途（コピー＆ペースト用）:

1. すべてのグループへの返信を無効化する

```json5
{
  channels: { whatsapp: { groupPolicy: "disabled" } }
}
```

2. 特定のグループのみ許可する（WhatsApp）

```json5
{
  channels: {
    whatsapp: {
      groups: {
        "123@g.us": { requireMention: true },
        "456@g.us": { requireMention: false }
      }
    }
  }
}
```

3. すべてのグループを許可するが、メンションを必須にする（明示的）

```json5
{
  channels: {
    whatsapp: {
      groups: { "*": { requireMention: true } }
    }
  }
}
```

4. グループではオーナーだけが起動できます（WhatsApp）

```json5
{
  channels: {
    whatsapp: {
      groupPolicy: "allowlist", // グループポリシー: 許可リスト
      groupAllowFrom: ["+15551234567"], // グループ許可送信者
      groups: { "*": { requireMention: true } } // すべてのグループ (メンション必須)
    }
  }
}
```

<div id="activation-owner-only">
  ## 有効化（オーナー専用）
</div>

グループオーナーは、グループごとに有効化状態を切り替えることができます:

* `/activation mention`
* `/activation always`

オーナーは `channels.whatsapp.allowFrom`（未設定の場合はボット自身の E.164 番号）で決まります。コマンドは単独のメッセージとして送信してください。他のチャネルでは現在 `/activation` は無視されます。

<div id="context-fields">
  ## コンテキストフィールド
</div>

グループの受信ペイロードには次のフィールドが設定されます:

* `ChatType=group`
* `GroupSubject`（判明している場合）
* `GroupMembers`（判明している場合）
* `WasMentioned`（メンションゲーティング結果）
* Telegram のフォーラムトピックには `MessageThreadId` と `IsForum` も含まれます。

エージェントの system プロンプトには、新しいグループセッションの最初のターンでグループ紹介文が含まれます。これは、モデルに対して人間のように応答すること、Markdown の表を避けること、そして `\n` を文字列としてそのまま出力しないことをリマインドします。

<div id="imessage-specifics">
  ## iMessage の詳細
</div>

* ルーティングや許可リスト設定では `chat_id:<id>` を優先して使用してください。
* チャットを一覧表示: `imsg chats --limit 20`。
* グループへの返信は常に同じ `chat_id` に送信されます。

<div id="whatsapp-specifics">
  ## WhatsApp 固有の動作
</div>

WhatsApp 固有の挙動（履歴インジェクション、メンション処理の詳細）については、[グループメッセージ](/ja/concepts/group-messages) を参照してください。