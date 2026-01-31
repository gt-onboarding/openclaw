---
title: API 使用成本
summary: "审计哪些内容会花钱、用了哪些密钥，以及如何查看用量"
read_when:
  - 你想了解哪些功能可能会调用付费 API
  - 你需要审计密钥、成本以及用量的呈现方式
  - 你在讲解 /status 或 /usage 的成本报告
---

<div id="api-usage-costs">
  # API 使用与费用
</div>

本文档列出了**可以触发 API 密钥的功能**以及其费用会在哪里体现。重点介绍会产生提供方用量或付费 API 调用的 OpenClaw 功能。

<div id="where-costs-show-up-chat-cli">
  ## 成本显示位置（聊天 + CLI）
</div>

**每个会话的成本快照**

* `/status` 显示当前会话的模型、上下文使用情况，以及上次响应的 tokens。
* 如果模型使用 **API-key 身份验证**，`/status` 还会显示上次回复的**预估成本**。

**每条消息的成本附注**

* `/usage full` 会在每条回复后追加一段使用情况附注，其中包括**预估成本**（仅限使用 API-key 时）。
* `/usage tokens` 只显示 tokens；OAuth 流程不会显示具体的美元成本。

**CLI 使用窗口（提供方配额）**

* `openclaw status --usage` 和 `openclaw channels list` 会显示提供方的**使用窗口**
  （配额快照，而不是按消息的成本）。

有关详细信息和示例，请参见 [Token 使用与成本](/zh/token-use)。

<div id="how-keys-are-discovered">
  ## 密钥是如何被发现的
</div>

OpenClaw 可以从以下位置获取凭证：

* **身份验证配置（Auth profiles）**（按智能体分别存储在 `auth-profiles.json` 中）。
* **环境变量**（例如 `OPENAI_API_KEY`、`BRAVE_API_KEY`、`FIRECRAWL_API_KEY`）。
* **配置**（`models.providers.*.apiKey`、`tools.web.search.*`、`tools.web.fetch.firecrawl.*`、
  `memorySearch.*`、`talk.apiKey`）。
* **技能**（`skills.entries.<name>.apiKey`），这些密钥可能会被导出到技能进程的环境变量中。

<div id="features-that-can-spend-keys">
  ## 会消耗 Key 配额的功能
</div>

<div id="1-core-model-responses-chat-tools">
  ### 1) 核心模型响应（聊天 + 工具）
</div>

每条回复或工具调用都会使用**当前模型提供方**（OpenAI、Anthropic 等）。这是用量和费用的主要来源。

有关定价配置参见 [模型](/zh/providers/models)，有关展示参见 [Token 使用与成本](/zh/token-use)。

<div id="2-media-understanding-audioimagevideo">
  ### 2) 媒体理解（音频 / 图像 / 视频）
</div>

传入媒体可以在生成回复前先进行摘要/转录。这会使用模型 / 提供方的 API。

* 音频：OpenAI / Groq / Deepgram（如果已配置密钥，则会**自动启用**）。
* 图像：OpenAI / Anthropic / Google。
* 视频：Google。

参见 [媒体理解](/zh/nodes/media-understanding)。

<div id="3-memory-embeddings-semantic-search">
  ### 3) 记忆向量嵌入 + 语义搜索
</div>

当配置为远程提供方时，语义记忆搜索会使用 **嵌入 API**：

* `memorySearch.provider = "openai"` → OpenAI 向量嵌入
* `memorySearch.provider = "gemini"` → Gemini 向量嵌入
* 如果本地向量嵌入失败，可以选择回退到 OpenAI

你也可以通过 `memorySearch.provider = "local"` 将其改为本地模式（不使用 API）。

参见 [Memory](/zh/concepts/memory)。

<div id="4-web-search-tool-brave-perplexity-via-openrouter">
  ### 4) Web 搜索工具（Brave / 通过 OpenRouter 的 Perplexity）
</div>

`web_search` 使用 API 密钥，可能产生使用成本：

* **Brave Search API**：`BRAVE_API_KEY` 或 `tools.web.search.apiKey`
* **Perplexity**（通过 OpenRouter）：`PERPLEXITY_API_KEY` 或 `OPENROUTER_API_KEY`

**Brave 免费套餐（额度相当宽松）：**

* **每月 2,000 次请求**
* **每秒 1 次请求**
* **需要信用卡** 进行验证（除非你升级，否则不会扣费）

参见 [Web 工具](/zh/tools/web)。

<div id="5-web-fetch-tool-firecrawl">
  ### 5）Web 抓取工具（Firecrawl）
</div>

当提供 API key 时，`web_fetch` 可以调用 **Firecrawl**：

* `FIRECRAWL_API_KEY` 或 `tools.web.fetch.firecrawl.apiKey`

如果未配置 Firecrawl，该工具会退回到直接抓取 + 可读性提取（不使用付费 API）。

参见 [Web 工具](/zh/tools/web)。

<div id="6-provider-usage-snapshots-statushealth">
  ### 6) 提供方使用情况快照（状态/健康）
</div>

某些状态命令会调用**提供方使用情况端点**，以显示配额时间窗口或认证健康状况。
这些通常是低频调用，但仍然会访问提供方 API：

* `openclaw status --usage`
* `openclaw models status --json`

参见 [Models CLI](/zh/cli/models)。

<div id="7-compaction-safeguard-summarization">
  ### 7) 压缩防护摘要
</div>

压缩防护机制可以使用**当前模型**对会话历史进行摘要，
在执行时会调用提供方的 API。

参见[会话管理与压缩](/zh/reference/session-management-compaction)。

<div id="8-model-scan-probe">
  ### 8）模型扫描 / 探测
</div>

在启用探测时，`openclaw models scan` 可以对 OpenRouter 模型进行探测，并会使用 `OPENROUTER_API_KEY`。

参见 [Models CLI](/zh/cli/models)。

<div id="9-talk-speech">
  ### 9) Talk（语音）
</div>

在配置完成后，Talk 模式可以调用 **ElevenLabs**：

* `ELEVENLABS_API_KEY` 或 `talk.apiKey`

参见 [Talk 模式](/zh/nodes/talk)。

<div id="10-skills-third-party-apis">
  ### 10) 技能（第三方 API）
</div>

技能可以将 `apiKey` 存储在 `skills.entries.<name>.apiKey` 中。如果某个技能使用该密钥访问外部
API，则可能会根据该技能提供方的计费规则产生费用。

参见 [技能](/zh/tools/skills)。