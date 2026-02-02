---
title: 模型故障切换
summary: "OpenClaw 如何轮换认证配置文件并在模型之间进行故障切换"
read_when:
  - 排查认证配置文件轮换、冷却期或模型故障切换行为
  - 更新认证配置文件或模型的故障切换规则
---

<div id="model-failover">
  # 模型故障切换
</div>

OpenClaw 分两个阶段处理故障：

1. 在当前提供方内进行**认证配置文件轮换（Auth profile rotation）**。
2. **模型回退（Model fallback）**到 `agents.defaults.model.fallbacks` 中的下一个模型。

本文档介绍运行时规则以及作为这些规则依据的数据。

<div id="auth-storage-keys-oauth">
  ## 认证存储（密钥 + OAuth）
</div>

OpenClaw 对 API 密钥和 OAuth 令牌都使用 **认证配置文件（auth profiles）**。

* 机密信息保存在 `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`（历史路径：`~/.openclaw/agent/auth-profiles.json`）。
* 配置项 `auth.profiles` / `auth.order` 仅用于**元数据和路由控制**（不存储任何机密信息）。
* 仅用于导入的旧版 OAuth 文件：`~/.openclaw/credentials/oauth.json`（首次使用时会被导入到 `auth-profiles.json` 中）。

更多详情：[/concepts/oauth](/zh/concepts/oauth)

凭证类型：

* `type: "api_key"` → `{ provider, key }`
* `type: "oauth"` → `{ provider, access, refresh, expires, email? }`（部分提供方还会包含 `projectId`/`enterpriseUrl`）

<div id="profile-ids">
  ## Profile ID
</div>

OAuth 登录会创建彼此独立的 profile，以便多个账户可以共存。

* 默认：当没有可用邮箱时为 `provider:default`。
* 带邮箱的 OAuth：`provider:<email>`（例如 `google-antigravity:user@gmail.com`）。

Profile 位于 `~/.openclaw/agents/<agentId>/agent/auth-profiles.json` 中的 `profiles` 字段下。

<div id="rotation-order">
  ## 轮换顺序
</div>

当某个提供方有多个 profile 时，OpenClaw 会按如下顺序选择：

1. **显式配置**：`auth.order[provider]`（如果已设置）。
2. **已配置的 profile**：按 provider 过滤后的 `auth.profiles`。
3. **已存储的 profile**：该 provider 在 `auth-profiles.json` 中的条目。

如果未配置显式顺序，OpenClaw 会使用轮询（round‑robin）顺序：

* **主排序键：** profile 类型（**OAuth 优先于 API key**）。
* **次排序键：** `usageStats.lastUsed`（在同一类型内，最久未使用的优先）。
* **处于冷却/已禁用的 profile** 会被移到末尾，并按最近到期时间排序。

<div id="session-stickiness-cache-friendly">
  ### 会话粘性（利于缓存）
</div>

OpenClaw 会**为每个会话固定选定的认证配置（auth profile）**，以保持提供方缓存处于预热状态。
它**不会**在每一次请求时进行轮换。被固定的配置会被重复使用，直到：

* 会话被重置（`/new` / `/reset`）
* 一次压缩（compaction）完成（压缩计数递增）
* 该配置处于冷却/已禁用状态

通过 `/model …@<profileId>` 手动选择会为该会话设置一个**用户覆写**，
在新会话开始之前不会自动轮换。

由会话路由器自动固定的配置会被视为一种**偏好**：
它们会被优先尝试，但在出现速率限制/超时时，OpenClaw 可能会轮换到另一个配置。
用户固定的配置则会始终锁定在该配置上；如果它失败且已配置模型回退，
OpenClaw 会切换到下一个模型，而不是切换配置。

<div id="why-oauth-can-look-lost">
  ### 为什么 OAuth 看起来像“失联了”
</div>

如果你针对同一个提供方同时配置了一个 OAuth 配置档和一个 API key 配置档，并且没有固定其中之一，那么轮询（round-robin）机制会在多条消息之间在它们之间切换。要强制使用单一配置档：

* 使用 `auth.order[provider] = ["provider:profileId"]` 进行固定，或
* 通过 `/model …` 为每个会话设置覆盖，并指定配置档覆盖（前提是你的 UI/聊天界面支持该功能）。

<div id="cooldowns">
  ## 冷却期
</div>

当某个 profile 因认证/频率限制错误（或类似频率限制的超时）而失败时，OpenClaw 会将其标记为进入冷却期，并切换到下一个 profile。
格式/无效请求错误（例如 Cloud Code Assist 工具调用 ID 校验失败）也会被视为可触发故障切换，并使用相同的冷却期机制。

冷却期采用指数退避策略：

* 1 分钟
* 5 分钟
* 25 分钟
* 1 小时（封顶）

状态存储在 `auth-profiles.json` 中的 `usageStats` 字段下：

```json
{
  "usageStats": {
    "provider:profile": {
      "lastUsed": 1736160000000,
      "cooldownUntil": 1736160600000,
      "errorCount": 2
    }
  }
}
```

<div id="billing-disables">
  ## 计费停用
</div>

计费/额度失败（例如 “insufficient credits” / “credit balance too low”）会被视为需要进行故障切换的情况，但通常并不是瞬时性问题。OpenClaw 不会只做短暂冷却，而是将该配置文件标记为**已停用**（使用更长的退避间隔），并切换到下一个配置文件/提供方。

状态会存储在 `auth-profiles.json` 中：

```json
{
  "usageStats": {
    "provider:profile": {
      "disabledUntil": 1736178000000,
      "disabledReason": "billing"
    }
  }
}
```

默认值：

* 计费退避从 **5 小时** 开始，每发生一次计费失败退避时间翻倍，最长不超过 **24 小时**。
* 如果某个计费配置 profile 在 **24 小时** 内没有再失败（可配置），其退避计数会被重置。

<div id="model-fallback">
  ## 模型回退
</div>

如果某个提供方的所有配置档案（profile）都失败，OpenClaw 会切换到
`agents.defaults.model.fallbacks` 中的下一个模型。这适用于认证失败、速率限制，以及在轮换耗尽所有配置档案后仍然发生的超时（其他错误不会触发下一层回退）。

当一次运行以模型覆盖配置（通过 hooks 或 CLI）开始时，在尝试所有已配置的回退模型后，回退流程最终仍会停在 `agents.defaults.model.primary`。

<div id="related-config">
  ## 相关配置
</div>

参见 [Gateway 配置](/zh/gateway/configuration) 了解：

* `auth.profiles` / `auth.order`
* `auth.cooldowns.billingBackoffHours` / `auth.cooldowns.billingBackoffHoursByProvider`
* `auth.cooldowns.billingMaxHours` / `auth.cooldowns.failureWindowHours`
* `agents.defaults.model.primary` / `agents.defaults.model.fallbacks`
* `agents.defaults.imageModel` 路由

参见 [模型](/zh/concepts/models) 以了解更完整的模型选择与回退机制概览。