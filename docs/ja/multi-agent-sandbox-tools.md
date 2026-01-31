---
summary: "エージェント単位のサンドボックスとツール制限、その優先順位と例"
title: マルチエージェント環境のサンドボックスとツール
read_when: "マルチエージェント構成の Gateway で、エージェントごとのサンドボックスやエージェントごとのツール許可／拒否ポリシーを設定したいとき。"
status: active
---

<div id="multi-agent-sandbox-tools-configuration">
  # マルチエージェント向けサンドボックスおよびツールの設定
</div>

<div id="overview">
  ## 概要
</div>

マルチエージェント構成では、各エージェントがそれぞれ独自に次のものを持つことができます:

* **サンドボックス設定**（`agents.list[].sandbox` が `agents.defaults.sandbox` を上書き）
* **ツール制限**（`tools.allow` / `tools.deny` に加えて `agents.list[].tools`）

これにより、異なるセキュリティプロファイルを持つ複数のエージェントを同時に実行できます:

* フルアクセス可能なパーソナルアシスタント
* ツールが制限された家族/仕事用エージェント
* サンドボックス内で動作する外部向けエージェント

`setupCommand` は `sandbox.docker`（グローバルまたはエージェント単位）の下に属し、
コンテナ作成時に一度だけ実行されます。

認証はエージェント単位です。各エージェントは、自身の `agentDir` 内の auth ストアから読み取ります。場所は次のとおりです:

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

認証情報はエージェント間で共有されることは**ありません**。`agentDir` をエージェント間で再利用しないでください。
それでも認証情報を共有する必要がある場合は、`auth-profiles.json` を別のエージェントの `agentDir` にコピーしてください。

サンドボックスが実行時にどのように動作するかについては、[Sandboxing](/ja/gateway/sandboxing) を参照してください。
「なぜこれがブロックされているのか」をデバッグするには、[Sandbox vs Tool Policy vs Elevated](/ja/gateway/sandbox-vs-tool-policy-vs-elevated) および `openclaw sandbox explain` を参照してください。

***

<div id="configuration-examples">
  ## 設定例
</div>

<div id="example-1-personal-restricted-family-agent">
  ### 例 1: 個人用 + 制限付きファミリー用エージェント
