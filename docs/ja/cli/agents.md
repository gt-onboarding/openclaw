---
title: エージェント群
summary: "`openclaw agents` の CLI リファレンス（一覧/追加/削除/アイデンティティ設定）"
read_when:
  - 相互に分離された複数のエージェントを使いたい場合（ワークスペース + ルーティング + 認証）
---

<div id="openclaw-agents">
  # `openclaw agents`
</div>

個別のエージェント（ワークスペース + 認証 + ルーティング単位）を管理します。

関連項目:

* マルチエージェントルーティング: [Multi-Agent Routing](/ja/concepts/multi-agent)
* エージェントワークスペース: [Agent workspace](/ja/concepts/agent-workspace)

<div id="examples">
  ## 使用例
</div>

```bash
openclaw agents list
openclaw agents add work --workspace ~/.openclaw/workspace-work
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
openclaw agents set-identity --agent main --avatar avatars/openclaw.png
openclaw agents delete work
```

<div id="identity-files">
  ## アイデンティティファイル
</div>

各エージェントのワークスペースでは、ワークスペースのルートに `IDENTITY.md` を含めることができます:

* パス例: `~/.openclaw/workspace/IDENTITY.md`
* `set-identity --from-identity` はワークスペースのルート（または明示的に指定した `--identity-file`）から read します

アバターのパスは、ワークスペースのルートからの相対パスとして解決されます。

<div id="set-identity">
  ## アイデンティティの設定
</div>

`set-identity` はフィールドを `agents.list[].identity` に書き込みます：

* `name`
* `theme`
* `emoji`
* `avatar`（ワークスペースからの相対パス、http(s) URL、または data URI）

`IDENTITY.md` から読み込みます：

```bash
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
```

フィールドを明示的に上書きする:

```bash
openclaw agents set-identity --agent main --name "OpenClaw" --emoji "🦞" --avatar avatars/openclaw.png
```

設定例:

```json5
{
  agents: {
    list: [
      {
        id: "main",
        identity: {
          name: "OpenClaw",
          theme: "space lobster",
          emoji: "🦞",
          avatar: "avatars/openclaw.png"
        }
      }
    ]
  }
}
```
