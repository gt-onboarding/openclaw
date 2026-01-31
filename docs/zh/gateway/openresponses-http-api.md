---
title: OpenResponses HTTP API
summary: "通过 Gateway 暴露一个兼容 OpenResponses 的 /v1/responses HTTP 端点"
read_when:
  - 集成使用 OpenResponses API 的客户端
  - 你需要基于条目的输入、客户端工具调用或 SSE 事件
---

<div id="openresponses-api-http">
  # OpenResponses API（HTTP）
</div>

OpenClaw 的 Gateway 可以提供兼容 OpenResponses 的 `POST /v1/responses` 端点。

该端点**默认处于禁用状态**。请先在配置中启用它。

- `POST /v1/responses`
- 与 Gateway 使用相同端口（WS + HTTP 复用）：`http://<gateway-host>:<port>/v1/responses`

在底层实现上，请求会作为一次正常的 Gateway 智能体运行过程来执行（与 `openclaw agent` 使用相同代码路径），因此路由、权限和配置行为都与 Gateway 保持一致。

<div id="authentication">
  ## 认证
</div>

使用 Gateway 的认证配置。发送一个 Bearer 令牌：

- `Authorization: Bearer <token>`

注意：

- 当 `gateway.auth.mode="token"` 时，使用 `gateway.auth.token`（或 `OPENCLAW_GATEWAY_TOKEN`）。
- 当 `gateway.auth.mode="password"` 时，使用 `gateway.auth.password`（或 `OPENCLAW_GATEWAY_PASSWORD`）。

<div id="choosing-an-agent">
  ## 选择智能体
</div>

不需要自定义请求头：在 OpenResponses 的 `model` 字段中写入智能体 ID：

- `model: "openclaw:<agentId>"`（示例：`"openclaw:main"`、`"openclaw:beta"`）
- `model: "agent:<agentId>"`（别名）

或者通过请求头指定特定的 OpenClaw Agent 代理：

- `x-openclaw-agent-id: <agentId>`（默认：`main`）

高级用法：

- `x-openclaw-session-key: <sessionKey>` 用于完全控制会话路由。

<div id="enabling-the-endpoint">
  ## 启用端点
</div>

将 `gateway.http.endpoints.responses.enabled` 设置为 `true`：

```json5
{
  gateway: {
    http: {
      endpoints: {
        responses: { enabled: true }
      }
    }
  }
}
```


<div id="disabling-the-endpoint">
  ## 禁用端点
</div>

将 `gateway.http.endpoints.responses.enabled` 设置为 `false`：

```json5
{
  gateway: {
    http: {
      endpoints: {
        responses: { enabled: false }
      }
    }
  }
}
```


<div id="session-behavior">
  ## 会话行为
</div>

默认情况下，该端点是**按请求无状态的**（每次调用都会生成新的会话键）。

如果请求中包含一个 OpenResponses `user` 字符串，Gateway 会基于它派生出一个稳定的会话键，从而使重复调用可以共享同一个智能体会话。

<div id="request-shape-supported">
  ## 请求结构（已支持）
</div>

该请求遵循 OpenResponses API 的基于条目的输入格式。目前支持：

- `input`：字符串或条目对象数组。
- `instructions`：合并到 system prompt 中。
- `tools`：客户端工具定义（函数类工具）。
- `tool_choice`：筛选或强制使用客户端工具。
- `stream`：启用 SSE 流式传输。
- `max_output_tokens`：尽力约束的输出上限（依赖提供方）。
- `user`：稳定的会话路由。

已接受但**当前会被忽略**：

- `max_tool_calls`
- `reasoning`
- `metadata`
- `store`
- `previous_response_id`
- `truncation`

<div id="items-input">
  ## 项（输入）
</div>

<div id="message">
  ### `message`
</div>

角色：`system`、`developer`、`user`、`assistant`。

- `system` 和 `developer` 会附加到系统提示（system prompt）中。
- 最新的 `user` 或 `function_call_output` 条目会成为“当前消息”。
- 更早的 `user` / `assistant` 消息会作为历史上下文被包含进来。

<div id="function_call_output-turn-based-tools">
  ### `function_call_output`（回合制工具）
</div>

将工具执行结果发送回模型：

```json
{
  "type": "function_call_output",
  "call_id": "call_123",
  "output": "{\"temperature\": \"72F\"}"
}
```


<div id="reasoning-and-item_reference">
  ### `reasoning` 和 `item_reference`
</div>

为了保持 schema 兼容性而被接受，但在构建 prompt 时会被忽略。

<div id="tools-client-side-function-tools">
  ## 工具（客户端函数工具）
</div>

