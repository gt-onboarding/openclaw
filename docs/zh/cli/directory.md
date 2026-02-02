---
title: 目录
summary: "`openclaw directory` 的 CLI 命令参考（self、peers、groups）"
read_when:
  - 你需要查询某个频道中的联系人 / 群组 / self 的 ID
  - 你正在开发一个频道目录适配器
---

<div id="openclaw-directory">
  # `openclaw directory`
</div>

对支持目录查询的通道执行目录查找（联系人/对等方、群组，以及“我”）。

<div id="common-flags">
  ## 常用参数
</div>

- `--channel <name>`：channel id/alias（配置多个 channel 时必需；仅配置一个时自动推断）
- `--account <id>`：account id（默认：使用该 channel 的默认账号）
- `--json`：输出 JSON

<div id="notes">
  ## 注意事项
</div>

- `directory` 旨在帮助你找到可以粘贴到其他命令中的 ID（尤其是 `openclaw message send --target ...`）。
- 对于许多通道，结果是基于配置的（允许列表 / 已配置的群组），而不是来自实时的提供方目录。
- 默认输出为以制表符分隔的 `id`（有时还包括 `name`）；如用于脚本，请使用 `--json`。

<div id="using-results-with-message-send">
  ## 将结果用于 `message send`
</div>

```bash
openclaw directory peers list --channel slack --query "U0"
openclaw message send --channel slack --target user:U012ABCDEF --message "hello"
```


<div id="id-formats-by-channel">
  ## ID 格式（按渠道）
</div>

- WhatsApp：`+15551234567`（私聊），`1234567890-1234567890@g.us`（群组）
- Telegram：`@username` 或数字聊天 ID；群组为数字 ID
- Slack：`user:U…` 和 `channel:C…`
- Discord：`user:<id>` 和 `channel:<id>`
- Matrix（插件）：`user:@user:server`、`room:!roomId:server`，或 `#alias:server`
- Microsoft Teams（插件）：`user:<id>` 和 `conversation:<id>`
- Zalo（插件）：用户 ID（Bot API）
- Zalo Personal / `zalouser`（插件）：从 `zca`（`me`、`friend list`、`group list`）中获取的线程 ID（私聊/群组）

<div id="self-me">
  ## 自己（“我”）
</div>

```bash
openclaw directory self --channel zalouser
```


<div id="peers-contactsusers">
  ## 成员（联系人/用户）
</div>

```bash
openclaw directory peers list --channel zalouser
openclaw directory peers list --channel zalouser --query "name"
openclaw directory peers list --channel zalouser --limit 50
```


<div id="groups">
  ## 组
</div>

```bash
openclaw directory groups list --channel zalouser
openclaw directory groups list --channel zalouser --query "work"
openclaw directory groups members --channel zalouser --group-id <id>
```
