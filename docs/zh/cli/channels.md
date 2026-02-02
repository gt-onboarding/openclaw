---
title: 渠道
summary: "`openclaw channels` 的 CLI 参考（账号、状态、登录/退出登录、日志）"
read_when:
  - 你想添加或移除渠道账号（WhatsApp/Telegram/Discord/Google Chat/Slack/Mattermost（插件）/Signal/iMessage）
  - 你想检查渠道状态或实时查看渠道日志
---

<div id="openclaw-channels">
  # `openclaw channels`
</div>

在 Gateway 上管理聊天通道账户及其运行时状态。

相关文档：

* 通道指南：[Channels](/zh/channels/index)
* Gateway 配置：[Configuration](/zh/gateway/configuration)

<div id="common-commands">
  ## 常用命令
</div>

```bash
openclaw channels list
openclaw channels status
openclaw channels capabilities
openclaw channels capabilities --channel discord --target channel:123
openclaw channels resolve --channel slack "#general" "@jane"
openclaw channels logs --channel all
```

<div id="add-remove-accounts">
  ## 添加 / 删除账户
</div>

```bash
openclaw channels add --channel telegram --token <bot-token>
openclaw channels remove --channel telegram --delete
```

提示：`openclaw channels add --help` 会显示各通道的专用选项（token、app token、signal-cli 路径等）。

<div id="login-logout-interactive">
  ## 登录 / 登出（交互模式）
</div>

```bash
openclaw channels login --channel whatsapp
openclaw channels logout --channel whatsapp
```

<div id="troubleshooting">
  ## 故障排查
</div>

* 运行 `openclaw status --deep` 进行全面探测。
* 使用 `openclaw doctor` 获取引导式修复步骤。
* `openclaw channels list` 打印出 `Claude: HTTP 403 ... user:profile` → 用量快照需要 `user:profile` scope。请使用 `--no-usage`，或提供 claude.ai 会话密钥（`CLAUDE_WEB_SESSION_KEY` / `CLAUDE_WEB_COOKIE`），或通过 Claude Code CLI 重新认证。

<div id="capabilities-probe">
  ## 能力探测
</div>

获取提供方的能力提示（在可用时包含意图/scope）以及静态功能支持信息：

```bash
openclaw channels capabilities
openclaw channels capabilities --channel discord --target channel:123
```

说明：

* `--channel` 是可选的；省略它会列出所有频道（包括扩展）。
* `--target` 接受 `channel:<id>` 或原始的数值频道 ID，并且只适用于 Discord。
* 探针因提供方而异：Discord intents + 可选频道权限；Slack 机器人 + 用户 scope；Telegram 机器人标志位 + webhook；Signal 守护进程版本；MS Teams 应用 token + Graph 角色/scope（在已知处予以标注）。没有探针的频道会显示 `Probe: unavailable`。

<div id="resolve-names-to-ids">
  ## 将名称解析为 ID
</div>

通过提供方目录将频道/用户名称解析为 ID：

```bash
openclaw channels resolve --channel slack "#general" "@jane"
openclaw channels resolve --channel discord "My Server/#support" "@someone"
openclaw channels resolve --channel matrix "Project Room"
```

Notes:

* 使用 `--kind user|group|auto` 强制指定目标类型。
* 当多个条目同名时，名称解析会优先选择处于活动状态的条目。
