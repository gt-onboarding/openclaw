---
title: 节点
summary: "`openclaw nodes` 的 CLI 参考（list/status/approve/invoke，camera/canvas/screen）"
read_when:
  - 你在管理已配对的节点（摄像头、屏幕、画布）
  - 你需要批准请求或调用节点命令
---

<div id="openclaw-nodes">
  # `openclaw nodes`
</div>

管理已配对的节点（设备），并调用节点功能。

相关内容：

* 节点概览：[Nodes](/zh/nodes)
* 相机：[Camera nodes](/zh/nodes/camera)
* 图像：[Image nodes](/zh/nodes/images)

通用选项：

* `--url`, `--token`, `--timeout`, `--json`

<div id="common-commands">
  ## 常用命令
</div>

```bash
openclaw nodes list
openclaw nodes list --connected
openclaw nodes list --last-connected 24h
openclaw nodes pending
openclaw nodes approve <requestId>
openclaw nodes status
openclaw nodes status --connected
openclaw nodes status --last-connected 24h
```

`nodes list` 会输出待配对/已配对节点的表格。已配对节点的行会包含最近一次连接距今的时间（Last Connect）。
使用 `--connected` 只显示当前已连接的节点。使用 `--last-connected <duration>` 来
过滤在指定时间范围内有过连接的节点（例如 `24h`、`7d`）。

<div id="invoke-run">
  ## 调用与运行
</div>

```bash
openclaw nodes invoke --node <id|name|ip> --command <command> --params <json>
openclaw nodes run --node <id|name|ip> <command...>
openclaw nodes run --raw "git status"
openclaw nodes run --agent main --node <id|name|ip> --raw "git status"
```

调用参数：

* `--params <json>`: JSON 对象字符串（默认 `{}`）。
* `--invoke-timeout <ms>`: 节点调用超时时间（默认 `15000`）。
* `--idempotency-key <key>`: 可选的幂等性键。

<div id="exec-style-defaults">
  ### Exec 风格默认值
</div>

`nodes run` 会对齐模型的 exec 行为（默认值 + 审批）：

* 读取 `tools.exec.*`（以及 `agents.list[].tools.exec.*` 覆盖项）。
* 在调用 `system.run` 之前使用 exec 审批（`exec.approval.request`）。
* 当已设置 `tools.exec.node` 时，可以省略 `--node`。
* 需要一个声明了 `system.run` 的节点（macOS 配套应用或无头节点主机）。

参数：

* `--cwd <path>`：工作目录。
* `--env <key=val>`：环境变量覆盖（可重复）。
* `--command-timeout <ms>`：命令超时时间。
* `--invoke-timeout <ms>`：节点调用超时时间（默认 `30000`）。
* `--needs-screen-recording`：需要屏幕录制权限。
* `--raw <command>`：运行 shell 字符串（`/bin/sh -lc` 或 `cmd.exe /c`）。
* `--agent <id>`：以智能体为作用域的审批/允许列表（默认为已配置的智能体）。
* `--ask <off|on-miss|always>`, `--security <deny|allowlist|full>`：覆盖默认设置。