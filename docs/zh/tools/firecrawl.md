---
title: Firecrawl
summary: "用于 web_fetch 的 Firecrawl 回退方案（反爬虫 + 缓存提取）"
read_when:
  - 你想要基于 Firecrawl 的网页内容提取
  - 你需要一个 Firecrawl API 密钥
  - 你希望为 web_fetch 启用具备反爬虫能力的网页提取
---

<div id="firecrawl">
  # Firecrawl
</div>

OpenClaw 可以将 **Firecrawl** 用作 `web_fetch` 的回退提取器。它是一个托管的
内容提取服务，支持绕过机器人防护并提供结果缓存，可用于处理大量依赖 JS 的站点，
或那些阻止普通 HTTP 抓取的页面。

<div id="get-an-api-key">
  ## 获取 API 密钥
</div>

1. 注册一个 Firecrawl 账号并生成一个 API 密钥。
2. 将其保存到配置文件中，或在 Gateway 运行环境中设置 `FIRECRAWL_API_KEY`。

<div id="configure-firecrawl">
  ## 配置 Firecrawl
</div>

```json5
{
  tools: {
    web: {
      fetch: {
        firecrawl: {
          apiKey: "FIRECRAWL_API_KEY_HERE",
          baseUrl: "https://api.firecrawl.dev",
          onlyMainContent: true,
          maxAgeMs: 172800000,
          timeoutSeconds: 60
        }
      }
    }
  }
}
```

备注：

* 当存在 api 密钥时，`firecrawl.enabled` 默认为 true。
* `maxAgeMs` 控制缓存结果的最大允许时长（毫秒）。默认值为 2 天。

<div id="stealth-bot-circumvention">
  ## 隐身 / 反机器人检测规避
</div>

Firecrawl 提供了一个用于绕过机器人检测的 **proxy 模式** 参数（`basic`、`stealth` 或 `auto`）。
OpenClaw 对 Firecrawl 请求始终使用 `proxy: "auto"` 加上 `storeInCache: true`。
如果省略 proxy，Firecrawl 会默认使用 `auto`。`auto` 会在 basic 模式的首次尝试失败时改用隐身代理重试，这可能会比仅使用 basic 模式抓取消耗更多额度。

<div id="how-web_fetch-uses-firecrawl">
  ## `web_fetch` 如何使用 Firecrawl
</div>

`web_fetch` 的提取顺序：

1. Readability（本地）
2. Firecrawl（如果已配置）
3. 基本 HTML 清理（最终兜底）

有关 Web 工具的完整配置说明，请参见 [Web tools](/zh/tools/web)。