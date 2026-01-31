---
title: Nextcloud Talk
summary: "Nextcloud Talk 支持状态、功能和配置"
read_when:
  - 在开发 Nextcloud Talk 通道功能时
---

<div id="nextcloud-talk-plugin">
  # Nextcloud Talk（插件）
</div>

状态：通过插件（webhook 机器人）提供支持。支持私信、房间、表情回应和 Markdown 消息。

<div id="plugin-required">
  ## 需要插件
</div>

Nextcloud Talk 以插件形式提供，不包含在核心安装中。

通过 CLI（npm 注册表）安装：

```bash
openclaw plugins install @openclaw/nextcloud-talk
```

本地代码检出（从 git 仓库运行时）：

```bash
openclaw plugins install ./extensions/nextcloud-talk
```

如果你在配置/初始化向导过程中选择了 Nextcloud Talk，并且检测到已有 git checkout，
OpenClaw 会自动填入本地安装路径。

详情见：[插件](/zh/plugin)

<div id="quick-setup-beginner">
  ## 快速设置（入门）
</div>

1. 安装 Nextcloud Talk 插件。
2. 在你的 Nextcloud 服务器上创建一个机器人：
   ```bash
   ./occ talk:bot:install "OpenClaw" "<shared-secret>" "<webhook-url>" --feature reaction
   ```
3. 在目标房间的设置中启用该机器人。
4. 配置 OpenClaw：
   * 配置项：`channels.nextcloud-talk.baseUrl` + `channels.nextcloud-talk.botSecret`
   * 或环境变量：`NEXTCLOUD_TALK_BOT_SECRET`（仅用于默认账户）
5. 重启 Gateway（或完成引导流程）。

最小配置：

```json5
{
  channels: {
    "nextcloud-talk": {
      enabled: true,
      baseUrl: "https://cloud.example.com",
      botSecret: "shared-secret",
      dmPolicy: "pairing"  // 配对
    }
  }
}
```

<div id="notes">
  ## 说明
</div>

* 机器人不能主动发起私信（DM），必须由用户先给机器人发消息。
* Webhook URL 必须能被 Gateway 访问；如果位于代理之后，请设置 `webhookPublicUrl`。
* 机器人 API 不支持媒体上传；媒体将以 URL 形式发送。
* Webhook 请求负载不会区分私信（DM）和房间；请设置 `apiUser` + `apiPassword` 以启用房间类型查询（否则私信会被当作房间处理）。

<div id="access-control-dms">
  ## 访问控制（私信）
</div>

* 默认：`channels.nextcloud-talk.dmPolicy = "pairing"`。未知发送方会收到一个配对码。
* 批准方式：
  * `openclaw pairing list nextcloud-talk`
  * `openclaw pairing approve nextcloud-talk <CODE>`
* 公开私信：`channels.nextcloud-talk.dmPolicy="open"` 加上 `channels.nextcloud-talk.allowFrom=["*"]`（dmPolicy 设置为 open 表示允许从任何用户无限制接收私信）。

<div id="rooms-groups">
  ## 房间（群组）
</div>

* 默认值：`channels.nextcloud-talk.groupPolicy = "allowlist"`（基于提及的访问控制）。
* 使用 `channels.nextcloud-talk.rooms` 为房间启用允许列表：

```json5
{
  channels: {
    "nextcloud-talk": {
      rooms: {
        "room-token": { requireMention: true }
      }
    }
  }
}
```

* 如果不允许任何房间，只需保持允许列表为空，或设置 `channels.nextcloud-talk.groupPolicy="disabled"`。

<div id="capabilities">
  ## 功能
</div>

| 功能项 | 状态 |
|---------|--------|
| 私聊 | 已支持 |
| 房间 | 已支持 |
| 线程 | 不支持 |
| 媒体 | 仅限 URL |
| 表情回应 | 已支持 |
| 原生命令 | 不支持 |

<div id="configuration-reference-nextcloud-talk">
  ## 配置参考（Nextcloud Talk）
</div>

完整配置： [Configuration](/zh/gateway/configuration)

提供方选项：

* `channels.nextcloud-talk.enabled`: 启用/禁用此通道的启动。
* `channels.nextcloud-talk.baseUrl`: Nextcloud 实例 URL。
* `channels.nextcloud-talk.botSecret`: 机器人共享密钥。
* `channels.nextcloud-talk.botSecretFile`: 密钥文件路径。
* `channels.nextcloud-talk.apiUser`: 用于房间查询（DM 检测）的 API 用户。
* `channels.nextcloud-talk.apiPassword`: 用于房间查询的 API/应用密码。
* `channels.nextcloud-talk.apiPasswordFile`: API 密码文件路径。
* `channels.nextcloud-talk.webhookPort`: webhook 监听端口（默认：8788）。
* `channels.nextcloud-talk.webhookHost`: webhook 主机（默认：0.0.0.0）。
* `channels.nextcloud-talk.webhookPath`: webhook 路径（默认：/nextcloud-talk-webhook）。
* `channels.nextcloud-talk.webhookPublicUrl`: 外部可访问的 webhook URL。
* `channels.nextcloud-talk.dmPolicy`: `pairing | allowlist | open | disabled`。
* `channels.nextcloud-talk.allowFrom`: DM 允许列表（用户 ID）。`open` 需要 `"*"`。
* `channels.nextcloud-talk.groupPolicy`: `allowlist | open | disabled`。
* `channels.nextcloud-talk.groupAllowFrom`: 群组允许列表（用户 ID）。
* `channels.nextcloud-talk.rooms`: 每个房间的设置和允许列表。
* `channels.nextcloud-talk.historyLimit`: 群组历史记录上限（0 表示禁用）。
* `channels.nextcloud-talk.dmHistoryLimit`: DM 历史记录上限（0 表示禁用）。
* `channels.nextcloud-talk.dms`: 针对每个 DM 的覆盖配置（historyLimit）。
* `channels.nextcloud-talk.textChunkLimit`: 出站文本块大小限制（字符数）。
* `channels.nextcloud-talk.chunkMode`: `length`（默认）或 `newline`，在按长度切分前先按空行（段落边界）拆分。
* `channels.nextcloud-talk.blockStreaming`: 为此通道禁用分块流式传输。
* `channels.nextcloud-talk.blockStreamingCoalesce`: 分块流式合并调优。
* `channels.nextcloud-talk.mediaMaxMb`: 入站媒体大小上限（MB）。