---
title: Openresponses Gateway
summary: "计划：新增 OpenResponses /v1/responses 端点，并在保持兼容的前提下逐步废弃 chat completions 接口"
owner: "openclaw"
status: "draft"
last_updated: "2026-01-19"
---

<div id="openresponses-gateway-integration-plan">
  # OpenResponses Gateway 集成方案
</div>

<div id="context">
  ## 上下文
</div>

OpenClaw Gateway 目前在 `/v1/chat/completions` 暴露了一个精简的、兼容 OpenAI 的 Chat Completions 端点（参见 [OpenAI Chat Completions](/zh/gateway/openai-http-api)）。

Open Responses 是一个基于 OpenAI Responses API 的开放推理标准。它为智能体工作流而设计，使用基于条目的输入以及语义流式事件。OpenResponses 规范定义的是 `/v1/responses`，而不是 `/v1/chat/completions`。

<div id="goals">
  ## 目标
</div>

* 添加一个遵循 OpenResponses 语义的 `/v1/responses` 端点。
* 继续将 Chat Completions 作为易于禁用并最终移除的兼容层。
* 使用独立、可复用的 schema 来标准化验证和解析。

<div id="non-goals">
  ## 非目标
</div>

* 在第一阶段就实现与 OpenResponses 的全部功能对齐（images、files、hosted tools）。
* 替换内部 Agent 代理执行逻辑或工具编排。
* 在第一阶段更改现有的 `/v1/chat/completions` 行为。

<div id="research-summary">
  ## 研究总结
</div>

来源：OpenResponses OpenAPI、OpenResponses 规范站点，以及 Hugging Face 博客文章。

提取出的要点：

* `POST /v1/responses` 接受 `CreateResponseBody` 字段，例如 `model`、`input`（字符串或
  `ItemParam[]`）、`instructions`、`tools`、`tool_choice`、`stream`、`max_output_tokens` 和
  `max_tool_calls`。
* `ItemParam` 是一个可判别联合类型，包括：
  * 角色为 `system`、`developer`、`user`、`assistant` 的 `message` 条目
  * `function_call` 和 `function_call_output`
  * `reasoning`
  * `item_reference`
* 成功的响应会返回一个 `ResponseResource`，其中包含 `object: "response"`、`status` 和
  `output` 条目。
* 流式输出使用语义事件，例如：
  * `response.created`、`response.in_progress`、`response.completed`、`response.failed`
  * `response.output_item.added`、`response.output_item.done`
  * `response.content_part.added`、`response.content_part.done`
  * `response.output_text.delta`、`response.output_text.done`
* 规范要求：
  * `Content-Type: text/event-stream`
  * `event:` 必须与 JSON 中的 `type` 字段匹配
  * 终止事件必须是字面量 `[DONE]`
* `reasoning` 条目可以包含 `content`、`encrypted_content` 和 `summary`。
* HF 示例在请求中包含 `OpenResponses-Version: latest`（可选请求头）。

<div id="proposed-architecture">
  ## 建议的架构
</div>

* 添加 `src/gateway/open-responses.schema.ts`，其中只包含 Zod schema（不引入 Gateway）。
* 添加 `src/gateway/openresponses-http.ts`（或 `open-responses-http.ts`），用于处理 `/v1/responses`。
* 保持 `src/gateway/openai-http.ts` 不变，作为遗留兼容适配器。
* 添加配置项 `gateway.http.endpoints.responses.enabled`（默认值为 `false`）。
* 保持 `gateway.http.endpoints.chatCompletions.enabled` 独立；允许两个端点分别单独启用或禁用。
* 当启用 Chat Completions 时，在启动时输出警告，提示其为遗留功能。

<div id="deprecation-path-for-chat-completions">
  ## Chat Completions 的弃用迁移路径
</div>

* 保持严格的模块边界：不要在 Responses 和 Chat Completions 之间共享 schema 类型。
* 通过配置将 Chat Completions 设为可选（opt-in），以便在无需修改代码的情况下可以禁用它。
* 一旦 `/v1/responses` 稳定，在文档中将 Chat Completions 标记为遗留功能（legacy）。
* 可选的后续步骤：将 Chat Completions 请求映射到 Responses 处理器，以简化后续的移除过程。

<div id="phase-1-support-subset">
  ## 第 1 阶段支持的子集
</div>

* 接受字符串类型的 `input`，或带有消息角色和 `function_call_output` 的 `ItemParam[]`。
* 将 system 和 developer 消息提取到 `extraSystemPrompt` 中。
* 使用最近的 `user` 或 `function_call_output` 作为智能体运行时的当前消息。
* 对于不支持的内容部分（image/file），以 `invalid_request_error` 拒绝请求。
* 返回包含 `output_text` 内容的单条 assistant 消息。
* 在接入 token 记账逻辑之前，返回各字段数值为 0 的 `usage`。

<div id="validation-strategy-no-sdk">
  ## 验证策略（无 SDK）
</div>

* 为以下受支持的子集实现 Zod schema：
  * `CreateResponseBody`
  * `ItemParam` + 消息内容部分的联合类型
  * `ResponseResource`
  * Gateway 使用的流式事件结构
* 将所有 schema 放在单一的独立模块中，以避免产生偏差并支持后续代码生成。

<div id="streaming-implementation-phase-1">
  ## 流式实现（阶段 1）
</div>

* 同时包含 `event:` 和 `data:` 的 SSE 行。
* 必需的事件顺序（最小可行集）：
  * `response.created`
  * `response.output_item.added`
  * `response.content_part.added`
  * `response.output_text.delta`（按需重复）
  * `response.output_text.done`
  * `response.content_part.done`
  * `response.completed`
  * `[DONE]`

<div id="tests-and-verification-plan">
  ## 测试与验证计划
</div>

* 为 `/v1/responses` 增加 e2e 覆盖：
  * 需要认证（Auth）
  * 非流式响应结构
  * 流式事件顺序与 `[DONE]`
  * 使用 headers 和 `user` 的会话路由
* 保持 `src/gateway/openai-http.e2e.test.ts` 不变。
* 手动：使用 curl 访问 `/v1/responses`，设置 `stream: true`，并验证事件顺序以及最终终止事件
  `[DONE]` 是否正确。

<div id="doc-updates-follow-up">
  ## 文档更新（后续）
</div>

* 新增一个介绍 `/v1/responses` 用法和示例的文档页面。
* 在 `/gateway/openai-http-api` 中添加一则“遗留说明”，并链接到 `/v1/responses`。