</div>

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "default": true,
        "name": "Personal Assistant",
        "workspace": "~/.openclaw/workspace",
        "sandbox": { "mode": "off" }
      },
      {
        "id": "family",
        "name": "Family Bot",
        "workspace": "~/.openclaw/workspace-family",
        "sandbox": {
          "mode": "all",
          "scope": "agent"
        },
        "tools": {
          "allow": ["read"],
          "deny": ["exec", "write", "edit", "apply_patch", "process", "browser"]
        }
      }
    ]
  },
  "bindings": [
    {
      "agentId": "family",
      "match": {
        "provider": "whatsapp",
        "accountId": "*",
        "peer": {
          "kind": "group",
          "id": "120363424282127706@g.us"
        }
      }
    }
  ]
}
```

**結果:**

* `main` エージェント: ホスト上で実行され、ツールにフルアクセス可能
* `family` エージェント: Docker 内で実行（エージェントごとに 1 コンテナ）、`read` ツールのみ使用可能

***

<div id="example-2-work-agent-with-shared-sandbox">
  ### 例 2: 共有サンドボックスを使う作業用エージェント
</div>

```json
{
  "agents": {
    "list": [
      {
        "id": "personal",
        "workspace": "~/.openclaw/workspace-personal",
        "sandbox": { "mode": "off" }
      },
      {
        "id": "work",
        "workspace": "~/.openclaw/workspace-work",
        "sandbox": {
          "mode": "all",
          "scope": "shared",
          "workspaceRoot": "/tmp/work-sandboxes"
        },
        "tools": {
          "allow": ["read", "write", "apply_patch", "exec"],
          "deny": ["browser", "gateway", "discord"]
        }
      }
    ]
  }
}
```

***

<div id="example-2b-global-coding-profile-messaging-only-agent">
  ### 例 2b: グローバルコーディングプロファイル + メッセージング専用エージェント
</div>

```json
{
  "tools": { "profile": "coding" },
  "agents": {
    "list": [
      {
        "id": "support",
        "tools": { "profile": "messaging", "allow": ["slack"] }
      }
    ]
  }
}
```

**結果:**

* デフォルトのエージェント群にはコーディング用ツールが付与される
* `support` エージェントはメッセージング専用（＋ Slack ツール）

***

<div id="example-3-different-sandbox-modes-per-agent">
  ### 例 3: エージェントごとに異なるサンドボックスのモード
</div>

```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "non-main",  // Global default
        "scope": "session"
      }
    },
    "list": [
      {
        "id": "main",
        "workspace": "~/.openclaw/workspace",
        "sandbox": {
          "mode": "off"  // Override: main never sandboxed
        }
      },
      {
        "id": "public",
        "workspace": "~/.openclaw/workspace-public",
        "sandbox": {
          "mode": "all",  // オーバーライド: public は常にサンドボックス化される
          "scope": "agent"
        },
        "tools": {
          "allow": ["read"],
          "deny": ["exec", "write", "edit", "apply_patch"]
        }
      }
    ]
  }
}
```

***

<div id="configuration-precedence">
  ## 設定の優先順位
</div>

グローバル設定（`agents.defaults.*`）とエージェント固有の設定（`agents.list[].*`）が両方存在する場合：

<div id="sandbox-config">
  ### サンドボックスの設定
</div>

エージェントごとの設定は、グローバル設定よりも優先されます。

```
agents.list[].sandbox.mode > agents.defaults.sandbox.mode
agents.list[].sandbox.scope > agents.defaults.sandbox.scope
agents.list[].sandbox.workspaceRoot > agents.defaults.sandbox.workspaceRoot
agents.list[].sandbox.workspaceAccess > agents.defaults.sandbox.workspaceAccess
agents.list[].sandbox.docker.* > agents.defaults.sandbox.docker.*
agents.list[].sandbox.browser.* > agents.defaults.sandbox.browser.*
agents.list[].sandbox.prune.* > agents.defaults.sandbox.prune.*
```

**注意:**

* `agents.list[].sandbox.{docker,browser,prune}.*` は、そのエージェントに対して `agents.defaults.sandbox.{docker,browser,prune}.*` をオーバーライドします（サンドボックスのスコープが `"shared"` に解決されている場合は無視されます）。

<div id="tool-restrictions">
  ### ツール制限
</div>

フィルタリングの順序は次のとおりです:

1. **ツールプロファイル** (`tools.profile` または `agents.list[].tools.profile`)
2. **プロバイダーツールプロファイル** (`tools.byProvider[provider].profile` または `agents.list[].tools.byProvider[provider].profile`)
3. **グローバルツールポリシー** (`tools.allow` / `tools.deny`)
4. **プロバイダーツールポリシー** (`tools.byProvider[provider].allow/deny`)
5. **エージェント固有ツールポリシー** (`agents.list[].tools.allow/deny`)
6. **エージェントプロバイダーポリシー** (`agents.list[].tools.byProvider[provider].allow/deny`)
7. **サンドボックスツールポリシー** (`tools.sandbox.tools` または `agents.list[].tools.sandbox.tools`)
8. **サブエージェントツールポリシー** (`tools.subagents.tools` が該当する場合)

各レベルではツールをさらに制限できますが、前のレベルで拒否されたツールを再び許可することはできません。
`agents.list[].tools.sandbox.tools` が設定されている場合、そのエージェントについては `tools.sandbox.tools` を置き換えます。
`agents.list[].tools.profile` が設定されている場合、そのエージェントについては `tools.profile` を上書きします。
プロバイダーツールキーには `provider`（例: `google-antigravity`）または `provider/model`（例: `openai/gpt-5.2`）のいずれかを指定できます。

<div id="tool-groups-shorthands">
  ### ツールグループ（省略表記）
</div>

ツールポリシー（グローバル、エージェント、サンドボックス）では、複数の具体的なツールに展開される `group:*` 形式のエントリをサポートしています:

* `group:runtime`: `exec`, `bash`, `process`
* `group:fs`: `read`, `write`, `edit`, `apply_patch`
* `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
* `group:memory`: `memory_search`, `memory_get`
* `group:ui`: `browser`, `canvas`
* `group:automation`: `cron`, `gateway`
* `group:messaging`: `message`
* `group:nodes`: `nodes`
* `group:openclaw`: すべての OpenClaw 組み込みツール（プロバイダー系プラグインを除く）

<div id="elevated-mode">
  ### 昇格モード
</div>

`tools.elevated` はグローバルなベースライン（送信者ベースの許可リスト）です。`agents.list[].tools.elevated` で特定のエージェント向けに昇格権限をさらに絞り込めます（両方で許可されている必要があります）。

緩和パターン:

