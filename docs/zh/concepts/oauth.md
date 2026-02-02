---
title: OAuth
summary: "OpenClaw 中的 OAuth：令牌交换、存储和多账户模式"
read_when:
  - 你想端到端了解 OpenClaw 的 OAuth 流程
  - 你遇到令牌失效 / 注销登录相关问题
  - 你想使用 setup-token 或 OAuth 认证流程
  - 你想要多个账户或基于 profile 的路由
---

<div id="oauth">
  # OAuth
</div>

对于提供 OAuth 的提供方，OpenClaw 支持通过 OAuth 的“订阅授权”（subscription auth）（尤其是 **OpenAI Codex（ChatGPT OAuth）**）。对于 Anthropic 订阅，请使用 **setup-token** 流程。本页将说明：

* 基于 PKCE 的 OAuth **token exchange**（令牌交换）是如何工作的
* 令牌被**存储**在何处（以及原因）
* 如何处理**多个账户**（profile + 按会话覆写）

OpenClaw 也支持自带 OAuth 或 API‑key 流程的**提供方插件**。通过以下方式运行它们：

```bash
openclaw models auth login --provider <id>
```

<div id="the-token-sink-why-it-exists">
  ## 令牌汇（存在的原因）
</div>

OAuth 提供方在登录 / 刷新流程中通常会生成一个**新的刷新令牌（refresh token）**。部分提供方（或 OAuth 客户端）在为同一用户 / 应用签发新的刷新令牌时，会使旧的刷新令牌失效。

典型现象：

* 你同时通过 OpenClaw *以及* Claude Code / Codex CLI 登录 → 之后其中一个会随机被“登出”

为减少这种情况，OpenClaw 将 `auth-profiles.json` 视为一个**令牌汇（token sink）**：

* 运行时只从**单一位置**读取凭据
* 我们可以保留多个配置文件（profile），并以确定性的方式对它们进行路由

<div id="storage-where-tokens-live">
  ## 存储（令牌存放位置）
</div>

机密信息按**每个智能体**单独存储：

* 授权配置（OAuth + API 密钥）：`~/.openclaw/agents/<agentId>/agent/auth-profiles.json`
* 运行时缓存（自动管理；请勿编辑）：`~/.openclaw/agents/<agentId>/agent/auth.json`

旧版的仅导入用文件（仍受支持，但不是主存储方式）：

* `~/.openclaw/credentials/oauth.json`（首次使用时会被导入到 `auth-profiles.json` 中）

上述所有路径同样会遵循 `$OPENCLAW_STATE_DIR`（状态目录覆盖）。完整说明参见：[/gateway/configuration](/zh/gateway/configuration#auth-storage-oauth--api-keys)

<div id="anthropic-setup-token-subscription-auth">
  ## Anthropic setup-token（订阅认证）
</div>

在任意一台机器上运行 `claude setup-token`，然后将其粘贴到 OpenClaw 中：

```bash
openclaw models auth setup-token --provider anthropic
```

如果你在其他地方生成了 token，请手动粘贴：

```bash
openclaw models auth paste-token --provider anthropic
```

验证：

```bash
openclaw models status
```

<div id="oauth-exchange-how-login-works">
  ## OAuth 交换（登录机制的工作原理）
</div>

OpenClaw 的交互式登录流程由 `@mariozechner/pi-ai` 实现，并集成到各类向导和命令中。

<div id="anthropic-claude-promax-setup-token">
  ### Anthropic（Claude Pro/Max）setup-token
</div>

流程概览：

1. 运行 `claude setup-token`
2. 将该 token 粘贴到 OpenClaw 中
3. 存储为基于 token 的认证配置（无刷新令牌）

向导路径为 `openclaw onboard` → 在鉴权方式中选择 `setup-token`（Anthropic）。

<div id="openai-codex-chatgpt-oauth">
  ### OpenAI Codex（ChatGPT OAuth）
</div>

流程概览（PKCE）：

1. 生成 PKCE verifier/challenge 和随机 `state`
2. 打开 `https://auth.openai.com/oauth/authorize?...`
3. 尝试在 `http://127.0.0.1:1455/auth/callback` 捕获回调请求
4. 如果回调端口无法绑定（或者你在远程/无头环境），手动粘贴重定向得到的 URL/code
5. 在 `https://auth.openai.com/oauth/token` 执行令牌交换
6. 从 access token 中提取 `accountId`，并存储 `{ access, refresh, expires, accountId }`

向导路径为 `openclaw onboard` → 认证选项 `openai-codex`。

<div id="refresh-expiry">
  ## 刷新与过期
</div>

Profile 会存储一个 `expires` 时间戳。

在运行时：

* 如果 `expires` 仍未到期 → 使用已存储的访问令牌
* 如果已过期 → 在文件锁下执行刷新，并覆盖已存储的凭据

刷新流程是自动进行的；通常你无需手动管理令牌。

<div id="multiple-accounts-profiles-routing">
  ## 多个账号（配置档案）+ 路由
</div>

两种模式：

<div id="1-preferred-separate-agents">
  ### 1) 首选方案：使用独立的 Agent 代理
</div>

如果你希望“个人”和“工作”之间完全隔离、互不交互，请使用彼此隔离的智能体（独立的会话 + 凭据 + 工作区）：

```bash
openclaw agents add work
openclaw agents add personal
```

然后通过向导为每个智能体配置认证，并将会话路由到对应的智能体。

<div id="2-advanced-multiple-profiles-in-one-agent">
  ### 2) 高级：在单个智能体中使用多个 profile
</div>

`auth-profiles.json` 支持为同一个提供方配置多个 profile ID。

选择要使用的 profile：

* 全局（通过配置顺序 `auth.order`）
* 每个会话单独指定（通过 `/model ...@<profileId>`）

示例（会话级覆盖）：

* `/model Opus@anthropic:work`

如何查看可用的 profile ID：

* `openclaw channels list --json`（显示 `auth[]`）

相关文档：

* [/concepts/model-failover](/zh/concepts/model-failover)（轮换 + 冷却规则）
* [/tools/slash-commands](/zh/tools/slash-commands)（命令入口）