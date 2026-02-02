---
title: Zalo
summary: "Zalo 机器人支持状态、功能与配置"
read_when:
  - 正在处理 Zalo 功能或 Webhook
---

<div id="zalo-bot-api">
  # Zalo（Bot API）
</div>

状态：实验阶段。仅支持私聊；根据 Zalo 文档说明，群聊支持即将推出。

<div id="plugin-required">
  ## 需要插件
</div>

Zalo 以插件形式提供，默认不随核心安装一同包含。

* 通过 CLI 安装：`openclaw plugins install @openclaw/zalo`
* 或在初始引导过程中选择 **Zalo** 并确认安装提示
* 详情参见：[插件](/zh/plugin)

<div id="quick-setup-beginner">
  ## 快速设置（入门）
</div>

1. 安装 Zalo 插件：
   * 从源码检出目录安装：`openclaw plugins install ./extensions/zalo`
   * 从 npm 安装（如果已发布）：`openclaw plugins install @openclaw/zalo`
   * 或在引导流程中选择 **Zalo** 并确认安装提示
2. 设置 token：
   * 环境变量：`ZALO_BOT_TOKEN=...`
   * 或配置项：`channels.zalo.botToken: "..."`
3. 重启 Gateway（或完成引导流程）。
4. 私信（DM）访问默认使用配对模式；首次联系时批准配对码。

最小配置：

```json5
{
  channels: {
    zalo: {
      enabled: true,
      botToken: "12345689:abc-xyz",
      dmPolicy: "配对"
    }
  }
}
```

<div id="what-it-is">
  ## 功能概述
</div>

Zalo 是一款面向越南用户的消息应用；它的 Bot API 允许 Gateway 运行一个用于一对一对话的机器人。
它非常适合需要确定性路由回 Zalo 的客服支持或通知场景。

* 一个由 Gateway 管理的 Zalo Bot API 渠道。
* 确定性路由：回复会回到 Zalo；模型不会自行决定使用哪个渠道。
* 私信共享该智能体的主会话。
* 目前尚不支持群聊（Zalo 文档声明“即将推出”）。

<div id="setup-fast-path">
  ## 设置（快速设置）
</div>

<div id="1-create-a-bot-token-zalo-bot-platform">
  ### 1) 创建机器人令牌（Zalo Bot Platform）
</div>

1. 访问 **https://bot.zaloplatforms.com** 并登录。
2. 创建一个新的机器人并完成相关设置。
3. 复制机器人令牌（格式：`12345689:abc-xyz`）。

<div id="2-configure-the-token-env-or-config">
  ### 2) 配置 Token（环境变量或配置）
</div>

示例：

```json5
{
  channels: {
    zalo: {
      enabled: true,
      botToken: "12345689:abc-xyz",
      dmPolicy: "pairing"
    }
  }
}
```

环境变量选项：`ZALO_BOT_TOKEN=...`（仅适用于默认账号）。

多账号支持：使用 `channels.zalo.accounts`，为每个账号配置 token，并可选设置 `name`。

3. 重启 Gateway。Zalo 在成功解析到 token（环境变量或配置）后启动。
4. 私信（DM）访问默认采用配对模式。首次联系机器人时，请批准该配对码。

<div id="how-it-works-behavior">
  ## 工作原理（行为）
</div>

* 入站消息会被规范化为带有媒体占位符的共享通道封装格式。
* 回复始终回发到同一个 Zalo 聊天会话。
* 默认使用长轮询；可通过 `channels.zalo.webhookUrl` 启用 webhook 模式。

<div id="limits">
  ## 限制
</div>

* 外发文本会被切分为每段 2000 个字符（Zalo API 限制）。
* 媒体下载/上传大小上限由 `channels.zalo.mediaMaxMb` 控制（默认 5 MB）。
* 由于 2000 字符限制会显著降低流式输出的实用性，因此默认禁用流式输出。

<div id="access-control-dms">
  ## 访问控制（私信）
</div>

<div id="dm-access">
  ### 私信（DM）访问
</div>

* 默认：`channels.zalo.dmPolicy = "pairing"`。未知发送方会收到配对码；在批准之前，其消息会被忽略（配对码 1 小时后过期）。
* 使用以下命令进行批准：
  * `openclaw pairing list zalo`
  * `openclaw pairing approve zalo <CODE>`
* 配对是默认的令牌交换机制。详情见：[配对](/zh/start/pairing)
* `channels.zalo.allowFrom` 仅接受数字形式的用户 ID（不支持按用户名查询）。