通过 `tools: [{ type: "function", function: { name, description?, parameters? } }]` 提供工具。

如果智能体决定调用某个工具，响应会返回一个 `function_call` 输出项。
然后你需要发送一个带有 `function_call_output` 的后续请求以继续该轮对话。

<div id="images-input_image">
  ## 图像（`input_image`）
</div>

支持 base64 编码或 URL 作为数据源：

```json
{
  "type": "input_image",
  "source": { "type": "url", "url": "https://example.com/image.png" }
}
```

当前允许的 MIME 类型：`image/jpeg`、`image/png`、`image/gif`、`image/webp`。
当前最大文件大小：10MB。


<div id="files-input_file">
  ## 文件（`input_file`）
</div>

支持 base64 或 URL 作为来源：

```json
{
  "type": "input_file",
  "source": {
    "type": "base64",
    "media_type": "text/plain",
    "data": "SGVsbG8gV29ybGQh",
    "filename": "hello.txt"
  }
}
```

允许的 MIME 类型（当前）：`text/plain`, `text/markdown`, `text/html`, `text/csv`,
`application/json`, `application/pdf`。

最大大小（当前）：5MB。

当前行为：

* 文件内容会被解码并添加到 **system prompt** 中，而不是用户消息，
  因此是临时的（不会保存在会话历史中）。
* 会对 PDF 进行文本解析。如果只找到很少的文本，则会将前几页光栅化为图片，
  并将这些图片传给模型。

PDF 解析使用适用于 Node 的 `pdfjs-dist` 旧版构建（无 worker）。现代的
PDF.js 构建依赖浏览器 workers/DOM 全局对象，因此在 Gateway 中不会使用。

URL 拉取默认配置：

* `files.allowUrl`: `true`
* `images.allowUrl`: `true`
* 请求会受到防护（DNS 解析检查、私有 IP 阻断、重定向上限、超时控制）。


<div id="file-image-limits-config">
  ## 文件和图片限制（配置）
</div>

可以在 `gateway.http.endpoints.responses` 下调整默认限制：

```json5
{
  gateway: {
    http: {
      endpoints: {
        responses: {
          enabled: true,
          maxBodyBytes: 20000000,
          files: {
            allowUrl: true,
            allowedMimes: ["text/plain", "text/markdown", "text/html", "text/csv", "application/json", "application/pdf"],
            maxBytes: 5242880,
            maxChars: 200000,
            maxRedirects: 3,
            timeoutMs: 10000,
            pdf: {
              maxPages: 4,
              maxPixels: 4000000,
              minTextChars: 200
            }
          },
          images: {
            allowUrl: true,
            allowedMimes: ["image/jpeg", "image/png", "image/gif", "image/webp"],
            maxBytes: 10485760,
            maxRedirects: 3,
            timeoutMs: 10000
          }
        }
      }
    }
  }
}
```

未显式设置时的默认值：

* `maxBodyBytes`: 20MB
* `files.maxBytes`: 5MB
* `files.maxChars`: 200k
* `files.maxRedirects`: 3
* `files.timeoutMs`: 10s
* `files.pdf.maxPages`: 4
* `files.pdf.maxPixels`: 4,000,000
* `files.pdf.minTextChars`: 200
* `images.maxBytes`: 10MB
* `images.maxRedirects`: 3
* `images.timeoutMs`: 10s


<div id="streaming-sse">
  ## 流式传输（SSE）
</div>

将 `stream: true` 设置为接收服务器发送事件（Server-Sent Events，SSE）：

- `Content-Type: text/event-stream`
- 每个事件行的格式为 `event: <type>` 和 `data: <json>`
- 流以 `data: [DONE]` 结束

当前会触发的事件类型包括：

- `response.created`
- `response.in_progress`
- `response.output_item.added`
- `response.content_part.added`
- `response.output_text.delta`
- `response.output_text.done`
- `response.content_part.done`
- `response.output_item.done`
- `response.completed`
- `response.failed`（出错时）

<div id="usage">
  ## 使用情况
</div>

当底层提供方返回 token 用量统计时，`usage` 字段会被填充。

<div id="errors">
  ## 错误
</div>

错误以如下 JSON 对象表示：

```json
{ "error": { "message": "...", "type": "invalid_request_error" } }
```

常见情况：

* `401` 缺失或无效的认证信息
* `400` 无效的请求体
* `405` 使用了错误的 HTTP 方法


<div id="examples">
  ## 示例
</div>

非流式响应：

```bash
curl -sS http://127.0.0.1:18789/v1/responses \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "input": "hi"
  }'
```

流式响应：

```bash
curl -N http://127.0.0.1:18789/v1/responses \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "stream": true,
    "input": "hi"
  }'
```
