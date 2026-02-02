---
title: 工具调用 HTTP api
summary: "通过 Gateway 的 HTTP 端点直接调用单个工具"
read_when:
  - 在不运行完整智能体对话轮次的情况下调用工具
  - 构建需要执行工具策略的自动化流程
---

<div id="tools-invoke-http">
  # Tools Invoke (HTTP)
</div>

OpenClaw 的 Gateway 提供了一个用于直接调用单个工具的简单 HTTP 端点。该端点始终启用，但受 Gateway 认证和工具策略的控制。

- `POST /tools/invoke`
- 与 Gateway 相同的端口（WS + HTTP 复用）：`http://<gateway-host>:<port>/tools/invoke`

默认最大请求体大小为 2 MB。

<div id="authentication">
  ## 身份验证
</div>

使用 Gateway 的认证配置。发送一个 Bearer 令牌：

- `Authorization: Bearer <token>`

注意：

- 当 `gateway.auth.mode="token"` 时，使用 `gateway.auth.token`（或 `OPENCLAW_GATEWAY_TOKEN`）。
- 当 `gateway.auth.mode="password"` 时，使用 `gateway.auth.password`（或 `OPENCLAW_GATEWAY_PASSWORD`）。

<div id="request-body">
  ## 请求体
</div>

```json
{
  "tool": "sessions_list",
  "action": "json",
  "args": {},
  "sessionKey": "main",
  "dryRun": false
}
```

Fields:

* `tool` (string, required): 要调用的工具名称。
* `action` (string, optional): 如果工具的 schema 支持 `action` 且在参数负载中省略了它，则会被映射到 args 中。
* `args` (object, optional): 工具特定的参数。
* `sessionKey` (string, optional): 目标会话键。若省略或为 `"main"`，Gateway 会使用配置的主会话键（遵循 `session.mainKey` 与默认智能体，或在 global scope 中使用 `global`）。
* `dryRun` (boolean, optional): 为将来用途预留；当前会被忽略。


<div id="policy-routing-behavior">
  ## 策略与路由行为
</div>

工具可用性会通过与 Gateway 中的 Agent 代理相同的策略链进行过滤：

- `tools.profile` / `tools.byProvider.profile`
- `tools.allow` / `tools.byProvider.allow`
- `agents.<id>.tools.allow` / `agents.<id>.tools.byProvider.allow`
- 组策略（如果会话键映射到某个群组或频道）
- 子智能体策略（当使用子智能体会话键进行调用时）

如果某个工具在策略中不被允许，端点会返回 **404**。

为了帮助组策略解析上下文，你可以选择设置：

- `x-openclaw-message-channel: <channel>`（示例：`slack`、`telegram`）
- `x-openclaw-account-id: <accountId>`（当存在多个账号时）

<div id="responses">
  ## 响应
</div>

- `200` → `{ ok: true, result }`
- `400` → `{ ok: false, error: { type, message } }`（请求无效或工具错误）
- `401` → 未授权访问
- `404` → 工具不可用（未找到或未在允许列表中）
- `405` → 不允许的请求方法

<div id="example">
  ## 示例
</div>

```bash
curl -sS http://127.0.0.1:18789/tools/invoke \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -d '{
    "tool": "sessions_list",
    "action": "json",
    "args": {}
  }'
```
