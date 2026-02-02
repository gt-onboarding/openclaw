---
title: LINE
summary: "LINE Messaging API 插件的安装、配置与使用"
read_when:
  - 你想将 OpenClaw 连接到 LINE
  - 你需要配置 LINE webhook 和凭据
  - 你希望使用 LINE 专用的消息选项
---

<div id="line-plugin">
  # LINE（插件）
</div>

LINE 通过 LINE Messaging API 连接到 OpenClaw。该插件在 Gateway 上作为
webhook 接收端运行，并使用你的 channel access token 和 channel secret 进行
身份验证。

状态：通过插件提供支持。支持私信、群聊、媒体、位置、Flex 消息、模板消息和
快速回复。不支持消息表态（reactions）和线程（threads）。

<div id="plugin-required">
  ## 必需插件
</div>

安装 LINE 插件：

```bash
openclaw plugins install @openclaw/line
```

本地代码检出（从 Git 仓库运行时）：

```bash
openclaw plugins install ./extensions/line
```


<div id="setup">
  ## 设置
</div>

1. 创建一个 LINE Developers 帐号并打开 Console：
   https://developers.line.biz/console/
2. 创建（或选择）一个 Provider，并添加一个 **Messaging API** 渠道。
3. 从该渠道的设置中复制 **Channel access token** 和 **Channel secret**。
4. 在 Messaging API 设置中启用 **Use webhook**。
5. 将 webhook URL 设置为你的 Gateway 端点（必须使用 HTTPS）：

```
https://gateway-host/line/webhook
```

Gateway 会响应 LINE 的 webhook 验证请求（GET）和传入事件（POST）。
如果你需要自定义路径，请设置 `channels.line.webhookPath` 或
`channels.line.accounts.&lt;id&gt;.webhookPath`，并相应地更新该 URL。


<div id="configure">
  ## 配置
</div>

最小配置示例：

```json5
{
  channels: {
    line: {
      enabled: true,
      channelAccessToken: "LINE_CHANNEL_ACCESS_TOKEN",
      channelSecret: "LINE_CHANNEL_SECRET",
      dmPolicy: "pairing"
    }
  }
}
```

环境变量（仅适用于默认帐户）：

* `LINE_CHANNEL_ACCESS_TOKEN`
* `LINE_CHANNEL_SECRET`

令牌/密钥文件：

```json5
{
  channels: {
    line: {
      tokenFile: "/path/to/line-token.txt",
      secretFile: "/path/to/line-secret.txt"
    }
  }
}
```

多个账户：

```json5
{
  channels: {
    line: {
      accounts: {
        marketing: {
          channelAccessToken: "...",
          channelSecret: "...",
          webhookPath: "/line/marketing"
        }
      }
    }
  }
}
```


<div id="access-control">
  ## 访问控制
</div>

私信默认进入配对流程。未知发送方会收到一个配对码，在通过批准之前，其消息都会被忽略。

```bash
openclaw pairing list line
openclaw pairing approve line <CODE>
```

允许列表和策略：

* `channels.line.dmPolicy`: `pairing | allowlist | open | disabled`
* `channels.line.allowFrom`: 用于私信的、在允许列表中的 LINE 用户 ID
* `channels.line.groupPolicy`: `allowlist | open | disabled`
* `channels.line.groupAllowFrom`: 用于群组的、在允许列表中的 LINE 用户 ID
* 按群组的覆盖设置：`channels.line.groups.<groupId>.allowFrom`

LINE ID 区分大小写。有效的 ID 格式如下：

* 用户：`U` + 32 个十六进制字符
* 群组：`C` + 32 个十六进制字符
* 聊天室：`R` + 32 个十六进制字符


<div id="message-behavior">
  ## 消息行为
</div>

- 文本按每 5000 个字符进行分块处理。
- 会去除 Markdown 格式；在可能的情况下，将代码块和表格转换为 Flex 卡片。
- 流式响应会先进行缓冲；当 Agent 代理正在处理时，LINE 会收到带有加载动画的完整分块。
- 媒体下载大小受 `channels.line.mediaMaxMb` 限制（默认 10 MB）。

<div id="channel-data-rich-messages">
  ## 频道数据（富媒体消息）
</div>

使用 `channelData.line` 发送快捷回复、位置、Flex 消息或模板消息。

```json5
{
  text: "Here you go",
  channelData: {
    line: {
      quickReplies: ["Status", "Help"],
      location: {
        title: "Office",
        address: "123 Main St",
        latitude: 35.681236,
        longitude: 139.767125
      },
      flexMessage: {
        altText: "Status card",
        contents: { /* Flex payload */ }
      },
      templateMessage: {
        type: "confirm",
        text: "Proceed?",
        confirmLabel: "Yes",
        confirmData: "yes",
        cancelLabel: "No",
        cancelData: "no"
      }
    }
  }
}
```

LINE 插件还提供了一个 `/card` 命令，用于 Flex 消息预设：

```
/card info "Welcome" "Thanks for joining!"
```


<div id="troubleshooting">
  ## 故障排查
</div>

- **Webhook 验证失败：** 确保 webhook URL 使用 HTTPS，并且
  `channelSecret` 与 LINE 控制台中的配置一致。
- **没有入站事件：** 确认 webhook 路径与 `channels.line.webhookPath` 匹配，
  并且 Gateway 可以从 LINE 正常访问。
- **媒体下载错误：** 如果媒体大小超过默认限制，请提高 `channels.line.mediaMaxMb` 的值。