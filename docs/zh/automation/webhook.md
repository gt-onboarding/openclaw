---
title: Webhook
summary: "用于唤醒智能体并执行隔离运行的 Webhook 入口"
read_when:
  - 添加或修改 Webhook 端点时
  - 将外部系统接入 OpenClaw 时
---

<div id="webhooks">
  # Webhooks
</div>

Gateway 可以提供一个轻量的 HTTP webhook 端点，用于处理外部触发事件。

<div id="enable">
  ## 启用
</div>

```json5
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks"
  }
}
```

注意事项：

* 当 `hooks.enabled=true` 时，必须设置 `hooks.token`。
* `hooks.path` 默认为 `/hooks`。

<div id="auth">
  ## 认证
</div>

每个请求都必须包含 hook 令牌。首选使用请求头：

* `Authorization: Bearer <token>`（推荐）
* `x-openclaw-token: <token>`
* `?token=<token>`（已弃用；会记录一条警告日志，并将在未来的主版本中移除）

<div id="endpoints">
  ## 端点
</div>

<div id="post-hookswake">
  ### `POST /hooks/wake`
</div>

请求体：

```json
{ "text": "系统行", "mode": "now" }
```

* `text` **必填**（string）：事件的描述（例如，“New email received”）。
* `mode` 可选（`now` | `next-heartbeat`）：是立即触发一次心跳（默认 `now`），还是等待下一次周期性检查。

效果：

* 为**主**会话入队一个系统事件
* 如果 `mode=now`，则立即触发一次心跳

<div id="post-hooksagent">
  ### `POST /hooks/agent`
</div>

请求体：

```json
{
  "message": "Run this",
  "name": "Email",
  "sessionKey": "hook:email:msg-123",
  "wakeMode": "now",
  "deliver": true,
  "channel": "last",
  "to": "+15551234567",
  "model": "openai/gpt-5.2-mini",
  "thinking": "low",
  "timeoutSeconds": 120
}
```

* `message` **必填** (string)：要由智能体处理的提示词或消息。
* `name` 可选 (string)：钩子的可读名称（例如 &quot;GitHub&quot;），会作为会话摘要中的前缀。
* `sessionKey` 可选 (string)：用于标识智能体会话的键。默认为随机的 `hook:<uuid>`。使用一致的键可以在该钩子上下文中实现多轮对话。
* `wakeMode` 可选 (`now` | `next-heartbeat`)：是立即触发一次心跳（默认 `now`），还是等待下一次周期性检查。
* `deliver` 可选 (boolean)：如果为 `true`，智能体的响应会被发送到消息通道。默认为 `true`。仅为心跳确认的响应会被自动跳过。
* `channel` 可选 (string)：用于投递的消息通道。可选值之一：`last`、`whatsapp`、`telegram`、`discord`、`slack`、`mattermost` (plugin)、`signal`、`imessage`、`msteams`。默认为 `last`。
* `to` 可选 (string)：通道的接收方标识符（例如 WhatsApp/Signal 的电话号码、Telegram 的 chat ID、Discord/Slack/Mattermost (plugin) 的 channel ID、MS Teams 的 conversation ID）。默认为主会话中的最后一个接收方。
* `model` 可选 (string)：模型覆盖（override）（例如 `anthropic/claude-3-5-sonnet` 或别名）。如果有限制，必须在允许的模型列表中。
* `thinking` 可选 (string)：思考级别覆盖（override）（例如 `low`、`medium`、`high`）。
* `timeoutSeconds` 可选 (number)：智能体运行的最长持续时间（秒）。

Effect:

* 运行一次**隔离的**智能体轮次（拥有自己的 session 键）
* 始终向**主**会话发送摘要
* 如果 `wakeMode=now`，则立即触发一次心跳

<div id="post-hooksname-mapped">
  ### `POST /hooks/<name>`（映射）
</div>

自定义 hook 名称通过 `hooks.mappings` 解析（见配置）。一个映射可以将任意
payload 转换为 `wake` 或 `agent` 动作，并可选使用模板或代码转换。

映射选项（摘要）：

* `hooks.presets: ["gmail"]` 启用内置的 Gmail 映射。
* `hooks.mappings` 允许你在配置中定义 `match`、`action` 和模板。
* `hooks.transformsDir` + `transform.module` 会加载一个 JS/TS 模块以实现自定义逻辑。
* 使用 `match.source` 来保留一个通用的接收端点（基于 payload 的路由）。
* TS 转换需要 TS loader（例如 `bun` 或 `tsx`），或者在运行时使用预编译好的 `.js`。
* 在映射上设置 `deliver: true` + `channel`/`to`，将回复路由到一个聊天界面
  （`channel` 默认为 `last`，否则回退到 WhatsApp）。
* `allowUnsafeExternalContent: true` 会对该 hook 禁用外部内容安全封装器
  （危险；仅用于可信的内部来源）。
* `openclaw webhooks gmail setup` 会为 `openclaw webhooks gmail run` 写入 `hooks.gmail` 配置。
  完整的 Gmail watch 流程参见 [Gmail Pub/Sub](/zh/automation/gmail-pubsub)。

<div id="responses">
  ## 响应
</div>

* `/hooks/wake` 返回 `200` 状态码
* `/hooks/agent` 返回 `202` 状态码（已启动异步运行）
* 身份验证失败返回 `401` 状态码
* 请求体无效返回 `400` 状态码
* 请求体过大返回 `413` 状态码

<div id="examples">
  ## 示例
</div>

```bash
curl -X POST http://127.0.0.1:18789/hooks/wake \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"text":"New email received","mode":"now"}'
```

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","wakeMode":"next-heartbeat"}'
```

<div id="use-a-different-model">
  ### 使用不同的模型
</div>

在智能体载荷（或映射）中添加 `model` 字段，以替换本次运行所使用的模型：

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","model":"openai/gpt-5.2-mini"}'
```

如果你强制使用 `agents.defaults.models`，请确保用于覆写的模型也包含在其中。

```bash
curl -X POST http://127.0.0.1:18789/hooks/gmail \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"source":"gmail","messages":[{"from":"Ada","subject":"Hello","snippet":"Hi"}]}'
```

<div id="security">
  ## 安全
</div>

* 将 Hook 端点放在 loopback、tailnet 或受信任的反向代理之后。
* 使用专用的 Hook 令牌；请勿重用 Gateway 身份验证令牌。
* 避免在 Webhook 日志中包含敏感的原始负载内容。
* Hook 负载默认被视为不受信任数据，并会被封装在安全边界内处理。
  如果你必须为某个特定 Hook 禁用这一机制，请在该 Hook 的映射中将 `allowUnsafeExternalContent` 设置为 `true`（危险）。