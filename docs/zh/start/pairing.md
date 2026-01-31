---
title: 配对
summary: "配对概览：批准谁可以向你发送私信，以及哪些节点可以加入"
read_when:
  - 设置私信访问控制
  - 为新的 iOS/Android 节点配对
  - 审查 OpenClaw 安全状况
---

<div id="pairing">
  # 配对
</div>

“配对” 是 OpenClaw 中明确的**所有者审批**步骤。
它用于两个场景：

1. **DM 配对**（谁可以与机器人对话）
2. **节点配对**（哪些设备/节点被允许加入 Gateway 网络）

安全上下文：[安全](/zh/gateway/security)

<div id="1-dm-pairing-inbound-chat-access">
  ## 1) 私信配对（入站聊天访问）
</div>

当某个通道的私信策略配置为 `pairing` 时，未知发送者会收到一个短配对码，在你批准之前，他们的消息**不会被处理**。

默认私信策略说明见：[Security](/zh/gateway/security)

配对码：

* 8 个字符，全大写字母，不包含易混淆字符（`0O1I`）。
* **1 小时后过期**。仅在创建新请求时（大致每个发送者每小时一次），机器人才会发送配对消息。
* 待处理的私信配对请求默认**每个通道最多 3 个**；超出的请求会被忽略，直到有请求过期或被批准。

<div id="approve-a-sender">
  ### 批准消息发送方
</div>

```bash
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

支持的渠道：`telegram`、`whatsapp`、`signal`、`imessage`、`discord`、`slack`。

### 状态存储位置

存储在 `~/.openclaw/credentials/` 目录下：

* 待处理请求：`<channel>-pairing.json`
* 已批准的允许列表：`<channel>-allowFrom.json`

将这些文件视为敏感数据（它们控制对你的助手的访问权限）。

<div id="2-node-device-pairing-iosandroidmacosheadless-nodes">
  ## 2) 节点设备配对（iOS/Android/macOS/无头节点）
</div>

节点以 `role: node` 的**设备**身份连接到 Gateway。Gateway
会创建一个必须经批准的设备配对请求。

<div id="approve-a-node-device">
  ### 批准节点设备
</div>

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
```

<div id="where-the-state-lives">
  ### 状态数据存放位置
</div>

存储在 `~/.openclaw/devices/` 目录下：

* `pending.json`（短暂存在；待处理请求会过期）
* `paired.json`（已配对设备及其令牌）

<div id="notes">
  ### 注意事项
</div>

* 旧版 `node.pair.*` API（CLI：`openclaw nodes pending/approve`）使用的是
  一个由 Gateway 持有的独立配对存储。WS 节点仍然需要进行设备配对。

<div id="related-docs">
  ## 相关文档
</div>

* 安全模型与提示词注入：[Security](/zh/gateway/security)
* 安全更新（运行 doctor 命令）：[Updating](/zh/install/updating)
* 通道配置：
  * Telegram：[Telegram](/zh/channels/telegram)
  * WhatsApp：[WhatsApp](/zh/channels/whatsapp)
  * Signal：[Signal](/zh/channels/signal)
  * iMessage：[iMessage](/zh/channels/imessage)
  * Discord：[Discord](/zh/channels/discord)
  * Slack：[Slack](/zh/channels/slack)