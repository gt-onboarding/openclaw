---
title: Msteams
summary: "Microsoft Teams 机器人支持状态、功能和配置"
read_when:
  - 正在处理 MS Teams 通道功能
---

<div id="microsoft-teams-plugin">
  # Microsoft Teams（插件）
</div>

> “凡进入此地者，断绝一切希望。”

更新日期：2026-01-21

状态：支持文本消息和私信附件；向频道/群组发送文件需要 `sharePointSiteId` 和 Microsoft Graph 权限（参见[在群聊中发送文件](#sending-files-in-group-chats)）。投票通过 Adaptive Cards 发送。

<div id="plugin-required">
  ## 需要插件
</div>

Microsoft Teams 以插件形式提供，不随核心安装包一同打包。

**重大变更（2026.1.15）：** MS Teams 已从核心中移除。如果你使用它，必须单独安装插件。

原因说明：让核心安装更轻量，并使 MS Teams 依赖可以独立更新。

通过 CLI 从 npm registry 安装：

```bash
openclaw plugins install @openclaw/msteams
```

本地检出（从 Git 仓库运行时）：

```bash
openclaw plugins install ./extensions/msteams
```

如果你在配置/初始设置过程中选择了 Teams，并且检测到存在 git checkout，
OpenClaw 会自动填充本地安装路径。

详情参见：[插件](/zh/plugin)

<div id="quick-setup-beginner">
  ## 快速设置（入门）
</div>

1. 安装 Microsoft Teams 插件。
2. 创建一个 **Azure Bot**（应用 ID + 客户端密钥 client secret + 租户 ID tenant ID）。
3. 使用这些凭据配置 OpenClaw。
4. 通过公共 URL 或隧道将 `/api/messages`（默认端口 3978）暴露到公网。
5. 安装 Teams 应用包并启动 Gateway。

最小配置：

```json5
{
  channels: {
    msteams: {
      enabled: true,
      appId: "<APP_ID>",
      appPassword: "<APP_PASSWORD>",
      tenantId: "<TENANT_ID>",
      webhook: { port: 3978, path: "/api/messages" }
    }
  }
}
```

注意：默认会阻止群聊（`channels.msteams.groupPolicy: "allowlist"`）。如需允许群组回复，请设置 `channels.msteams.groupAllowFrom`（或使用 `groupPolicy: "open"` 以允许任何成员在被提及时发送消息；`open` 表示允许来自任意用户的不受限消息接收策略）。

<div id="goals">
  ## 目标
</div>

* 通过 Teams 私信、群组聊天或频道与 OpenClaw 交互。
* 保持路由行为确定可预测：回复始终回到其最初到达的频道。
* 默认采用安全的频道行为（除非另有配置，否则必须使用 @ 提及）。

<div id="config-writes">
  ## 配置写入
</div>

默认情况下，Microsoft Teams 可以通过 `/config set|unset` 写入配置更新（需要 `commands.config: true`）。

禁用方法：

```json5
{
  channels: { msteams: { configWrites: false } }
}
```

<div id="access-control-dms-groups">
  ## 访问控制（私信 + 群组）
</div>

**私信访问**

* 默认：`channels.msteams.dmPolicy = "pairing"`。在获得批准之前，未知发送者会被忽略。
* `channels.msteams.allowFrom` 接受 AAD 对象 ID、UPN 或显示名称。如果凭据允许，向导会通过 Microsoft Graph 将名称解析为 ID。

**群组访问**

* 默认：`channels.msteams.groupPolicy = "allowlist"`（如果你不添加 `groupAllowFrom`，群组/频道将被阻止）。在未单独设置时，你可以使用 `channels.defaults.groupPolicy` 覆盖该默认值。
* `channels.msteams.groupAllowFrom` 控制哪些发送者可以在群聊/频道中触发（未设置时会回退到 `channels.msteams.allowFrom`）。
* 将 `groupPolicy` 设置为 `"open"`，以允许任何成员触发（默认仍需通过 @ 提及触发）。
* 若要不允许**任何频道**，将 `channels.msteams.groupPolicy` 设置为 `"disabled"`。

示例：

```json5
{
  channels: {
    msteams: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["user@org.com"]
    }
  }
}
```

**Teams + 频道允许列表**

* 通过在 `channels.msteams.teams` 下列出团队和频道，将回复限制在对应的团队/频道内。
* 键可以是团队 ID 或名称；频道键可以是会话 ID 或名称。
* 当 `groupPolicy="allowlist"` 且存在 Teams 允许列表时，只有允许列表中列出的团队/频道才会被接受（需通过 @ 提及触发）。
* 配置向导支持输入 `Team/Channel` 条目，并为你保存这些配置。
* 在启动时，OpenClaw 会将团队/频道和用户允许列表中的名称解析为 ID（在 Graph 权限允许的情况下），
  并将映射关系写入日志；未能解析的条目会按原样保留。

示例：

```json5
{
  channels: {
    msteams: {
      groupPolicy: "allowlist",
      teams: {
        "My Team": {
          channels: {
            "General": { requireMention: true }
          }
        }
      }
    }
  }
}
```

<div id="how-it-works">
  ## 工作原理
</div>

1. 安装 Microsoft Teams 插件。
2. 创建一个 **Azure Bot**（App ID + 密钥 + 租户 ID）。
3. 构建一个引用该 Bot 并包含下方 RSC 权限的 **Teams 应用包**。
4. 将该 Teams 应用上传/安装到某个团队中（或用于私信的个人 scope）。
5. 在 `~/.openclaw/openclaw.json`（或环境变量）中配置 `msteams`，并启动 Gateway 服务。
6. Gateway 默认在 `/api/messages` 路径上监听来自 Bot Framework 的 webhook 请求流量。

<div id="azure-bot-setup-prerequisites">
  ## Azure Bot 配置（前置条件）
</div>

在配置 OpenClaw 之前，你需要先创建一个 Azure Bot 资源。

<div id="step-1-create-azure-bot">
  ### 步骤 1：创建 Azure Bot
</div>

1. 访问 [Create Azure Bot](https://portal.azure.com/#create/Microsoft.AzureBot)
2. 在 **Basics**（基本信息）选项卡中填写：

   | 字段 | 值 |
   |-------|-------|
   | **Bot handle** | 你的机器人名称，例如 `openclaw-msteams`（必须全局唯一） |
   | **Subscription** | 选择你的 Azure 订阅 |
   | **Resource group** | 新建或使用现有资源组 |
   | **Pricing tier** | 选择用于开发/测试的 **Free** |
   | **Type of App** | **Single Tenant**（推荐——见下方说明） |
   | **Creation type** | **Create new Microsoft App ID** |

> **弃用通知：** 自 2025-07-31 起，创建新的多租户机器人已被弃用。新建机器人请使用 **Single Tenant**。

3. 点击 **Review + create** → **Create**（等待约 1–2 分钟）

<div id="step-2-get-credentials">
  ### 步骤 2：获取凭据
</div>

1. 在你的 Azure Bot 资源中 → 打开 **Configuration**
2. 复制 **Microsoft App ID** → 这就是你的 `appId`
3. 点击 **Manage Password** → 跳转到 App Registration
4. 在 **Certificates &amp; secrets** 下 → 选择 **New client secret** → 复制 **Value** → 这就是你的 `appPassword`
5. 打开 **Overview** → 复制 **Directory (tenant) ID** → 这就是你的 `tenantId`

<div id="step-3-configure-messaging-endpoint">
  ### 步骤 3：配置消息终结点
</div>

1. 在 Azure Bot → **Configuration** 中
2. 将 **Messaging endpoint** 设置为你的 webhook URL：
   * 生产环境：`https://your-domain.com/api/messages`
   * 本地开发：使用隧道（参见下文的 [本地开发](#local-development-tunneling)）

<div id="step-4-enable-teams-channel">
  ### 步骤 4：启用 Teams 渠道
</div>

1. 在 Azure Bot 中转到 → **Channels**
2. 点击 **Microsoft Teams** → Configure → Save
3. 接受服务条款

<div id="local-development-tunneling">
  ## 本地开发（隧道）
</div>

Microsoft Teams 无法直接访问 `localhost`。本地开发时请使用隧道：

**选项 A：ngrok**

```bash
ngrok http 3978
# Copy the https URL, e.g., https://abc123.ngrok.io
# 将消息端点设置为:https://abc123.ngrok.io/api/messages
```

**选项 B：Tailscale Funnel**

```bash
tailscale funnel 3978
# 使用你的 Tailscale Funnel URL 作为消息端点
```

<div id="teams-developer-portal-alternative">
  ## Teams Developer Portal（替代方案）
</div>

与其手动创建 manifest ZIP，你可以使用 [Teams Developer Portal](https://dev.teams.microsoft.com/apps)：

1. 点击 **+ New app**
2. 填写基本信息（名称、描述、开发者信息）
3. 前往 **App features** → **Bot**
4. 选择 **Enter a bot ID manually** 并粘贴你的 Azure Bot App ID
5. 检查 scopes：**Personal**、**Team**、**Group Chat**
6. 点击 **Distribute** → **Download app package**
7. 在 Teams 中：**Apps** → **Manage your apps** → **Upload a custom app** → 选择该 ZIP

这通常比手动编辑 JSON manifest 文件更省事。

<div id="testing-the-bot">
  ## 测试 Bot
</div>

**选项 A：Azure Web Chat（先验证 webhook）**

1. 在 Azure Portal → 你的 Azure Bot 资源 → **Test in Web Chat**
2. 发送一条消息，你应该会看到回复
3. 这可以在配置 Teams 之前确认你的 webhook 端点工作正常

**选项 B：Teams（安装应用后）**

1. 安装 Teams 应用（侧载或通过组织目录）
2. 在 Teams 中找到该 bot 并发送一条私聊消息
3. 在 Gateway 日志中检查是否有传入事件

<div id="setup-minimal-text-only">
  ## 设置（精简纯文本）
</div>

1. **安装 Microsoft Teams 插件**
   * 通过 npm：`openclaw plugins install @openclaw/msteams`
   * 通过本地检出：`openclaw plugins install ./extensions/msteams`

2. **注册 Bot**
   * 创建一个 Azure Bot（见上文），并记录：
     * App ID
     * Client secret（App password）
     * Tenant ID（单租户）

3. **Teams 应用清单（manifest）**
   * 包含一个 `bot` 条目，设置 `botId = <App ID>`。
   * Scopes：`personal`、`team`、`groupChat`。
   * `supportsFiles: true`（personal scope 下的文件处理所必需）。
   * 添加 RSC 权限（见下文）。
   * 创建图标：`outline.png`（32x32）和 `color.png`（192x192）。
   * 将这三个文件一起打包为 zip 压缩包：`manifest.json`、`outline.png`、`color.png`。

4. **配置 OpenClaw**

   ```json
   {
     "msteams": {
       "enabled": true,
       "appId": "<APP_ID>",
       "appPassword": "<APP_PASSWORD>",
       "tenantId": "<TENANT_ID>",
       "webhook": { "port": 3978, "path": "/api/messages" }
     }
   }
   ```

   也可以使用环境变量替代配置键：

   * `MSTEAMS_APP_ID`
   * `MSTEAMS_APP_PASSWORD`
   * `MSTEAMS_TENANT_ID`

5. **Bot 端点**
   * 将 Azure Bot 的 Messaging Endpoint 设置为：
     * `https://<host>:3978/api/messages`（或自行选择的路径/端口）。

6. **运行 Gateway**
   * 当插件已安装且存在携带凭据的 `msteams` 配置时，Teams 通道会自动启动。

<div id="history-context">
  ## 历史上下文
</div>

* `channels.msteams.historyLimit` 控制会将多少条最近的频道/群组消息包含到提示词中。
* 若未设置则回退为使用 `messages.groupChat.historyLimit`。设置为 `0` 可禁用（默认 50）。
* 可使用 `channels.msteams.dmHistoryLimit` 限制私信历史（按用户轮次计）。按用户级覆盖：`channels.msteams.dms["<user_id>"].historyLimit`。

<div id="current-teams-rsc-permissions-manifest">
  ## 当前 Teams RSC 权限（Manifest）
</div>

以下是我们 Teams 应用 manifest 中**现有的 resourceSpecific 权限**。它们仅适用于安装了该应用的团队或聊天中。

**针对频道（团队范围）：**

* `ChannelMessage.Read.Group` (Application) - 接收所有未带 @mention 的频道消息
* `ChannelMessage.Send.Group` (Application)
* `Member.Read.Group` (Application)
* `Owner.Read.Group` (Application)
* `ChannelSettings.Read.Group` (Application)
* `TeamMember.Read.Group` (Application)
* `TeamSettings.Read.Group` (Application)

**针对群聊：**

* `ChatMessage.Read.Chat` (Application) - 接收所有未带 @mention 的群聊消息

<div id="example-teams-manifest-redacted">
  ## 示例 Teams Manifest（已删减）
</div>

包含所有必填字段的最小有效示例。请替换其中的 ID 和 URL。

```json
{
  "$schema": "https://developer.microsoft.com/en-us/json-schemas/teams/v1.23/MicrosoftTeams.schema.json",
  "manifestVersion": "1.23",
  "version": "1.0.0",
  "id": "00000000-0000-0000-0000-000000000000",
  "name": { "short": "OpenClaw" },
  "developer": {
    "name": "Your Org",
    "websiteUrl": "https://example.com",
    "privacyUrl": "https://example.com/privacy",
    "termsOfUseUrl": "https://example.com/terms"
  },
  "description": { "short": "OpenClaw in Teams", "full": "OpenClaw in Teams" },
  "icons": { "outline": "outline.png", "color": "color.png" },
  "accentColor": "#5B6DEF",
  "bots": [
    {
      "botId": "11111111-1111-1111-1111-111111111111",
      "scopes": ["personal", "team", "groupChat"],
      "isNotificationOnly": false,
      "supportsCalling": false,
      "supportsVideo": false,
      "supportsFiles": true
    }
  ],
  "webApplicationInfo": {
    "id": "11111111-1111-1111-1111-111111111111"
  },
  "authorization": {
    "permissions": {
      "resourceSpecific": [
        { "name": "ChannelMessage.Read.Group", "type": "Application" },
        { "name": "ChannelMessage.Send.Group", "type": "Application" },
        { "name": "Member.Read.Group", "type": "Application" },
        { "name": "Owner.Read.Group", "type": "Application" },
        { "name": "ChannelSettings.Read.Group", "type": "Application" },
        { "name": "TeamMember.Read.Group", "type": "Application" },
        { "name": "TeamSettings.Read.Group", "type": "Application" },
        { "name": "ChatMessage.Read.Chat", "type": "Application" }
      ]
    }
  }
}
```

<div id="manifest-caveats-must-have-fields">
  ### Manifest 注意事项（必需字段）
</div>

* `bots[].botId` **必须** 与 Azure Bot 应用 ID 匹配。
* `webApplicationInfo.id` **必须** 与 Azure Bot 应用 ID 匹配。
* `bots[].scopes` 必须包含你计划使用的使用场景（`personal`、`team`、`groupChat`）。
* 为了在 `personal` scope 中处理文件，必须设置 `bots[].supportsFiles: true`。
* 如果你需要频道流量，`authorization.permissions.resourceSpecific` 必须包含对频道的读取和发送权限。

<div id="updating-an-existing-app">
  ### 更新现有应用
</div>

要更新已安装的 Teams 应用（例如添加 RSC 权限）：

1. 使用新的设置更新你的 `manifest.json` 文件
2. **递增 `version` 字段**（例如，`1.0.0` → `1.1.0`）
3. **重新打包为 zip**，将清单和图标一起打包（`manifest.json`、`outline.png`、`color.png`）
4. 上传新的 zip：
   * **选项 A（Teams Admin Center）：** Teams Admin Center → Teams apps → Manage apps → 找到你的应用 → Upload new version
   * **选项 B（侧载 / Sideload）：** 在 Teams 中 → Apps → Manage your apps → Upload a custom app
5. **对于团队频道：** 在每个团队中重新安装该应用，使新权限生效
6. **完全退出并重新启动 Teams**（不仅仅是关闭窗口），以清除缓存的应用元数据

<div id="capabilities-rsc-only-vs-graph">
  ## 功能对比：仅 RSC 与 Graph
</div>

<div id="with-teams-rsc-only-app-installed-no-graph-api-permissions">
  ### 仅使用 **Teams RSC**（应用已安装，但未授予 Graph API 权限）
</div>

可用功能：

* 读取频道消息的 **文本** 内容。
* 发送频道消息的 **文本** 内容。
* 接收 **个人（DM）** 文件附件。

不可用功能：

* 频道/群组的 **图片或文件内容**（有效负载中仅包含 HTML 占位片段）。
* 下载存储在 SharePoint/OneDrive 中的附件。
* 读取消息历史记录（超出当前 webhook 事件范围的部分）。

<div id="with-teams-rsc-microsoft-graph-application-permissions">
  ### 使用 **Teams RSC + Microsoft Graph 应用权限**
</div>

新增能力：

* 下载托管的内容（粘贴到消息中的图片）。
* 下载存储在 SharePoint/OneDrive 中的文件附件。
* 通过 Graph 读取频道/聊天消息历史记录。

<div id="rsc-vs-graph-api">
  ### RSC 与 Graph API
</div>

| 功能 | RSC 权限 | Graph API |
|------------|-----------------|-----------|
| **实时消息** | 是（通过 webhook） | 否（仅轮询） |
| **历史消息** | 否 | 是（可查询历史） |
| **配置复杂度** | 只需应用 manifest | 需要管理员同意 + 令牌流程 |
| **离线可用性** | 否（必须保持运行） | 是（可随时查询） |

**结论：**RSC 适用于实时监听；Graph API 用于访问历史记录。要在离线期间补上错过的消息，你需要带有 `ChannelMessage.Read.All` 的 Graph API（需要管理员同意）。

<div id="graph-enabled-media-history-required-for-channels">
  ## 启用 Graph 的媒体和历史记录（频道所必需）
</div>

如果你在**频道**中需要图片/文件，或希望获取**消息历史记录**，必须启用 Microsoft Graph 权限并授予管理员同意。

1. 在 Entra ID（Azure AD）的 **App Registration** 中，添加 Microsoft Graph 的 **Application permissions**：
   * `ChannelMessage.Read.All`（频道附件 + 历史记录）
   * `Chat.Read.All` 或 `ChatMessage.Read.All`（群组聊天）
2. 为租户**授予管理员同意**。
3. 更新 Teams 应用的 **manifest 版本**，重新上传，并**在 Teams 中重新安装该应用**。
4. **完全退出并重新启动 Teams**，以清除缓存的应用元数据。

<div id="known-limitations">
  ## 已知限制
</div>

<div id="webhook-timeouts">
  ### Webhook 超时
</div>

Teams 通过 HTTP webhook 发送消息。如果处理耗时过长（例如 LLM 响应较慢），你可能会遇到：

* Gateway 超时
* Teams 重试发送该消息（导致重复）
* 回复被丢弃

OpenClaw 通过快速返回并主动发送回复来处理这种情况，但响应过慢时仍然可能出现问题。

<div id="formatting">
  ### 格式
</div>

Teams 的 markdown 功能比 Slack 或 Discord 更受限：

* 基本格式可用：**粗体**、*斜体*、`code`、链接
* 复杂的 markdown（如表格、嵌套列表）可能无法正确渲染
* 支持使用 Adaptive Cards 创建投票并发送任意卡片（见下文）

<div id="configuration">
  ## 配置
</div>

关键设置（共享通道通用配置模式参见 `/gateway/configuration`）：

* `channels.msteams.enabled`：启用/禁用此通道。
* `channels.msteams.appId`, `channels.msteams.appPassword`, `channels.msteams.tenantId`：bot 凭据。
* `channels.msteams.webhook.port`（默认值 `3978`）
* `channels.msteams.webhook.path`（默认值 `/api/messages`）
* `channels.msteams.dmPolicy`：`pairing | allowlist | open | disabled`（默认：pairing）
* `channels.msteams.allowFrom`：用于私信 (DM) 的允许列表（AAD 对象 ID、UPN 或显示名称）。在具备 Graph 访问权限时，向导会在设置过程中将名称解析为 ID。
* `channels.msteams.textChunkLimit`：出站文本分块大小。
* `channels.msteams.chunkMode`：`length`（默认）或 `newline`，在按长度分块前，按空行（段落边界）拆分。
* `channels.msteams.mediaAllowHosts`：入站附件主机的允许列表（默认为 Microsoft/Teams 域）。
* `channels.msteams.requireMention`：在频道/群组中要求使用 @ 提及（默认 true）。
* `channels.msteams.replyStyle`：`thread | top-level`（见 [回复样式](#reply-style-threads-vs-posts)）。
* `channels.msteams.teams.<teamId>.replyStyle`：团队级覆盖配置。
* `channels.msteams.teams.<teamId>.requireMention`：团队级覆盖配置。
* `channels.msteams.teams.<teamId>.tools`：默认的团队级工具策略覆盖配置（`allow`/`deny`/`alsoAllow`），当缺少频道级覆盖时生效。
* `channels.msteams.teams.<teamId>.toolsBySender`：默认的团队级、按发送者划分的工具策略覆盖配置（支持 `"*"` 通配符）。
* `channels.msteams.teams.<teamId>.channels.<conversationId>.replyStyle`：频道级覆盖配置。
* `channels.msteams.teams.<teamId>.channels.<conversationId>.requireMention`：频道级覆盖配置。
* `channels.msteams.teams.<teamId>.channels.<conversationId>.tools`：频道级工具策略覆盖配置（`allow`/`deny`/`alsoAllow`）。
* `channels.msteams.teams.<teamId>.channels.<conversationId>.toolsBySender`：频道级、按发送者划分的工具策略覆盖配置（支持 `"*"` 通配符）。
* `channels.msteams.sharePointSiteId`：在群聊/频道中文件上传所使用的 SharePoint 站点 ID（见 [在群聊中发送文件](#sending-files-in-group-chats)）。

<div id="routing-sessions">
  ## 路由与会话
</div>

* 会话键遵循标准智能体格式（参见 [/concepts/session](/zh/concepts/session)）：
  * 私信共享主会话（`agent:<agentId>:<mainKey>`）。
  * 频道/群组消息使用会话 ID（conversationId）：
    * `agent:<agentId>:msteams:channel:<conversationId>`
    * `agent:<agentId>:msteams:group:<conversationId>`

<div id="reply-style-threads-vs-posts">
  ## 回复样式：Threads 与 Posts
</div>

Teams 最近在相同的底层数据模型之上引入了两种频道 UI 样式：

| Style                    | Description                | Recommended `replyStyle` |
| ------------------------ | -------------------------- | ------------------------ |
| **Posts** (classic)      | 消息以卡片形式展示，回复作为子线程显示在下方     | `thread` (默认)            |
| **Threads** (Slack-like) | 消息按时间顺序线性排列，更类似 Slack 的对话流 | `top-level`              |

**问题：** Teams API 不会指明频道实际使用的是哪种 UI 样式。如果你使用了错误的 `replyStyle`：

* 在 Threads 风格频道中使用 `thread` → 回复会以很别扭的嵌套形式出现
* 在 Posts 风格频道中使用 `top-level` → 回复会变成单独的顶级帖子，而不是在线程中显示

**解决方案：** 根据频道的实际配置，为每个频道单独设置 `replyStyle`：

```json
{
  "msteams": {
    "replyStyle": "thread",
    "teams": {
      "19:abc...@thread.tacv2": {
        "channels": {
          "19:xyz...@thread.tacv2": {
            "replyStyle": "top-level"
          }
        }
      }
    }
  }
}
```

<div id="attachments-images">
  ## 附件与图片
</div>

**当前限制：**

* **DMs：** 图片和文件附件通过 Teams 机器人文件 API 进行传输和处理。
* **频道/群组：** 附件存储在 M365 存储（SharePoint/OneDrive）中。webhook 负载仅包含一个 HTML 占位片段（stub），不包含实际文件的二进制内容。下载频道附件**需要 Graph API 权限**。

如果没有 Graph 权限，包含图片的频道消息将只会以纯文本形式接收（bot 无法访问图片内容）。
默认情况下，OpenClaw 只会从 Microsoft/Teams 主机名下载媒体。可通过 `channels.msteams.mediaAllowHosts` 覆盖此行为（使用 `["*"]` 以允许任意主机）。

<div id="sending-files-in-group-chats">
  ## 在群聊中发送文件
</div>

机器人可以使用内置的 FileConsentCard 流程在私聊（DM）中发送文件。然而，**在群聊/频道中发送文件** 需要额外配置：

| 上下文 | 文件发送方式 | 所需配置 |
|---------|-------------------|--------------|
| **DMs** | FileConsentCard → 用户接受 → 机器人上传 | 开箱即用 |
| **群聊/频道** | 上传到 SharePoint → 分享链接 | 需要 `sharePointSiteId` + Graph 权限 |
| **图片（任意上下文）** | 以内联 Base64 编码方式发送 | 开箱即用 |

<div id="why-group-chats-need-sharepoint">
  ### 为什么群聊需要 SharePoint
</div>

机器人没有个人的 OneDrive 网盘（`/me/drive` Graph API 端点对应用身份不可用）。要在群聊或频道中发送文件，机器人会先上传到一个 **SharePoint 站点**，然后创建一个共享链接。

<div id="setup">
  ### 设置
</div>

1. 在 Entra ID (Azure AD) → 应用注册中**添加 Graph API 权限**：
   * `Sites.ReadWrite.All` (Application) - 将文件上传到 SharePoint
   * `Chat.Read.All` (Application) - 可选，用于启用按用户划分的共享链接

2. 为租户**授予管理员同意**。

3. **获取 SharePoint 站点 ID：**
   ```bash
   # 通过 Graph Explorer 或使用携带有效令牌的 curl：
   curl -H "Authorization: Bearer $TOKEN" \
     "https://graph.microsoft.com/v1.0/sites/{hostname}:/{site-path}"

   # 示例：对于位于 "contoso.sharepoint.com/sites/BotFiles" 的站点
   curl -H "Authorization: Bearer $TOKEN" \
     "https://graph.microsoft.com/v1.0/sites/contoso.sharepoint.com:/sites/BotFiles"

   # 响应中包含："id": "contoso.sharepoint.com,guid1,guid2"
   ```

4. **配置 OpenClaw：**
   ```json5
   {
     channels: {
       msteams: {
         // ... 其他配置 ...
         sharePointSiteId: "contoso.sharepoint.com,guid1,guid2"
       }
     }
   }
   ```

<div id="sharing-behavior">
  ### 共享行为
</div>

| 权限 | 共享行为 |
|------------|------------------|
| 仅 `Sites.ReadWrite.All` | 组织范围共享链接（组织内任何人都可访问） |
| `Sites.ReadWrite.All` + `Chat.Read.All` | 按用户的共享链接（仅聊天成员可访问） |

按用户共享更安全，因为只有聊天参与者才能访问该文件。如果缺少 `Chat.Read.All` 权限，机器人会回退到组织范围共享。

<div id="fallback-behavior">
  ### 回退行为
</div>

| 场景 | 结果 |
|----------|--------|
| 群聊 + 文件 + 已配置 `sharePointSiteId` | 上传到 SharePoint，并发送共享链接 |
| 群聊 + 文件 + 无 `sharePointSiteId` | 尝试上传到 OneDrive（可能失败），仅发送文本消息 |
| 私聊 + 文件 | FileConsentCard 流程（无需 SharePoint 也可用） |
| 任意会话场景 + 图片 | 以 Base64 编码的内联形式发送（无需 SharePoint 也可用） |

<div id="files-stored-location">
  ### 文件保存位置
</div>

上传的文件会保存在已配置的 SharePoint 站点默认文档库中的 `/OpenClawShared/` 文件夹内。

<div id="polls-adaptive-cards">
  ## 投票（Adaptive Cards）
</div>

OpenClaw 会以 Adaptive Cards 的形式向 Teams 发送投票（因为没有原生的 Teams 投票 API）。

* CLI: `openclaw message poll --channel msteams --target conversation:<id> ...`
* 投票结果会由 Gateway 记录在 `~/.openclaw/msteams-polls.json` 中。
* Gateway 必须保持在线才能记录投票。
* 目前投票还不会自动发布结果汇总（如有需要请检查该存储文件）。

<div id="adaptive-cards-arbitrary">
  ## Adaptive Cards（任意）
</div>

使用 `message` 工具或 CLI 将任意 Adaptive Card 的 JSON 发送给 Teams 用户或会话。

`card` 参数接受一个 Adaptive Card JSON 对象。当提供 `card` 时，消息文本是可选的。

**Agent 代理工具：**

```json
{
  "action": "send",
  "channel": "msteams",
  "target": "user:<id>",
  "card": {
    "type": "AdaptiveCard",
    "version": "1.5",
    "body": [{"type": "TextBlock", "text": "Hello!"}]
  }
}
```

**CLI：**

```bash
openclaw message send --channel msteams \
  --target "conversation:19:abc...@thread.tacv2" \
  --card '{"type":"AdaptiveCard","version":"1.5","body":[{"type":"TextBlock","text":"Hello!"}]}'
```

有关卡片 schema 和示例，请参阅 [Adaptive Cards 文档](https://adaptivecards.io/)。有关目标格式的详细说明，请参见下文的 [目标格式](#target-formats)。

<div id="target-formats">
  ## 目标格式
</div>

MSTeams 目标使用前缀来区分用户和会话：

| 目标类型      | 格式                               | 示例                                          |
| --------- | -------------------------------- | ------------------------------------------- |
| 用户（按 ID）  | `user:<aad-object-id>`           | `user:40a1a0ed-4ff2-4164-a219-55518990c197` |
| 用户（按显示名称） | `user:<display-name>`            | `user:John Smith`（需要 Graph API）             |
| 组/频道      | `conversation:<conversation-id>` | `conversation:19:abc123...@thread.tacv2`    |
| 组/频道（原始）  | `<conversation-id>`              | `19:abc123...@thread.tacv2`（如果包含 `@thread`） |

**CLI 示例：**

```bash
# Send to a user by ID
openclaw message send --channel msteams --target "user:40a1a0ed-..." --message "Hello"

# 按显示名称向用户发送(触发 Graph API 查找)
openclaw message send --channel msteams --target "user:John Smith" --message "Hello"

# Send to a group chat or channel
openclaw message send --channel msteams --target "conversation:19:abc...@thread.tacv2" --message "Hello"

# Send an Adaptive Card to a conversation
openclaw message send --channel msteams --target "conversation:19:abc...@thread.tacv2" \
  --card '{"type":"AdaptiveCard","version":"1.5","body":[{"type":"TextBlock","text":"Hello"}]}'
```

**Agent 代理工具示例：**

```json
{
  "action": "send",
  "channel": "msteams",
  "target": "user:John Smith",
  "message": "Hello!"
}
```

```json
{
  "action": "send",
  "channel": "msteams",
  "target": "conversation:19:abc...@thread.tacv2",
  "card": {"type": "AdaptiveCard", "version": "1.5", "body": [{"type": "TextBlock", "text": "Hello"}]}
}
```

注意：如果没有 `user:` 前缀，名称将默认解析为群组/团队。按显示名称定位到个人用户时务必使用 `user:`。

<div id="proactive-messaging">
  ## 主动消息
</div>

* 只有在用户与机器人发生过交互**之后**，才可以发送主动消息，因为我们会在那时存储会话引用信息。
* 有关 `dmPolicy` 和 允许列表 访问控制的配置，请参见 `/gateway/configuration`。

<div id="team-and-channel-ids-common-gotcha">
  ## 团队和频道 ID（常见易错点）
</div>

Teams URL 中的 `groupId` 查询参数 **并不是** 用于配置的团队 ID。请改为从 URL 路径中提取 ID：

**团队 URL：**

```
https://teams.microsoft.com/l/team/19%3ABk4j...%40thread.tacv2/conversations?groupId=...
                                    └────────────────────────────┘
                                    Team ID (URL-decode this)
```

**频道 URL：**

```
https://teams.microsoft.com/l/channel/19%3A15bc...%40thread.tacv2/ChannelName?groupId=...
                                      └─────────────────────────┘
                                      频道 ID(对此进行 URL 解码)
```

**在配置中：**

* Team ID = `/team/` 之后的路径段（URL 解码后，例如 `19:Bk4j...@thread.tacv2`）
* Channel ID = `/channel/` 之后的路径段（URL 解码后）
* **忽略** `groupId` 查询参数

<div id="private-channels">
  ## 私有频道
</div>

Bot 在私有频道中的支持是有限的：

| 功能 | 标准频道 | 私有频道 |
|---------|-------------------|------------------|
| Bot 安装 | 支持 | 受限 |
| 实时消息（webhook） | 支持 | 可能无法正常工作 |
| RSC 权限 | 支持 | 行为可能不同 |
| @提及 | 支持 | 如果可以访问该 bot |
| Graph API 历史记录 | 支持 | 支持（需权限） |

**当私有频道不可用时的替代方案：**

1. 使用标准频道与 bot 交互
2. 使用 DM（私信）——用户始终可以直接给 bot 发消息
3. 使用 Graph API 获取历史记录（需要 `ChannelMessage.Read.All`）

<div id="troubleshooting">
  ## 故障排查
</div>

<div id="common-issues">
  ### 常见问题
</div>

* **频道中不显示图片：** 缺少 Graph 权限或管理员同意。重新安装 Teams 应用，并完全退出后重新打开 Teams。
* **频道中没有回复：** 默认需要提及（mention）；请设置 `channels.msteams.requireMention=false`，或按团队/频道分别配置。
* **版本不匹配（Teams 仍显示旧 manifest）：** 先移除再重新添加该应用，并完全退出后重新打开 Teams 以刷新。
* **Webhook 返回 401 Unauthorized：** 在没有 Azure JWT 的情况下手动测试时属预期行为——这表示端点可访问但认证失败。请使用 Azure Web Chat 进行规范测试。

<div id="manifest-upload-errors">
  ### Manifest 上传错误
</div>

* **&quot;Icon file cannot be empty&quot;:** Manifest 引用的图标文件大小为 0 字节。请创建有效的 PNG 图标文件（`outline.png` 为 32×32，`color.png` 为 192×192）。
* **&quot;webApplicationInfo.Id already in use&quot;:** 该应用仍安装在其他团队/聊天中。先找到并卸载该应用，或等待 5–10 分钟让变更传播生效。
* **&quot;Something went wrong&quot; on upload:** 改为通过 https://admin.teams.microsoft.com 上传，打开浏览器开发者工具 DevTools（F12）→ Network 选项卡，并检查响应体以获取实际错误信息。
* **Sideload failing:** 尝试使用 &quot;Upload an app to your org&#39;s app catalog&quot; 而不是 &quot;Upload a custom app&quot; —— 这通常可以绕过旁加载限制。

<div id="rsc-permissions-not-working">
  ### RSC 权限不起作用
</div>

1. 确认 `webApplicationInfo.id` 与你的机器人应用的 App ID 完全一致
2. 重新上传应用，并在团队/聊天中重新安装
3. 检查你们组织的管理员是否已阻止 RSC 权限
4. 确认你使用了正确的 scope：团队使用 `ChannelMessage.Read.Group`，群聊使用 `ChatMessage.Read.Chat`

<div id="references">
  ## 参考资料
</div>

* [Create Azure Bot](https://learn.microsoft.com/en-us/azure/bot-service/bot-service-quickstart-registration) - Azure Bot 设置指南
* [Teams Developer Portal](https://dev.teams.microsoft.com/apps) - 创建/管理 Teams 应用
* [Teams app manifest schema](https://learn.microsoft.com/en-us/microsoftteams/platform/resources/schema/manifest-schema)
* [Receive channel messages with RSC](https://learn.microsoft.com/en-us/microsoftteams/platform/bots/how-to/conversations/channel-messages-with-rsc)
* [RSC permissions reference](https://learn.microsoft.com/en-us/microsoftteams/platform/graph-api/rsc/resource-specific-consent)
* [Teams bot file handling](https://learn.microsoft.com/en-us/microsoftteams/platform/bots/how-to/bots-filesv4)（频道/群组场景需要 Graph）
* [Proactive messaging](https://learn.microsoft.com/en-us/microsoftteams/platform/bots/how-to/conversations/send-proactive-messages)