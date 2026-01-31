---
title: OpenAI HTTP API
summary: "通过 Gateway 暴露一个兼容 OpenAI 的 /v1/chat/completions HTTP 端点"
read_when:
  - 当你需要集成依赖 OpenAI Chat Completions 的工具时
---

<div id="openai-chat-completions-http">
  # OpenAI 聊天补全（HTTP）
</div>

OpenClaw 的 Gateway 可以提供一个轻量级的、兼容 OpenAI 的 Chat Completions 端点。

此端点**默认是禁用的**。请先在配置中将其启用。

- `POST /v1/chat/completions`
- 与 Gateway 使用相同端口（WS + HTTP 多路复用）：`http://<gateway-host>:<port>/v1/chat/completions`

在底层实现上，请求会作为一次普通的 Gateway 智能体运行来执行（与 `openclaw agent` 使用相同的代码路径），因此路由 / 权限 / 配置会与你的 Gateway 保持一致。

<div id="authentication">
  ## 认证
</div>

使用 Gateway 的认证配置。发送 Bearer 令牌：

- `Authorization: Bearer <token>`

注意：

- 当 `gateway.auth.mode="token"` 时，使用 `gateway.auth.token`（或 `OPENCLAW_GATEWAY_TOKEN`）。
- 当 `gateway.auth.mode="password"` 时，使用 `gateway.auth.password`（或 `OPENCLAW_GATEWAY_PASSWORD`）。

<div id="choosing-an-agent">
  ## 选择智能体
</div>

不需要自定义请求头：在 OpenAI 的 `model` 字段中写入智能体 ID：

- `model: "openclaw:<agentId>"`（示例：`"openclaw:main"`、`"openclaw:beta"`）
- `model: "agent:<agentId>"`（别名）

或者通过请求头指定特定的 OpenClaw 智能体：

- `x-openclaw-agent-id: <agentId>`（默认：`main`）

高级用法：

- `x-openclaw-session-key: <sessionKey>` 用于完全控制会话路由。

<div id="enabling-the-endpoint">
  ## 启用端点
</div>

将 `gateway.http.endpoints.chatCompletions.enabled` 设置为 `true`：

```json5
{
  gateway: {
    http: {
      endpoints: {
        chatCompletions: { enabled: true }
      }
    }
  }
}
```


<div id="disabling-the-endpoint">
  ## 禁用端点
</div>

将 `gateway.http.endpoints.chatCompletions.enabled` 设置为 `false`：

```json5
{
  gateway: {
    http: {
      endpoints: {
        chatCompletions: { enabled: false }
      }
    }
  }
}
```


<div id="session-behavior">
  ## 会话行为
</div>

默认情况下，该端点对每个请求都是**无状态**的（每次调用都会生成一个新的会话键）。

如果请求中包含 OpenAI 的 `user` 字符串，Gateway 会基于它派生出一个稳定的会话键，从而使重复调用可以共享同一个智能体会话。

<div id="streaming-sse">
  ## 流式传输（SSE）
</div>

将 `stream` 设置为 `true` 以通过 Server-Sent Events（SSE）接收流式事件：

- `Content-Type: text/event-stream`
- 每个事件行形如 `data: <json>`
- 流以 `data: [DONE]` 结束

<div id="examples">
  ## 示例
</div>

非流式：

```bash
curl -sS http://127.0.0.1:18789/v1/chat/completions \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "messages": [{"role":"user","content":"hi"}]
  }'
```

流式响应：

```bash
curl -N http://127.0.0.1:18789/v1/chat/completions \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "stream": true,
    "messages": [{"role":"user","content":"hi"}]
  }'
```
