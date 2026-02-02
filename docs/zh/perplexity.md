---
title: Perplexity
summary: "用于 web_search 的 Perplexity Sonar 设置"
read_when:
  - 你希望使用 Perplexity Sonar 进行网页搜索
  - 你需要 PERPLEXITY_API_KEY 或已完成 OpenRouter 的设置
---

<div id="perplexity-sonar">
  # Perplexity Sonar
</div>

OpenClaw 可以将 Perplexity Sonar 用于 `web_search` 工具。你可以通过 Perplexity 提供的直接 api 或 OpenRouter 进行连接。

<div id="api-options">
  ## API 选项
</div>

<div id="perplexity-direct">
  ### Perplexity（直接连接）
</div>

* 基本 URL：https://api.perplexity.ai
* 环境变量：`PERPLEXITY_API_KEY`

<div id="openrouter-alternative">
  ### OpenRouter（备选）
</div>

* 基础 URL：https://openrouter.ai/api/v1
* 环境变量：`OPENROUTER_API_KEY`
* 支持预付费/加密货币余额。

<div id="config-example">
  ## 配置示例
</div>

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

<div id="switching-from-brave">
  ## 从 Brave 迁移过来
</div>

```json5
{
  tools: {
    web: {
      search: {
        provider: "perplexity",
        perplexity: {
          apiKey: "pplx-...",
          baseUrl: "https://api.perplexity.ai"
        }
      }
    }
  }
}
```

如果同时设置了 `PERPLEXITY_API_KEY` 和 `OPENROUTER_API_KEY`，请设置
`tools.web.search.perplexity.baseUrl`（或 `tools.web.search.perplexity.apiKey`）
来消除歧义。

如果未设置 base URL，OpenClaw 会基于 API 密钥来源选择默认值：

* `PERPLEXITY_API_KEY` 或 `pplx-...` → 直连 Perplexity（`https://api.perplexity.ai`）
* `OPENROUTER_API_KEY` 或 `sk-or-...` → OpenRouter（`https://openrouter.ai/api/v1`）
* 未知密钥格式 → OpenRouter（安全兜底）

<div id="models">
  ## 模型
</div>

* `perplexity/sonar` — 通过网页搜索实现快速问答
* `perplexity/sonar-pro` (默认) — 多步推理 + 网页搜索
* `perplexity/sonar-reasoning-pro` — 深度研究

完整的 `web_search` 配置请参见 [Web 工具](/zh/tools/web)。