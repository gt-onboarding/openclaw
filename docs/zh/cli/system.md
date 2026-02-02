---
title: 系统
summary: "用于 `openclaw system` 的 CLI 参考（系统事件、心跳、在线状态）"
read_when:
  - 你想在不创建 cron 任务的情况下将系统事件加入队列
  - 你需要启用或禁用心跳
  - 你想查看系统在线状态记录
---

<div id="openclaw-system">
  # `openclaw system`
</div>

Gateway 的系统级辅助命令：将系统事件入队、控制心跳并查看在线状态。

<div id="common-commands">
  ## 常用命令
</div>

```bash
openclaw system event --text "Check for urgent follow-ups" --mode now
openclaw system heartbeat enable
openclaw system heartbeat last
openclaw system presence
```


<div id="system-event">
  ## `system event`
</div>

在**主**会话中入队一个系统事件。下一次心跳会将其作为提示中的一行 `System:` 注入。使用 `--mode now` 以立即触发心跳；`next-heartbeat` 会等待下一次预定心跳触发。

Flags:

- `--text <text>`：必需的系统事件文本。
- `--mode <mode>`：`now` 或 `next-heartbeat`（默认）。
- `--json`：机器可读的输出。

<div id="system-heartbeat-lastenabledisable">
  ## `system heartbeat last|enable|disable`
</div>

心跳控制：

- `last`: 显示最近一次心跳事件。
- `enable`: 重新开启心跳（如果之前被禁用，使用此命令恢复）。
- `disable`: 暂停心跳。

选项：

- `--json`: 机器可读的输出。

<div id="system-presence">
  ## `system presence`
</div>

列出 Gateway 当前已知的系统在线状态条目（节点、实例以及类似的状态行）。

选项：

- `--json`: 机器可读格式的输出。

<div id="notes">
  ## 注意事项
</div>

- 需要有一个正在运行的 Gateway，并且当前配置（本地或远程）能够访问它。
- 系统事件是临时的，不会在重启之间持久化存储。