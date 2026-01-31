---
title: Gmail Pub/Sub
summary: "通过 gogcli 将 Gmail Pub/Sub 推送接入 OpenClaw webhooks"
read_when:
  - 将 Gmail 收件箱触发事件接入 OpenClaw
  - 为智能体唤醒配置 Pub/Sub 推送
---

<div id="gmail-pubsub-openclaw">
  # Gmail Pub/Sub -&gt; OpenClaw
</div>

目标流程：Gmail 监控（watch）-&gt; Pub/Sub 推送（push）-&gt; `gog gmail watch serve` -&gt; OpenClaw webhook。

<div id="prereqs">
  ## 前置条件
</div>

* 已安装并登录 `gcloud`（[安装指南](https://docs.cloud.google.com/sdk/docs/install-sdk)）。
* 已安装并为对应 Gmail 帐号完成授权的 `gog`（gogcli）（[gogcli.sh](https://gogcli.sh/)）。
* 已启用 OpenClaw hooks（参见 [Webhooks](/zh/automation/webhook)）。
* 已登录 `tailscale`（[tailscale.com](https://tailscale.com/)）。受支持的推荐配置使用 Tailscale Funnel 作为公网 HTTPS 端点。
  其他隧道服务也可以使用，但属于自行搭建/不受支持方案，并且需要手动接入配置。
  目前我们仅支持 Tailscale。

示例 hook 配置（启用 Gmail 预设映射）：

```json5
{
  hooks: {
    enabled: true,
    token: "OPENCLAW_HOOK_TOKEN",
    path: "/hooks",
    presets: ["gmail"]
  }
}
```

要将 Gmail 摘要发送到某个聊天界面，你需要覆盖预设，改为使用一个映射，
在其中设置 `deliver`，以及可选的 `channel`/`to` 字段：

```json5
{
  hooks: {
    enabled: true,
    token: "OPENCLAW_HOOK_TOKEN",
    presets: ["gmail"],
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
        wakeMode: "now",
        name: "Gmail",
        sessionKey: "hook:gmail:{{messages[0].id}}",
        messageTemplate:
          "New email from {{messages[0].from}}\nSubject: {{messages[0].subject}}\n{{messages[0].snippet}}\n{{messages[0].body}}",
        model: "openai/gpt-5.2-mini",
        deliver: true,
        channel: "last"
        // to: "+15551234567"
      }
    ]
  }
}
```

如果你想使用固定的渠道，同时设置 `channel` 和 `to`。否则，`channel: "last"`
会使用上一次的投递路径（必要时回退到 WhatsApp）。

要在 Gmail 运行中强制使用更便宜的模型，在映射中设置 `model`
（`provider/model` 或别名）。如果你强制设置了 `agents.defaults.models`，也要在那里包含它。

要为 Gmail 钩子单独设置默认模型和思考级别，在你的配置中添加
`hooks.gmail.model` / `hooks.gmail.thinking`：

```json5
{
  hooks: {
    gmail: {
      model: "openrouter/meta-llama/llama-3.3-70b-instruct:free",
      thinking: "off"
    }
  }
}
```

注意：

* 映射中按 hook 设置的 `model`/`thinking` 仍然会覆盖这些默认值。
* 回退顺序：`hooks.gmail.model` → `agents.defaults.model.fallbacks` → 主模型（由认证/速率限制/超时决定）。
* 如果设置了 `agents.defaults.models`，则 Gmail 模型必须在允许列表中。
* 默认情况下，Gmail hook 内容会被包裹在外部内容（external-content）安全边界内。
  如需禁用（危险），请设置 `hooks.gmail.allowUnsafeExternalContent: true`。

若需进一步自定义载荷处理，请在 `hooks.mappings` 中添加映射，或在 `hooks.transformsDir` 下添加 JS/TS 转换模块（参见 [Webhooks](/zh/automation/webhook)）。

<div id="wizard-recommended">
  ## 向导（推荐）
</div>

使用 OpenClaw 助手完成整体配置（在 macOS 上会通过 brew 安装依赖）：

```bash
openclaw webhooks gmail setup \
  --account openclaw@gmail.com
```

默认行为：

* 使用 Tailscale Funnel 作为公共推送端点。
* 为 `openclaw webhooks gmail run` 写入 `hooks.gmail` 配置。
* 启用 Gmail hook 预设配置（`hooks.presets: ["gmail"]`）。

路径说明：当启用 `tailscale.mode` 时，OpenClaw 会自动将
`hooks.gmail.serve.path` 设为 `/`，并将公共路径保留在
`hooks.gmail.tailscale.path`（默认 `/gmail-pubsub`），因为 Tailscale
在代理前会去掉 set-path 设置的路径前缀。
如果你需要后端接收到带前缀的路径，请将
`hooks.gmail.tailscale.target`（或 `--tailscale-target`）设置为完整 URL，例如
`http://127.0.0.1:8788/gmail-pubsub`，并与 `hooks.gmail.serve.path` 保持一致。

想要自定义端点？使用 `--push-endpoint <url>` 或 `--tailscale off`。

平台说明：在 macOS 上，向导会通过 Homebrew 安装 `gcloud`、`gogcli` 和 `tailscale`；
在 Linux 上需要先手动安装它们。

Gateway 自动启动（推荐）：

* 当 `hooks.enabled=true` 且已设置 `hooks.gmail.account` 时，Gateway 在启动时会
  运行 `gog gmail watch serve` 并自动续期该 watch。
* 设置 `OPENCLAW_SKIP_GMAIL_WATCHER=1` 可禁用此行为（如果你自己运行该守护进程时很有用）。
* 不要同时运行手动守护进程，否则你会遇到
  `listen tcp 127.0.0.1:8788: bind: address already in use` 错误。

手动守护进程（启动 `gog gmail watch serve` + 自动续期）：

```bash
openclaw webhooks gmail run
```

<div id="one-time-setup">
  ## 一次性设置
</div>

1. 选择 `gog` 所使用的 **OAuth 客户端所属的** GCP 项目。

```bash
gcloud auth login
gcloud config set project <project-id>
```

注意：Gmail watch 要求 Pub/Sub 主题与 OAuth 客户端位于同一项目中。

2. 启用 API：

```bash
gcloud services enable gmail.googleapis.com pubsub.googleapis.com
```

3. 创建主题：

```bash
gcloud pubsub topics create gog-gmail-watch
```

4. 允许 Gmail 进行推送发布：

```bash
gcloud pubsub topics add-iam-policy-binding gog-gmail-watch \
  --member=serviceAccount:gmail-api-push@system.gserviceaccount.com \
  --role=roles/pubsub.publisher
```

<div id="start-the-watch">
  ## 启动监听
</div>

```bash
gog gmail watch start \
  --account openclaw@gmail.com \
  --label INBOX \
  --topic projects/<project-id>/topics/gog-gmail-watch
```

从输出中保存好 `history_id`（便于调试）。

<div id="run-the-push-handler">
  ## 运行推送处理程序
</div>

本地示例（共享令牌授权）：

```bash
gog gmail watch serve \
  --account openclaw@gmail.com \
  --bind 127.0.0.1 \
  --port 8788 \
  --path /gmail-pubsub \
  --token <shared> \
  --hook-url http://127.0.0.1:18789/hooks/gmail \
  --hook-token OPENCLAW_HOOK_TOKEN \
  --include-body \
  --max-bytes 20000
```

备注：

* `--token` 用于保护推送端点（`x-gog-token` 或 `?token=`）。
* `--hook-url` 指向 OpenClaw 的 `/hooks/gmail`（已映射；隔离运行，并将摘要发送到主会话）。
* `--include-body` 和 `--max-bytes` 用于控制发送到 OpenClaw 的正文片段。

推荐：使用 `openclaw webhooks gmail run` 封装相同流程，并自动续订 watch 订阅。

<div id="expose-the-handler-advanced-unsupported">
  ## 暴露处理程序（高级用法，不受支持）
</div>

如果你需要使用非 Tailscale 的隧道，请手动配置并在推送订阅中使用该公共 URL（不受支持，无任何防护机制）：

```bash
cloudflared tunnel --url http://127.0.0.1:8788 --no-autoupdate
```

将生成的 URL 用作推送端点：

```bash
gcloud pubsub subscriptions create gog-gmail-watch-push \
  --topic gog-gmail-watch \
  --push-endpoint "https://<public-url>/gmail-pubsub?token=<shared>"
```

生产环境：使用稳定的 HTTPS 端点并配置 Pub/Sub OIDC JWT，然后运行：

```bash
gog gmail watch serve --verify-oidc --oidc-email <svc@...>
```

<div id="test">
  ## 测试
</div>

向受监控的收件箱发送一条消息：

```bash
gog gmail send \
  --account openclaw@gmail.com \
  --to openclaw@gmail.com \
  --subject "watch test" \
  --body "ping"
```

检查 Watch 状态和历史记录：

```bash
gog gmail watch status --account openclaw@gmail.com
gog gmail history --account openclaw@gmail.com --since <historyId>
```

<div id="troubleshooting">
  ## 故障排除
</div>

* `Invalid topicName`：项目不一致（topic 不在该 OAuth 客户端所属的项目中）。
* `User not authorized`：该 topic 缺少 `roles/pubsub.publisher` 角色。
* 空消息：Gmail push 仅提供 `historyId`；请通过 `gog gmail history` 获取。

<div id="cleanup">
  ## 清理
</div>

```bash
gog gmail watch stop --account openclaw@gmail.com
gcloud pubsub subscriptions delete gog-gmail-watch-push
gcloud pubsub topics delete gog-gmail-watch
```
