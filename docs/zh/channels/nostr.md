---
title: Nostr
summary: "通过 NIP-04 加密消息实现的 Nostr 私信通道"
read_when:
  - 你希望 OpenClaw 通过 Nostr 接收私信（DM）
  - 你正在设置去中心化消息传递
---

<div id="nostr">
  # Nostr
</div>

**状态：** 可选插件（默认关闭）。

Nostr 是一种用于社交网络的去中心化协议。此通道使 OpenClaw 能够通过 NIP-04 接收和回复加密的私信（DM）。

<div id="install-on-demand">
  ## 按需安装
</div>

<div id="onboarding-recommended">
  ### 上手引导（推荐）
</div>

- 上手向导（`openclaw onboard`）和 `openclaw channels add` 会列出可选的渠道插件。
- 当你选择 Nostr 时，会提示你按需安装该插件。

默认安装行为：

- **开发渠道 + 可用 git checkout：** 使用本地插件路径。
- **稳定版/测试版：** 从 npm 下载。

你始终可以在提示时修改这个选择。

<div id="manual-install">
  ### 手动安装
</div>

```bash
openclaw plugins install @openclaw/nostr
```

使用本地检出（开发流程）：

```bash
openclaw plugins install --link <path-to-openclaw>/extensions/nostr
```

在安装或启用插件后，请重启 Gateway。


<div id="quick-setup">
  ## 快速设置
</div>

1. 生成 Nostr 密钥对（如有需要）：

```bash
# 使用 nak
nak key generate
```

2. 添加到配置文件：

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}"
    }
  }
}
```

3. 导出密钥：

```bash
export NOSTR_PRIVATE_KEY="nsec1..."
```

4. 重启 Gateway。


<div id="configuration-reference">
  ## 配置参考
</div>

| 键 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `privateKey` | string | 必填 | `nsec` 或十六进制格式的私钥 |
| `relays` | string[] | `['wss://relay.damus.io', 'wss://nos.lol']` | 中继 URL（WebSocket） |
| `dmPolicy` | string | `pairing` | 私信（DM）访问策略 |
| `allowFrom` | string[] | `[]` | 允许的发送方公钥 |
| `enabled` | boolean | `true` | 启用/禁用渠道 |
| `name` | string | - | 显示名称 |
| `profile` | object | - | NIP-01 个人资料元数据 |

<div id="profile-metadata">
  ## 个人资料元数据
</div>

个人资料数据会作为 NIP-01 的 `kind:0` 事件发布。你可以在 Control UI 中管理它（Channels -&gt; Nostr -&gt; Profile），或者直接在配置文件中设置。

示例：

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "profile": {
        "name": "openclaw",
        "displayName": "OpenClaw",
        "about": "个人助理私信机器人",
        "picture": "https://example.com/avatar.png",
        "banner": "https://example.com/banner.png",
        "website": "https://example.com",
        "nip05": "openclaw@example.com",
        "lud16": "openclaw@example.com"
      }
    }
  }
}
```

注意：

* 个人资料 URL 必须使用 `https://`。
* 从中继导入时会合并各字段，并保留本地覆盖的设置。


<div id="access-control">
  ## 访问控制
</div>

<div id="dm-policies">
  ### 私信策略
</div>

- **pairing**（默认）：未知发送者会收到配对码。
- **allowlist**：只有 `allowFrom` 中的公钥可以发起私信。
- **open**：公开入站私信（需要 `allowFrom: ["*"]`，表示允许来自任意用户的消息）。
- **disabled**：忽略所有入站私信。

<div id="allowlist-example">
  ### 允许列表示例
</div>

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "dmPolicy": "allowlist",
      "allowFrom": ["npub1abc...", "npub1xyz..."]
    }
  }
}
```


<div id="key-formats">
  ## 密钥格式
</div>

支持的格式：

- **私钥：**`nsec...` 或 64 字符十六进制字符串
- **公钥（`allowFrom`）：**`npub...` 或十六进制字符串

<div id="relays">
  ## 中继
</div>

默认值：`relay.damus.io` 和 `nos.lol`。

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "relays": [
        "wss://relay.damus.io",
        "wss://relay.primal.net",
        "wss://nostr.wine"
      ]
    }
  }
}
```

提示：

* 使用 2–3 个 relay 提供冗余。
* 避免使用过多 relay（会增加延迟并导致消息重复）。
* 使用付费 relay 可以提升可靠性。
* 本地 relay 适合用于测试（`ws://localhost:7777`）。


<div id="protocol-support">
  ## 协议支持
</div>

| NIP | 状态 | 说明 |
| --- | --- | --- |
| NIP-01 | 已支持 | 基本事件格式和个人资料元数据 |
| NIP-04 | 已支持 | 加密私信（`kind:4`） |
| NIP-17 | 计划支持 | Gift-wrapped 私信 |
| NIP-44 | 计划支持 | 版本化加密 |

<div id="testing">
  ## 测试
</div>

<div id="local-relay">
  ### 本地中继节点
</div>

```bash
# 启动 strfry
docker run -p 7777:7777 ghcr.io/hoytech/strfry
```

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "relays": ["ws://localhost:7777"]
    }
  }
}
```


<div id="manual-test">
  ### 手动测试
</div>

1) 从日志中记下机器人的 pubkey（npub）。
2) 打开一个 Nostr 客户端（Damus、Amethyst 等）。
3) 向该机器人的 pubkey 发送私信（DM）。
4) 确认返回的响应。

<div id="troubleshooting">
  ## 故障排查
</div>

<div id="not-receiving-messages">
  ### 没有收到消息
</div>

- 验证私钥是否有效。
- 确保中继 URL 可访问并使用 `wss://`（本地环境可使用 `ws://`）。
- 确认 `enabled` 不是 `false`。
- 检查 Gateway 日志中的中继连接错误。

<div id="not-sending-responses">
  ### 未发送响应
</div>

- 检查中继是否允许写入。
- 检查对外网络连通性。
- 注意中继的速率限制。

<div id="duplicate-responses">
  ### 重复响应
</div>

- 在使用多个中继时属于预期行为。
- 消息会根据事件 ID 进行去重；只有首次投递会触发响应。

<div id="security">
  ## 安全
</div>

- 切勿将私钥提交到代码仓库。
- 使用环境变量来存放密钥。
- 在生产环境的机器人中考虑使用 `allowlist`（允许列表）。

<div id="limitations-mvp">
  ## 限制（MVP）
</div>

- 目前仅支持私信（不支持群聊）。
- 不支持媒体类附件。
- 仅支持 NIP-04（计划支持 NIP-17 gift-wrap）。