<div id="long-polling-vs-webhook">
  ## 长轮询 vs webhook
</div>

* 默认：长轮询（无需公网 URL）。
* Webhook 模式：设置 `channels.zalo.webhookUrl` 和 `channels.zalo.webhookSecret`。
  * Webhook 密钥长度必须为 8-256 个字符。
  * Webhook URL 必须使用 HTTPS。
  * Zalo 会在请求头中携带 `X-Bot-Api-Secret-Token` 用于校验。
  * Gateway HTTP 在 `channels.zalo.webhookPath` 上处理 webhook 请求（默认为 webhook URL 的路径部分）。

**注意：**根据 Zalo API 文档，`getUpdates`（轮询）和 webhook 互斥，不能同时使用。

<div id="supported-message-types">
  ## 支持的消息类型
</div>

* **文本消息**：完全支持，并按每 2000 个字符进行分块。
* **图片消息**：下载并处理传入图片；通过 `sendPhoto` 发送图片。
* **贴纸**：会被记录但不会进一步处理（无智能体响应）。
* **不支持的类型**：会被记录（例如来自受保护用户的消息）。

<div id="capabilities">
  ## 功能概览
</div>

| 功能 | 状态 |
|---------|--------|
| 私信 | ✅ 支持 |
| 群组 | ❌ 即将支持（参考 Zalo 文档） |
| 媒体（图片） | ✅ 支持 |
| 表情回应 | ❌ 不支持 |
| 线程会话 | ❌ 不支持 |
| 投票 | ❌ 不支持 |
| 原生命令 | ❌ 不支持 |
| 流式传输 | ⚠️ 已阻止（2000 字符上限） |

<div id="delivery-targets-clicron">
  ## 投递目标（CLI/cron）
</div>

* 使用聊天 ID 作为目标。
* 示例：`openclaw message send --channel zalo --target 123456789 --message "hi"`。

<div id="troubleshooting">
  ## 故障排查
</div>

**Bot 无响应：**

* 检查 token 是否有效：`openclaw channels status --probe`
* 确认发送者已被批准（配对或 allowFrom）
* 检查 Gateway 日志：`openclaw logs --follow`

**Webhook 未接收到事件：**

* 确保 webhook URL 使用 HTTPS
* 验证密钥 token 长度为 8-256 个字符
* 确认 Gateway HTTP 端点在配置的路径上可访问
* 检查是否有 `getUpdates` 轮询在运行（两者互斥）

<div id="configuration-reference-zalo">
  ## 配置参考（Zalo）
</div>

完整配置说明： [Configuration](/zh/gateway/configuration)

提供方配置项：

* `channels.zalo.enabled`：启用/禁用频道启动。
* `channels.zalo.botToken`：来自 Zalo Bot Platform 的 bot token。
* `channels.zalo.tokenFile`：从文件路径读取 token。
* `channels.zalo.dmPolicy`：`pairing | allowlist | open | disabled`（默认：`pairing`，配对模式）。
* `channels.zalo.allowFrom`：私信（DM）允许列表（用户 ID）。`open` 表示对任意用户不设限，此时需要设置为 `"*"`。向导会提示你输入数字 ID。
* `channels.zalo.mediaMaxMb`：入站/出站媒体大小上限（MB，默认 5）。
* `channels.zalo.webhookUrl`：启用 webhook 模式（需要 HTTPS）。
* `channels.zalo.webhookSecret`：webhook 密钥（8–256 个字符）。
* `channels.zalo.webhookPath`：Gateway HTTP 服务器上的 webhook 路径。
* `channels.zalo.proxy`：API 请求使用的代理 URL。

多账号选项：

* `channels.zalo.accounts.<id>.botToken`：每个账号的 token。
* `channels.zalo.accounts.<id>.tokenFile`：每个账号的 token 文件。
* `channels.zalo.accounts.<id>.name`：显示名称。
* `channels.zalo.accounts.<id>.enabled`：启用/禁用该账号。
* `channels.zalo.accounts.<id>.dmPolicy`：每个账号的私信策略。
* `channels.zalo.accounts.<id>.allowFrom`：每个账号的允许列表。
* `channels.zalo.accounts.<id>.webhookUrl`：每个账号的 webhook URL。
* `channels.zalo.accounts.<id>.webhookSecret`：每个账号的 webhook 密钥。
* `channels.zalo.accounts.<id>.webhookPath`：每个账号的 webhook 路径。
* `channels.zalo.accounts.<id>.proxy`：每个账号的代理 URL。