* 信頼できないエージェントでは `exec` を拒否する（`agents.list[].tools.deny: ["exec"]`）
* 制限されたエージェントにルーティングされる送信者を許可リストに追加しない
* サンドボックス実行のみにしたい場合は、グローバルで昇格を無効化する（`tools.elevated.enabled: false`）
* 機密性の高いプロファイルでは、エージェント単位で昇格を無効化する（`agents.list[].tools.elevated.enabled: false`）

***

<div id="migration-from-single-agent">
  ## 単一エージェントからの移行
</div>

**以前（単一エージェント構成の場合）：**

```json
{
  "agents": {
    "defaults": {
      "workspace": "~/.openclaw/workspace",
      "sandbox": {
        "mode": "non-main"
      }
    }
  },
  "tools": {
    "sandbox": {
      "tools": {
        "allow": ["read", "write", "apply_patch", "exec"],
        "deny": []
      }
    }
  }
}
```

**変更後（プロファイルが異なるマルチエージェント構成）：**

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "default": true,
        "workspace": "~/.openclaw/workspace",
        "sandbox": { "mode": "off" }
      }
    ]
  }
}
```

従来の `agent.*` 設定は `openclaw doctor` によって移行されます。今後は `agents.defaults` および `agents.list` の利用を優先してください。

***

<div id="tool-restriction-examples">
  ## ツール利用制限の例
</div>

<div id="read-only-agent">
  ### 読み取り専用エージェント
</div>

```json
{
  "tools": {
    "allow": ["read"],
    "deny": ["exec", "write", "edit", "apply_patch", "process"]
  }
}
```

<div id="safe-execution-agent-no-file-modifications">
  ### 安全実行エージェント（ファイルを変更しない）
</div>

```json
{
  "tools": {
    "allow": ["read", "exec", "process"],
    "deny": ["write", "edit", "apply_patch", "browser", "gateway"]
  }
}
```

<div id="communication-only-agent">
  ### 対話専用エージェント
</div>

```json
{
  "tools": {
    "allow": ["sessions_list", "sessions_send", "sessions_history", "session_status"],
    "deny": ["exec", "write", "edit", "apply_patch", "read", "browser"]
  }
}
```

***

<div id="common-pitfall-non-main">
  ## よくある落とし穴: &quot;non-main&quot;
</div>

`agents.defaults.sandbox.mode: "non-main"` はエージェント ID ではなく、`session.mainKey`（デフォルトは `"main"`）に基づいています。
グループ／チャネルのセッションは常に独自のキーを持つため、&quot;non-main&quot; として扱われ、サンドボックス化されます。特定のエージェントでサンドボックスを一切使わせたくない場合は、`agents.list[].sandbox.mode: "off"` を設定してください。

***

<div id="testing">
  ## テスト
</div>

マルチエージェントサンドボックスとツールの設定が完了したら、次を実行します:

1. **エージェント解決を確認する:**
   ```exec
   openclaw agents list --bindings
   ```

2. **サンドボックスコンテナを確認する:**
   ```exec
   docker ps --filter "name=openclaw-sbx-"
   ```

3. **ツール制限をテストする:**
   * 利用が制限されているツールを必要とするメッセージを送信する
   * エージェントが禁止されているツールを使用できないことを確認する

4. **ログを監視する:**
   ```exec
   tail -f "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}/logs/gateway.log" | grep -E "routing|sandbox|tools"
   ```

***

<div id="troubleshooting">
  ## トラブルシューティング
</div>

<div id="agent-not-sandboxed-despite-mode-all">
  ### `mode: "all"` にもかかわらずエージェントがサンドボックスで実行されない
</div>

* それを上書きしているグローバルな `agents.defaults.sandbox.mode` が設定されていないか確認する
* エージェント固有の設定が優先されるため、`agents.list[].sandbox.mode: "all"` に設定する

<div id="tools-still-available-despite-deny-list">
  ### deny リストに入れたのにツールがまだ利用できる
</div>

* ツールのフィルタ順序を確認する：global → agent → sandbox → subagent
* 各レベルでは権限を緩和することはできず、さらに制限することしかできない
* ログで確認する：`[tools] filtering tools for agent:${agentId}`

<div id="container-not-isolated-per-agent">
  ### コンテナがエージェントごとに分離されない
</div>

* エージェント専用のサンドボックス設定で `scope: "agent"` を指定する
* デフォルトは `"session"` で、セッションごとに 1 つのコンテナが作成される

***

<div id="see-also">
  ## 関連項目
</div>

* [マルチエージェントルーティング](/ja/concepts/multi-agent)
* [サンドボックス設定](/ja/gateway/configuration#agentsdefaults-sandbox)
* [セッション管理](/ja/concepts/session)