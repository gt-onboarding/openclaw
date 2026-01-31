---
title: Web
summary: "网页搜索与抓取工具（Brave Search API、Perplexity direct/OpenRouter）"
read_when:
  - 你想启用 web_search 或 web_fetch
  - 你需要配置 Brave Search API 密钥
  - 你想使用 Perplexity Sonar 进行网页搜索
---

<div id="web-tools">
  # Web 工具
</div>

OpenClaw 内置了两个轻量级 Web 工具：

* `web_search` — 通过 Brave Search API（默认）或 Perplexity Sonar（直接或经由 OpenRouter）进行网页搜索。
* `web_fetch` — 执行 HTTP 请求并进行可读内容抽取（HTML → markdown/文本）。

这两个**不是**浏览器自动化工具。对于严重依赖 JS 的网站或需要登录的站点，请使用
[Browser 工具](/zh/tools/browser)。

<div id="how-it-works">
  ## 工作原理
</div>

* `web_search` 调用你已配置的提供方并返回结果。
  * **Brave**（默认）：返回结构化结果（标题、URL、摘要）。
  * **Perplexity**：返回由 AI 综合生成、并附带实时网页搜索引用的回答。
* 结果会按查询缓存 15 分钟（可配置）。
* `web_fetch` 执行普通的 HTTP GET 并提取可读内容
  （HTML → markdown/文本）。它**不会**执行 JavaScript。
* `web_fetch` 默认启用（除非明确禁用）。

<div id="choosing-a-search-provider">
  ## 选择搜索提供方
</div>

| 提供方            | 优点                  | 缺点                              | API Key                                     |
| -------------- | ------------------- | ------------------------------- | ------------------------------------------- |
| **Brave**（默认）  | 速度快、结果结构化、有免费套餐     | 传统搜索结果                          | `BRAVE_API_KEY`                             |
| **Perplexity** | AI 综合生成的回答、带引用、实时更新 | 需要 Perplexity 或 OpenRouter 访问权限 | `OPENROUTER_API_KEY` 或 `PERPLEXITY_API_KEY` |

参见 [Brave Search 设置](/zh/brave-search) 和 [Perplexity Sonar](/zh/perplexity) 以获取各提供方的详细说明。

在配置中设置提供方：

```json5
{
  tools: {
    web: {
      search: {
        provider: "brave"  // 或者 "perplexity"
      }
    }
  }
}
```

示例：切换到 Perplexity Sonar（直接 api 接入）：

```json5
{
  tools: {
    web: {
      search: {
        provider: "perplexity",
        perplexity: {
          apiKey: "pplx-...",
          baseUrl: "https://api.perplexity.ai",
          model: "perplexity/sonar-pro"
        }
      }
    }
  }
}
```

<div id="getting-a-brave-api-key">
  ## 获取 Brave API 密钥
</div>

1. 在 https://brave.com/search/api/ 创建一个 Brave Search API 账号
2. 在控制台中选择 **Data for Search** 方案（不是 “Data for AI”），并生成一个 API 密钥。
3. 运行 `openclaw configure --section web` 将密钥存储到配置中（推荐），或者在环境变量中设置 `BRAVE_API_KEY`。

Brave 提供免费配额和付费方案；请在 Brave API 门户查看当前的额度限制和价格。

<div id="where-to-set-the-key-recommended">
  ### 在哪里设置密钥（推荐）
</div>

**推荐：** 运行 `openclaw configure --section web`。它会将密钥存储在
`~/.openclaw/openclaw.json` 的 `tools.web.search.apiKey` 字段下。

**环境变量方式（可选）：** 在 Gateway 进程的环境中设置 `BRAVE_API_KEY`。对于 Gateway 安装，把它放在 `~/.openclaw/.env`（或你的
服务运行环境）中。参见 [环境变量 Env vars](/zh/help/faq#how-does-openclaw-load-environment-variables)。

<div id="using-perplexity-direct-or-via-openrouter">
  ## 使用 Perplexity（直接使用或通过 OpenRouter）
</div>

Perplexity 的 Sonar 模型内置网页搜索功能，并会返回带引用的 AI 综合答案。你可以通过 OpenRouter 使用这些模型（无需信用卡——支持加密货币/预付费）。

<div id="getting-an-openrouter-api-key">
  ### 获取 OpenRouter API 密钥
</div>

1. 在 https://openrouter.ai/ 上注册一个账户
2. 为账户充值（支持加密货币、预付卡或信用卡）
3. 在账户设置中生成一个 API 密钥

<div id="setting-up-perplexity-search">
  ### 配置 Perplexity 搜索
</div>

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        provider: "perplexity",
        perplexity: {
          // API 密钥(如果已设置 OPENROUTER_API_KEY 或 PERPLEXITY_API_KEY 则为可选)
          apiKey: "sk-or-v1-...",
          // Base URL (key-aware default if omitted)
          baseUrl: "https://openrouter.ai/api/v1",
          // Model (defaults to perplexity/sonar-pro)
          model: "perplexity/sonar-pro"
        }
      }
    }
  }
}
```

**环境变量替代方案：**在 Gateway 的环境中设置 `OPENROUTER_API_KEY` 或 `PERPLEXITY_API_KEY`。对于 Gateway 安装，将其写入 `~/.openclaw/.env`。

如果没有设置 base URL，OpenClaw 会根据 API 密钥的来源选择一个默认值：

* `PERPLEXITY_API_KEY` 或 `pplx-...` → `https://api.perplexity.ai`
* `OPENROUTER_API_KEY` 或 `sk-or-...` → `https://openrouter.ai/api/v1`
* 未知密钥格式 → OpenRouter（安全的回退方案）

