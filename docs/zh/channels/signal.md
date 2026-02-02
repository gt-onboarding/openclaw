---
title: Signal
summary: "通过 signal-cli（JSON-RPC + SSE）集成 Signal 支持、配置及号码模型"
read_when:
  - 配置 Signal 支持
  - 调试 Signal 发送/接收
---

<div id="signal-signal-cli">
  # Signal (signal-cli)
</div>

状态：外部 CLI 集成。Gateway 通过 HTTP JSON-RPC 和 SSE 与 `signal-cli` 进行通信。

<div id="quick-setup-beginner">
  ## 快速设置（入门）
</div>

1. 为机器人使用一个**单独的 Signal 号码**（推荐）。
2. 安装 `signal-cli`（需要 Java）。
3. 关联机器人设备并启动守护进程：
   * `signal-cli link -n "OpenClaw"`
4. 配置 OpenClaw 并启动 Gateway。

最简配置：

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"]
    }
  }
}
```

<div id="what-it-is">
  ## 功能简介
</div>

* 通过 `signal-cli` 的 Signal 渠道（而非嵌入式 libsignal 库）。
* 确定性路由：回复始终回到 Signal。
* 私信共享该智能体的主会话；群组是隔离的（`agent:<agentId>:signal:group:<groupId>`）。

<div id="config-writes">
  ## 配置写操作
</div>

默认情况下，允许通过 Signal 写入由 `/config set|unset` 触发的配置更新（需要 `commands.config: true`）。

可通过以下方式禁用：

```json5
{
  channels: { signal: { configWrites: false } }
}
```

<div id="the-number-model-important">
  ## 号码模型（重要）
</div>

* Gateway 会连接到一个 **Signal 设备**（即 `signal-cli` 账号）。
* 如果你在**自己的个人 Signal 账号**上运行机器人，它会忽略你自己发出的消息（用于防止消息循环）。
* 如果你想实现“我给机器人发消息，它回复我”，请使用一个**单独的机器人号码**。

<div id="setup-fast-path">
  ## 设置（快速开始）
</div>

1. 安装 `signal-cli`（需要 Java）。
2. 绑定一个机器人账号：
   * 运行 `signal-cli link -n "OpenClaw"`，然后在 Signal 中扫描二维码。
3. 配置 Signal 并启动 Gateway。

示例：

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"]
    }
  }
}
```

多账户支持：使用 `channels.signal.accounts` 为每个账户进行配置，并可选指定 `name`。有关通用模式，参见 [`gateway/configuration`](/zh/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts)。

<div id="external-daemon-mode-httpurl">
  ## 外部守护进程模式（httpUrl）
</div>

如果你想自行管理 `signal-cli`（例如处理 JVM 冷启动较慢、容器初始化或共享 CPU 的情况），可以单独运行守护进程，并在 OpenClaw 中配置其地址以指向该守护进程：

```json5
{
  channels: {
    signal: {
      httpUrl: "http://127.0.0.1:8080",
      autoStart: false
    }
  }
}
```

这会跳过 OpenClaw 内部的自动拉起以及启动等待。对于自动拉起但启动较慢的情况，请设置 `channels.signal.startupTimeoutMs`。

<div id="access-control-dms-groups">
  ## 访问控制（私信 + 群组）
</div>

私信（DMs）：

* 默认值：`channels.signal.dmPolicy = "pairing"`。
* 未知发送方会收到一个配对码；在批准之前其消息都会被忽略（配对码 1 小时后失效）。
* 通过以下方式批准：
  * `openclaw pairing list signal`
  * `openclaw pairing approve signal <CODE>`
* 配对是 Signal 私信的默认令牌交换方式。详情见：[Pairing](/zh/start/pairing)
* 仅提供 UUID 的发送方（来自 `sourceUuid`）会以 `uuid:<id>` 的形式存储在 `channels.signal.allowFrom` 中。

群组（Groups）：

* `channels.signal.groupPolicy = open | allowlist | disabled`。其中 `open` 表示允许来自任意用户的不受限消息接收。
* 当 `allowlist` 已设置时，`channels.signal.groupAllowFrom` 控制谁可以在群组中触发。

<div id="how-it-works-behavior">
  ## 工作原理（行为）
</div>

* `signal-cli` 以守护进程方式运行；Gateway 通过 SSE 读取事件。
* 传入消息会被规范化为统一的通用通道封装格式。
* 回复始终会被路由回同一个号码或群组。

<div id="media-limits">
  ## 媒体与限制
</div>

* 出站文本会根据 `channels.signal.textChunkLimit` 进行分片（默认 4000）。
* 可选换行分片：将 `channels.signal.chunkMode="newline"` 以在长度分片前按空行（段落边界）拆分。
* 支持附件（从 `signal-cli` 获取的 base64）。
* 默认媒体上限：`channels.signal.mediaMaxMb`（默认 8）。
* 使用 `channels.signal.ignoreAttachments` 来跳过下载媒体。
* 群聊历史上下文使用 `channels.signal.historyLimit`（或 `channels.signal.accounts.*.historyLimit`），否则回退为 `messages.groupChat.historyLimit`。设为 `0` 可禁用（默认 50）。

