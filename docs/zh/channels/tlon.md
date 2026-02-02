---
title: Tlon
summary: "Tlon/Urbit 支持状态、功能与配置"
read_when:
  - 在处理 Tlon/Urbit 通道相关功能时
---

<div id="tlon-plugin">
  # Tlon（插件）
</div>

Tlon 是一个构建在 Urbit 之上的去中心化即时通讯工具。OpenClaw 会连接到你的 Urbit ship，并且可以
回复私信（DM）和群聊消息。群聊中的回复默认需要 @ 提及，并且可以通过允许列表进一步限制。

状态：通过插件支持。支持私信、群组提及、线程回复，以及纯文本媒体回退
（URL 会附加在说明文字后）。目前不支持表情反应、投票以及原生媒体上传。

<div id="plugin-required">
  ## 需要插件
</div>

Tlon 以插件形式提供，不包含在核心安装中。

使用 CLI（npm 注册表）安装：

```bash
openclaw plugins install @openclaw/tlon
```

本地代码检出（从 git 仓库运行时）：

```bash
openclaw plugins install ./extensions/tlon
```

详细信息: [插件](/zh/plugin)

<div id="setup">
  ## 设置
</div>

1. 安装 Tlon 插件。
2. 获取你的 ship 的 URL 和登录码。
3. 配置 `channels.tlon`。
4. 重启 Gateway。
5. 私信机器人，或在群组频道中提及它。

最小配置（单账号）：

```json5
{
  channels: {
    tlon: {
      enabled: true,
      ship: "~sampel-palnet",
      url: "https://your-ship-host",
      code: "lidlut-tabwed-pillex-ridrup"
    }
  }
}
```

<div id="group-channels">
  ## 群组频道
</div>

自动发现功能默认启用。你也可以手动固定频道：

```json5
{
  channels: {
    tlon: {
      groupChannels: [
        "chat/~host-ship/general",
        "chat/~host-ship/support"
      ]
    }
  }
}
```

关闭自动发现功能：

```json5
{
  channels: {
    tlon: {
      autoDiscoverChannels: false
    }
  }
}
```

<div id="access-control">
  ## 访问控制
</div>

私信允许列表（留空 = 允许所有）：

```json5
{
  channels: {
    tlon: {
      dmAllowlist: ["~zod", "~nec"]
    }
  }
}
```

群组授权（默认受限）：

```json5
{
  channels: {
    tlon: {
      defaultAuthorizedShips: ["~zod"],
      authorization: {
        channelRules: {
          "chat/~host-ship/general": {
            mode: "restricted",
            allowedShips: ["~zod", "~nec"]
          },
          "chat/~host-ship/announcements": {
            mode: "open"
          }
        }
      }
    }
  }
}
```

<div id="delivery-targets-clicron">
  ## 投递目标（CLI/cron）
</div>

在 `openclaw message send` 或 cron 投递中使用以下目标：

* 私信（DM）：`~sampel-palnet` 或 `dm/~sampel-palnet`
* 群组（Group）：`chat/~host-ship/channel` 或 `group:~host-ship/channel`

<div id="notes">
  ## 说明
</div>

* 群组回复需要提及（例如 `~your-bot-ship`）后才会响应。
* 线程回复：如果入站消息位于某个线程中，OpenClaw 会在该线程中回复。
* 媒体：`sendMedia` 会回退为文本 + URL（不支持原生上传）。