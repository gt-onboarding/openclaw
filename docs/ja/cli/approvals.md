---
title: 承認
summary: "Gateway またはノード ホストにおける exec 実行承認用 `openclaw approvals` の CLI リファレンス"
read_when:
  - CLI から exec 実行承認を編集したい
  - Gateway またはノード ホスト上の許可リストを管理する必要がある
---

<div id="openclaw-approvals">
  # `openclaw approvals`
</div>

**ローカルホスト**、**Gateway ホスト**、または **ノード ホスト** の exec 承認を管理します。
デフォルトでは、コマンドはディスク上のローカル承認ファイルを操作対象とします。`--gateway` を使用すると Gateway を、`--node` を使用すると特定のノードを対象にできます。

関連項目:

* Exec 承認: [Exec approvals](/ja/tools/exec-approvals)
* ノード: [Nodes](/ja/nodes)

<div id="common-commands">
  ## 共通コマンド
</div>

```bash
openclaw approvals get
openclaw approvals get --node <id|name|ip>
openclaw approvals get --gateway
```

<div id="replace-approvals-from-a-file">
  ## ファイルの内容で承認を置き換える
</div>

```bash
openclaw approvals set --file ./exec-approvals.json
openclaw approvals set --node <id|name|ip> --file ./exec-approvals.json
openclaw approvals set --gateway --file ./exec-approvals.json
```

<div id="allowlist-helpers">
  ## 許可リスト用ヘルパー
</div>

```bash
openclaw approvals allowlist add "~/Projects/**/bin/rg"
openclaw approvals allowlist add --agent main --node <id|name|ip> "/usr/bin/uptime"
openclaw approvals allowlist add --agent "*" "/usr/bin/uname"

openclaw approvals allowlist remove "~/Projects/**/bin/rg"
```

<div id="notes">
  ## 注意事項
</div>

* `--node` は `openclaw nodes` と同じリゾルバ（id、name、ip、または id プレフィックス）を使用します。
* `--agent` のデフォルトは `"*"` で、すべてのエージェントに適用されます。
* ノードホストは `system.execApprovals.get/set` をアドバタイズ（公開）している必要があります（macOS アプリまたはヘッドレスのノードホスト）。
* 承認情報ファイルはホストごとに `~/.openclaw/exec-approvals.json` に保存されます。