<div id="typing-read-receipts">
  ## 输入状态与已读回执
</div>

* **输入状态指示**：OpenClaw 通过 `signal-cli sendTyping` 发送输入状态信号，并在回复生成期间持续刷新。
* **已读回执**：当 `channels.signal.sendReadReceipts` 为 true 时，OpenClaw 会为允许的私信转发已读回执。
* signal-cli 不提供群聊的已读回执。

<div id="reactions-message-tool">
  ## 回应（message 工具）
</div>

* 使用 `message action=react`，并设置 `channel=signal`。
* 目标：发送者的 E.164 或 UUID（使用配对输出中的 `uuid:&lt;id&gt;`；裸 UUID 也可以）。
* `messageId` 是你要添加回应的那条消息的 Signal 时间戳。
* 群组回应需要提供 `targetAuthor` 或 `targetAuthorUuid`。

示例：

```
message action=react channel=signal target=uuid:123e4567-e89b-12d3-a456-426614174000 messageId=1737630212345 emoji=🔥
message action=react channel=signal target=+15551234567 messageId=1737630212345 emoji=🔥 remove=true
message action=react channel=signal target=signal:group:<groupId> targetAuthor=uuid:<sender-uuid> messageId=1737630212345 emoji=✅
```

配置：

* `channels.signal.actions.reactions`：启用/禁用表情回应（reaction）动作（默认为 true）。
* `channels.signal.reactionLevel`：`off | ack | minimal | extensive`。
  * `off`/`ack` 禁用智能体反应（消息工具 `react` 会报错）。
  * `minimal`/`extensive` 启用智能体反应并设置引导级别。
* 按账户级别覆盖配置：`channels.signal.accounts.<id>.actions.reactions`、`channels.signal.accounts.<id>.reactionLevel`。

<div id="delivery-targets-clicron">
  ## 发送目标（CLI/cron）
</div>

* 私信（DMs）：`signal:+15551234567`（或纯 E.164 格式）。
* UUID 私信：`uuid:<id>`（或纯 UUID）。
* 群组：`signal:group:<groupId>`。
* 用户名：`username:<name>`（如果你的 Signal 账户支持）。

<div id="configuration-reference-signal">
  ## 配置参考（Signal）
</div>

完整配置： [Configuration](/zh/gateway/configuration)

提供方配置选项：

* `channels.signal.enabled`：启用/禁用频道启动。
* `channels.signal.account`：机器人账号的 E.164 格式号码。
* `channels.signal.cliPath`：`signal-cli` 的路径。
* `channels.signal.httpUrl`：完整守护进程 URL（覆盖 host/port）。
* `channels.signal.httpHost`, `channels.signal.httpPort`：守护进程绑定地址（默认 127.0.0.1:8080）。
* `channels.signal.autoStart`：自动启动守护进程（如果未设置 `httpUrl`，默认 true）。
* `channels.signal.startupTimeoutMs`：启动等待超时时间（毫秒，最大 120000）。
* `channels.signal.receiveMode`：`on-start | manual`。
* `channels.signal.ignoreAttachments`：跳过附件下载。
* `channels.signal.ignoreStories`：忽略来自守护进程的故事（stories）。
* `channels.signal.sendReadReceipts`：转发已读回执。
* `channels.signal.dmPolicy`：`pairing | allowlist | open | disabled`（默认：pairing）。
* `channels.signal.allowFrom`：DM 允许列表（E.164 或 `uuid:<id>`）。`open` 需要 `"*"`。Signal 没有用户名；请使用手机号/UUID 标识。
* `channels.signal.groupPolicy`：`open | allowlist | disabled`（默认：allowlist）。
* `channels.signal.groupAllowFrom`：群消息发件人允许列表。
* `channels.signal.historyLimit`：作为上下文包含的群消息最大数量（0 表示禁用）。
* `channels.signal.dmHistoryLimit`：按用户轮次计的 DM 历史记录上限。每用户覆盖配置：`channels.signal.dms["<phone_or_uuid>"].historyLimit`。
* `channels.signal.textChunkLimit`：出站分块大小（字符数）。
* `channels.signal.chunkMode`：`length`（默认）或 `newline`，在按长度分块前先按空行（段落边界）拆分。
* `channels.signal.mediaMaxMb`：入站/出站媒体大小上限（MB）。

相关全局选项：

* `agents.list[].groupChat.mentionPatterns`（Signal 不支持原生 @ 提及）。
* `messages.groupChat.mentionPatterns`（全局回退配置）。
* `messages.responsePrefix`。