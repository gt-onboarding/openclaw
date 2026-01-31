---
title: 执行审批
summary: "`openclaw approvals` 的 CLI 参考（用于管理 Gateway 或节点主机的执行审批）"
read_when:
  - 你想通过 CLI 编辑执行审批
  - 你需要在 Gateway 或节点主机上管理允许列表
---

<div id="openclaw-approvals">
  # `openclaw approvals`
</div>

管理 **本地主机**、**Gateway 主机** 或 **节点主机** 的执行审批。
默认情况下，命令会作用于本地磁盘上的审批文件。使用 `--gateway` 操作 Gateway，或使用 `--node` 操作特定节点。

相关：

* 执行审批：[Exec approvals](/zh/tools/exec-approvals)
* 节点：[Nodes](/zh/nodes)

<div id="common-commands">
  ## 常用命令
</div>

```bash
openclaw approvals get
openclaw approvals get --node <id|name|ip>
openclaw approvals get --gateway
```

<div id="replace-approvals-from-a-file">
  ## 使用文件替换审批列表
</div>

```bash
openclaw approvals set --file ./exec-approvals.json
openclaw approvals set --node <id|name|ip> --file ./exec-approvals.json
openclaw approvals set --gateway --file ./exec-approvals.json
```

<div id="allowlist-helpers">
  ## 允许列表辅助工具
</div>

```bash
openclaw approvals allowlist add "~/Projects/**/bin/rg"
openclaw approvals allowlist add --agent main --node <id|name|ip> "/usr/bin/uptime"
openclaw approvals allowlist add --agent "*" "/usr/bin/uname"

openclaw approvals allowlist remove "~/Projects/**/bin/rg"
```

<div id="notes">
  ## 说明
</div>

* `--node` 使用与 `openclaw nodes` 相同的解析规则（id、name、ip 或 id 前缀）。
* `--agent` 默认为 `"*"`, 适用于所有智能体。
* 节点主机必须对外声明 `system.execApprovals.get/set` 能力（macOS 应用或无头节点主机）。
* 审批文件按主机分别存储在 `~/.openclaw/exec-approvals.json` 中。