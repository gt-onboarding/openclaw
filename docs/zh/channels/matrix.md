---
title: Matrix
summary: "Matrix 支持状态、功能和配置"
read_when:
  - 处理 Matrix 渠道功能时
---

<div id="matrix-plugin">
  # Matrix（插件）
</div>

Matrix 是一种开放的、去中心化的消息协议。OpenClaw 会以 Matrix **用户**身份连接到任意 homeserver，因此你需要为机器人准备一个 Matrix 账号。登录之后，你可以直接给机器人发私信，也可以把它邀请进房间（Matrix「群组」）。Beeper 也是一个可选的客户端，但它要求启用 E2EE。

状态：通过插件（@vector-im/matrix-bot-sdk）提供支持。支持私信、房间、线程、媒体、表情回应、投票（以文本形式发送 send + poll-start）、位置，以及 E2EE（需启用加密支持）。

<div id="plugin-required">
  ## 必须安装插件
</div>

Matrix 作为插件提供，不包含在核心安装包中。

通过 CLI（npm 注册表）安装：

```bash
openclaw plugins install @openclaw/matrix
```

本地代码副本（从 git 仓库运行时）：

```bash
openclaw plugins install ./extensions/matrix
```

如果你在配置/引导过程中选择了 Matrix，并且检测到存在 git checkout，
OpenClaw 会自动填入本地安装路径。

详情：[插件](/zh/plugin)

<div id="setup">
  ## 设置
</div>

1. 安装 Matrix 插件：
   * 通过 npm 安装：`openclaw plugins install @openclaw/matrix`
   * 通过本地检出安装：`openclaw plugins install ./extensions/matrix`
