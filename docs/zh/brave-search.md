---
title: Brave Search
summary: "用于 web_search 的 Brave Search API 配置"
read_when:
  - 你想在 web_search 中使用 Brave Search
  - 你需要 BRAVE_API_KEY 或套餐详情
---

<div id="brave-search-api">
  # Brave Search API
</div>

OpenClaw 将 Brave Search 用作 `web_search` 的默认提供方。

<div id="get-an-api-key">
  ## 获取 API 密钥
</div>

1. 在 https://brave.com/search/api/ 创建一个 Brave Search API 账号。
2. 在控制面板中选择 **Data for Search** 方案并生成一个 API 密钥。
3. 将该密钥存储在配置文件中（推荐），或在 Gateway 运行环境中将 `BRAVE_API_KEY` 设置为环境变量。

<div id="config-example">
  ## 配置示例
</div>

```json5
{
  tools: {
    web: {
      search: {
        provider: "brave",
        apiKey: "BRAVE_API_KEY_HERE",
        maxResults: 5,
        timeoutSeconds: 30
      }
    }
  }
}
```

<div id="notes">
  ## 注意事项
</div>

* Data for AI 计划**不**兼容 `web_search`。
* Brave 提供免费额度和付费计划；请在 Brave API 门户网站中查看当前限制。

有关完整的 web&#95;search 配置，请参见 [Web tools](/zh/tools/web)。