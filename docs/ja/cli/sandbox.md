---
title: サンドボックス CLI
summary: "サンドボックスコンテナを管理し、適用されるサンドボックスポリシーを確認します"
read_when: "サンドボックスコンテナの管理や、サンドボックス／ツールポリシー動作のデバッグを行っているとき。"
status: active
---

<div id="sandbox-cli">
  # サンドボックス CLI
</div>

隔離されたエージェント実行用の Docker ベースのサンドボックスコンテナを管理します。

<div id="overview">
  ## 概要
</div>

OpenClaw はセキュリティ確保のために、エージェントを隔離された Docker コンテナ内で実行できます。`sandbox` コマンドは、特にアップデートや設定変更の後に、これらのコンテナの管理を支援します。

<div id="commands">
  ## コマンド
</div>

<div id="openclaw-sandbox-explain">
  ### `openclaw sandbox explain`
</div>

有効なサンドボックスのモード/scope/ワークスペースアクセス、サンドボックスのツールポリシー、および昇格されたゲート（修正に使える設定キーのパス付き）を確認します。

```bash
openclaw sandbox explain
openclaw sandbox explain --session agent:main:main
openclaw sandbox explain --agent work
openclaw sandbox explain --json
```

<div id="openclaw-sandbox-list">
  ### `openclaw sandbox list`
</div>

すべてのサンドボックスコンテナのステータスと設定を一覧表示します。

```bash
openclaw sandbox list
openclaw sandbox list --browser  # ブラウザコンテナのみをリスト表示
openclaw sandbox list --json     # JSON output
```

**出力内容には次が含まれます:**

* コンテナ名とステータス（実行中／停止中）
* Docker イメージと、設定と一致しているかどうか
* 経過時間（作成されてからの時間）
* アイドル時間（最後に使用されてからの時間）
* 関連付けられているセッション／エージェント

<div id="openclaw-sandbox-recreate">
  ### `openclaw sandbox recreate`
</div>

サンドボックスコンテナを削除し、更新されたイメージや設定での再作成を強制します。

```bash
openclaw sandbox recreate --all                # すべてのコンテナを再作成
openclaw sandbox recreate --session main       # Specific session
openclaw sandbox recreate --agent mybot        # Specific agent
openclaw sandbox recreate --browser            # Only browser containers
openclaw sandbox recreate --all --force        # Skip confirmation
```

**オプション:**

* `--all`: すべてのサンドボックスコンテナを再作成
* `--session <key>`: 特定のセッション用コンテナを再作成
* `--agent <id>`: 特定のエージェント用コンテナを再作成
* `--browser`: ブラウザコンテナのみ再作成
* `--force`: 確認メッセージをスキップ

**重要:** コンテナは、そのエージェントが次回使用されるタイミングで自動的に再作成されます。

<div id="use-cases">
  ## 利用例
</div>

<div id="after-updating-docker-images">
  ### Docker イメージ更新後
</div>

```bash
# 新しいイメージをプル
docker pull openclaw-sandbox:latest
docker tag openclaw-sandbox:latest openclaw-sandbox:bookworm-slim

# 新しいイメージを使用するよう設定を更新
# Edit config: agents.defaults.sandbox.docker.image (or agents.list[].sandbox.docker.image)

# コンテナを再作成
openclaw sandbox recreate --all
```

<div id="after-changing-sandbox-configuration">
  ### サンドボックス設定を変更した後
</div>

```bash
# 設定を編集: agents.defaults.sandbox.* (または agents.list[].sandbox.*)

# 新しい設定を適用するために再作成
openclaw sandbox recreate --all
```

<div id="after-changing-setupcommand">
  ### setupCommand を変更した後
</div>

```bash
openclaw sandbox recreate --all
# または単一のエージェントのみ:
openclaw sandbox recreate --agent family
```

<div id="for-a-specific-agent-only">
  ### 特定のエージェントのみ
</div>

```bash
# 1つのエージェントのコンテナのみを更新
openclaw sandbox recreate --agent alfred
```

<div id="why-is-this-needed">
  ## なぜこれが必要なのか？
</div>

**問題:** サンドボックス用の Docker イメージや設定を更新しても、次のようなことが起こります:

* 既存コンテナは古い設定のまま動き続ける
* コンテナは 24 時間連続でアイドル状態にならない限り削除されない
* 頻繁に使われるエージェントは、古いコンテナを事実上無期限に動かし続けてしまう

**解決策:** `openclaw sandbox recreate` を使用して、古いコンテナの削除を強制します。次に必要になったときに、現在の設定で自動的に再作成されます。

ヒント: 手動で `docker rm` を実行するのではなく、`openclaw sandbox recreate` を優先して使用してください。Gateway のコンテナ命名規則を利用することで、scope/session キーが変わったときに発生しうる不整合を回避できます。

<div id="configuration">
  ## 設定
</div>

サンドボックスの設定は、`~/.openclaw/openclaw.json` の `agents.defaults.sandbox` セクションにあります（エージェントごとの上書き設定は `agents.list[].sandbox` に記述します）。

```jsonc
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "all",                    // off, non-main, all
        "scope": "agent",                 // session, agent, shared
        "docker": {
          "image": "openclaw-sandbox:bookworm-slim",
          "containerPrefix": "openclaw-sbx-"
          // ... その他のDockerオプション
        },
        "prune": {
          "idleHours": 24,               // 24時間アイドル後に自動削除
          "maxAgeDays": 7                // 7日後に自動削除
        }
      }
    }
  }
}
```

<div id="see-also">
  ## 関連項目
</div>

* [サンドボックスに関するドキュメント](/ja/gateway/sandboxing)
* [エージェント設定](/ja/concepts/agent-workspace)
* [Doctor コマンド](/ja/gateway/doctor) - サンドボックスのセットアップ状況を確認