2. 在某个 homeserver 上创建 Matrix 账户：
   * 在 [https://matrix.org/ecosystem/hosting/](https://matrix.org/ecosystem/hosting/) 浏览托管选项
   * 或自行搭建 homeserver。
3. 为机器人账户获取访问令牌（access token）：

   * 在你的 homeserver 上使用 `curl` 调用 Matrix 登录 API：

   ```bash
   curl --request POST \
     --url https://matrix.example.org/_matrix/client/v3/login \
     --header 'Content-Type: application/json' \
     --data '{
     "type": "m.login.password",
     "identifier": {
       "type": "m.id.user",
       "user": "your-user-name"
     },
     "password": "your-password"
   }'
   ```

   * 将 `matrix.example.org` 替换为你的 homeserver 地址（URL）。
   * 或设置 `channels.matrix.userId` + `channels.matrix.password`：OpenClaw 会调用同一
     登录端点，将访问令牌存储在 `~/.openclaw/credentials/matrix/credentials.json` 中，
     并在下次启动时复用。
4. 配置凭据：
   * 环境变量：`MATRIX_HOMESERVER`、`MATRIX_ACCESS_TOKEN`（或 `MATRIX_USER_ID` + `MATRIX_PASSWORD`）
   * 或配置：`channels.matrix.*`
   * 如果两者都设置，配置优先生效。
   * 使用访问令牌时：用户 ID 会通过 `/whoami` 自动获取。
   * 设置时，`channels.matrix.userId` 应为完整的 Matrix ID（例如：`@bot:example.org`）。
5. 重启 Gateway（或完成引导流程）。
6. 从任意 Matrix 客户端与机器人发起私信（DM），或将其邀请到房间中
   （Element、Beeper 等；见 https://matrix.org/ecosystem/clients/）。Beeper 要求启用端到端加密（E2EE），
   因此需设置 `channels.matrix.encryption: true` 并验证该设备。

最小配置（仅访问令牌，用户 ID 自动获取）：

```json5
{
  channels: {
    matrix: {
      enabled: true,
      homeserver: "https://matrix.example.org",
      accessToken: "syt_***",
      dm: { policy: "pairing" }
    }
  }
}
```

E2EE 配置（已启用端到端加密）：

```json5
{
  channels: {
    matrix: {
      enabled: true,
      homeserver: "https://matrix.example.org",
      accessToken: "syt_***",
      encryption: true,
      dm: { policy: "pairing" }
    }
  }
}
```

<div id="encryption-e2ee">
  ## 加密（E2EE）
</div>

OpenClaw 通过 Rust 加密 SDK **支持**端到端加密。

通过 `channels.matrix.encryption: true` 启用：

* 如果加密模块加载成功，加密房间会被自动解密。
* 向加密房间发送消息时，出站媒体会被加密。
* 在首次连接时，OpenClaw 会向你的其他会话请求设备验证。
* 在另一款 Matrix 客户端（Element 等）中验证该设备，以启用密钥共享。
* 如果无法加载加密模块，则会禁用 E2EE，加密房间将无法解密；
  OpenClaw 会记录一条警告日志。
* 如果你看到缺少加密模块的错误（例如 `@matrix-org/matrix-sdk-crypto-nodejs-*`），
  请允许 `@matrix-org/matrix-sdk-crypto-nodejs` 的构建脚本并运行
  `pnpm rebuild @matrix-org/matrix-sdk-crypto-nodejs`，或通过
  `node node_modules/@matrix-org/matrix-sdk-crypto-nodejs/download-lib.js` 获取二进制文件。

加密状态按账号 + 访问令牌存储在
`~/.openclaw/matrix/accounts/<account>/<homeserver>__<user>/<token-hash>/crypto/`
（SQLite 数据库）中。同步状态保存在与其同一目录下的 `bot-storage.json` 中。
如果访问令牌（设备）发生变化，将创建一个新的存储目录，机器人必须针对加密房间重新完成验证。

**设备验证：**
启用 E2EE 时，机器人在启动时会向你的其他会话请求验证。
打开 Element（或其他客户端）并批准验证请求以建立信任关系。
验证完成后，机器人就可以解密加密房间中的消息。

<div id="routing-model">
  ## 路由模型
</div>

* 回复始终发回到 Matrix。
* 私聊共享该智能体的主会话；房间对应到群组会话。

<div id="access-control-dms">
  ## 访问控制（私信）
</div>

* 默认：`channels.matrix.dm.policy = "pairing"`。未知发送方会收到一个配对码。
* 通过以下命令批准：
  * `openclaw pairing list matrix`
  * `openclaw pairing approve matrix <CODE>`
* 公开私信：`channels.matrix.dm.policy="open"`（policy 取值 `open` 表示允许从任意用户不受限制地接收消息）加上 `channels.matrix.dm.allowFrom=["*"]`。
* `channels.matrix.dm.allowFrom` 可以是用户 ID 或显示名称。当目录搜索可用时，向导会将显示名称解析为用户 ID。

<div id="rooms-groups">
  ## 房间（群组）
</div>

* 默认：`channels.matrix.groupPolicy = "allowlist"`（基于 @ 提及的门控）。在未显式设置时，使用 `channels.defaults.groupPolicy` 覆盖该默认值。
* 使用 `channels.matrix.groups` 将房间加入允许列表（房间 ID、别名或名称）：

```json5
{
  channels: {
    matrix: {
      groupPolicy: "allowlist",
      groups: {
        "!roomId:example.org": { allow: true },
        "#alias:example.org": { allow: true }
      },
      groupAllowFrom: ["@owner:example.org"]
    }
  }
}
```

* `requireMention: false` 会在该房间启用自动回复。
* `groups."*"` 可为各房间的 @ 提及门控行为设置默认值。
* `groupAllowFrom` 用于限制哪些发送者可以在房间内触发机器人（可选）。
* 按房间配置的 `users` 允许列表可以在特定房间内进一步限制发送者。
* 配置向导会提示你填写房间允许列表（房间 ID、别名或名称），并在可能时解析名称。
* 启动时，OpenClaw 会将允许列表中的房间/用户名称解析为 ID 并记录映射；解析失败的条目将按原样保留。
* 默认会自动加入被邀请的房间；可通过 `channels.matrix.autoJoin` 和 `channels.matrix.autoJoinAllowlist` 控制。
* 若要**不允许任何房间**，将 `channels.matrix.groupPolicy` 设为 `"disabled"`（或保持允许列表为空）。
* 旧版配置键：`channels.matrix.rooms`（结构与 `groups` 相同）。

<div id="threads">
  ## 线程
</div>

* 支持线程式回复。
* `channels.matrix.threadReplies` 控制回复是否保留在线程中：
  * `off`、`inbound`（默认）、`always`
* `channels.matrix.replyToMode` 控制在不使用线程回复时的 reply-to 元数据行为：
  * `off`（默认）、`first`、`all`

<div id="capabilities">
  ## 功能
</div>

| 功能 | 状态 |
|---------|--------|
| 私聊 | ✅ 已支持 |
| 房间 | ✅ 已支持 |
| 线程 | ✅ 已支持 |
| 媒体 | ✅ 已支持 |
| 端到端加密（E2EE） | ✅ 已支持（需要 crypto 模块） |
| 表情回应 | ✅ 已支持（通过工具发送/read） |
| 投票 | ✅ 支持发送；收到的投票发起会被转换为文本（忽略响应/结束） |
| 位置 | ✅ 已支持（geo URI；忽略高度） |
| 原生命令 | ✅ 已支持 |

<div id="configuration-reference-matrix">
  ## 配置参考（Matrix）
</div>

完整配置： [Configuration](/zh/gateway/configuration)

提供方配置项：

* `channels.matrix.enabled`: 启用/禁用频道启动。
* `channels.matrix.homeserver`: 主服务器（homeserver）URL。
* `channels.matrix.userId`: Matrix 用户 ID（与 access token 同用时可选）。
* `channels.matrix.accessToken`: access token。
* `channels.matrix.password`: 登录密码（token 会被存储）。
* `channels.matrix.deviceName`: 设备显示名称。
* `channels.matrix.encryption`: 启用端到端加密（默认：false）。
* `channels.matrix.initialSyncLimit`: 初始同步上限。
* `channels.matrix.threadReplies`: `off | inbound | always`（默认：inbound）。
* `channels.matrix.textChunkLimit`: 出站文本分块大小（字符数）。
* `channels.matrix.chunkMode`: `length`（默认）或 `newline`，在按长度分块前先按空行（段落边界）拆分。
* `channels.matrix.dm.policy`: `pairing | allowlist | open | disabled`（默认：pairing）。
* `channels.matrix.dm.allowFrom`: DM 允许列表（用户 ID 或显示名称）。`open` 需要 `"*"`。向导会在可能时将名称解析为 ID。
* `channels.matrix.groupPolicy`: `allowlist | open | disabled`（默认：allowlist）。
* `channels.matrix.groupAllowFrom`: 群消息允许列表中的发送方。
* `channels.matrix.allowlistOnly`: 对私信和房间强制应用允许列表规则。
* `channels.matrix.groups`: 群组允许列表 + 每个房间的设置映射。
* `channels.matrix.rooms`: 旧版群组允许列表/配置。
* `channels.matrix.replyToMode`: 线程/标签的回复模式。
* `channels.matrix.mediaMaxMb`: 入站/出站媒体大小上限（MB）。
* `channels.matrix.autoJoin`: 邀请处理策略（`always | allowlist | off`，默认：always）。
* `channels.matrix.autoJoinAllowlist`: 允许自动加入的房间 ID/别名。
* `channels.matrix.actions`: 各类操作的工具权限控制（reactions/messages/pins/memberInfo/channelInfo）。