<div id="available-perplexity-models">
  ### 可用的 Perplexity 模型
</div>

| 模型 | 说明 | 最佳用途 |
|-------|-------------|----------|
| `perplexity/sonar` | 通过网页搜索进行快速问答 | 快速查询 |
| `perplexity/sonar-pro` (默认) | 结合网页搜索的多步推理 | 复杂问题 |
| `perplexity/sonar-reasoning-pro` | 思维链（Chain-of-thought）分析 | 深度研究 |

<div id="web_search">
  ## web_search
</div>

使用已配置的提供方进行网页搜索。

<div id="requirements">
  ### 前提条件
</div>

* `tools.web.search.enabled` 不得为 `false`（默认：启用）
* 你所选提供方的 API 密钥：
  * **Brave**：`BRAVE_API_KEY` 或 `tools.web.search.apiKey`
  * **Perplexity**：`OPENROUTER_API_KEY`、`PERPLEXITY_API_KEY` 或 `tools.web.search.perplexity.apiKey`

<div id="config">
  ### 配置
</div>

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        apiKey: "BRAVE_API_KEY_HERE", // 如果已设置 BRAVE_API_KEY 则为可选
        maxResults: 5,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15
      }
    }
  }
}
```

<div id="tool-parameters">
  ### 工具参数
</div>

* `query`（必填）
* `count`（1–10；默认值来自配置）
* `country`（可选）：用于区域相关结果的 2 个字母国家/地区代码（例如，“DE”、“US”、“ALL”）。如果省略，Brave 会使用其默认区域。
* `search_lang`（可选）：搜索结果的 ISO 语言代码（例如，“de”、“en”、“fr”）
* `ui_lang`（可选）：UI 元素的 ISO 语言代码
* `freshness`（可选，仅 Brave）：按发现时间进行过滤（`pd`、`pw`、`pm`、`py` 或 `YYYY-MM-DDtoYYYY-MM-DD`）

**示例：**

```javascript
// German-specific search
await web_search({
  query: "TV online schauen",
  count: 10,
  country: "DE",
  search_lang: "de"
});

// 法语搜索,使用法语 UI
await web_search({
  query: "actualités",
  country: "FR",
  search_lang: "fr",
  ui_lang: "fr"
});

// Recent results (past week)
await web_search({
  query: "TMBG interview",
  freshness: "pw"
});
```

<div id="web_fetch">
  ## web_fetch
</div>

获取 URL 并提取可读内容。

### 要求

* `tools.web.fetch.enabled` 不能为 `false`（默认：启用）
* 可选的 Firecrawl 备用方案：设置 `tools.web.fetch.firecrawl.apiKey` 或 `FIRECRAWL_API_KEY`。

<div id="config">
  ### 配置
</div>

```json5
{
  tools: {
    web: {
      fetch: {
        enabled: true,
        maxChars: 50000,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
        maxRedirects: 3,
        userAgent: "Mozilla/5.0 (Macintosh; Intel Mac OS X 14_7_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36",
        readability: true,
        firecrawl: {
          enabled: true,
          apiKey: "FIRECRAWL_API_KEY_HERE", // 如果已设置 FIRECRAWL_API_KEY 环境变量则此项可选
          baseUrl: "https://api.firecrawl.dev",
          onlyMainContent: true,
          maxAgeMs: 86400000, // ms (1 day)
          timeoutSeconds: 60
        }
      }
    }
  }
}
```

<div id="tool-parameters">
  ### 工具参数
</div>

* `url`（必填，仅支持 http/https）
* `extractMode`（`markdown` | `text`）
* `maxChars`（截断超长页面）

注意：

* `web_fetch` 会优先使用 Readability（主体内容提取），然后使用 Firecrawl（如果已配置）。如果两者都失败，该工具会返回错误。
* Firecrawl 请求默认使用绕过机器人检测的模式，并对结果进行缓存。
* `web_fetch` 默认发送类似 Chrome 的 User-Agent 和 `Accept-Language`；如有需要，可以通过 `userAgent` 覆盖。
* `web_fetch` 会阻止访问私有/内部主机名，并对重定向进行重新检查（可通过 `maxRedirects` 限制）。
* `web_fetch` 仅做尽力提取；有些站点需要使用浏览器工具。
* 参见 [Firecrawl](/zh/tools/firecrawl) 了解密钥配置和服务细节。
* 响应会被缓存（默认 15 分钟），以减少重复抓取。
* 如果你使用工具配置文件/允许列表，请添加 `web_search`/`web_fetch` 或 `group:web`。
* 如果缺少 Brave 密钥，`web_search` 会返回一个简短的配置提示及文档链接。