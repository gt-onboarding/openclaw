---
title: 模型提供方
summary: "模型提供方概览，以及示例配置与 CLI 操作流程"
read_when:
  - 你需要按提供方划分的模型配置参考
  - 你想要模型提供方的示例配置或 CLI 入门命令
---

<div id="model-providers">
  # 模型提供方
</div>

本页介绍的是 **LLM/模型提供方**（而不是 WhatsApp、Telegram 之类的聊天渠道）。
关于模型选择规则，请参见 [/concepts/models](/zh/concepts/models)。

<div id="quick-rules">
  ## 快速规则
</div>

* 模型引用使用 `provider/model` 格式（例如：`opencode/claude-opus-4-5`）。
* 如果你设置了 `agents.defaults.models`，它就会成为允许列表。
* CLI 辅助命令：`openclaw onboard`、`openclaw models list`、`openclaw models set <provider/model>`。

<div id="built-in-providers-pi-ai-catalog">
  ## 内置提供方（pi-ai 目录）
</div>

OpenClaw 内置了 pi‑ai 目录。这些提供方**不需要**任何
`models.providers` 配置；你只需设置认证信息并选择一个模型即可。

<div id="openai">
  ### OpenAI
</div>

* 提供方：`openai`
* 认证方式：`OPENAI_API_KEY`
* 示例模型：`openai/gpt-5.2`
* CLI：`openclaw onboard --auth-choice openai-api-key`

```json5
{
  agents: { defaults: { model: { primary: "openai/gpt-5.2" } } }
}
```

<div id="anthropic">
  ### Anthropic
</div>

* 提供方：`anthropic`
* 认证方式：`ANTHROPIC_API_KEY` 或 `claude setup-token`
* 示例模型：`anthropic/claude-opus-4-5`
* CLI：`openclaw onboard --auth-choice token`（粘贴 setup-token）或 `openclaw models auth paste-token --provider anthropic`

```json5
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-5" } } }
}
```

<div id="openai-code-codex">
  ### OpenAI 代码（Codex）
</div>

* 提供方：`openai-codex`
* 认证：OAuth（ChatGPT）
* 示例模型：`openai-codex/gpt-5.2`
* CLI：`openclaw onboard --auth-choice openai-codex` 或 `openclaw models auth login --provider openai-codex`

```json5
{
  agents: { defaults: { model: { primary: "openai-codex/gpt-5.2" } } }
}
```

<div id="opencode-zen">
  ### OpenCode Zen
</div>

* 提供方: `opencode`
* 认证: `OPENCODE_API_KEY`（或 `OPENCODE_ZEN_API_KEY`）
* 示例模型: `opencode/claude-opus-4-5`
* CLI: `openclaw onboard --auth-choice opencode-zen`

```json5
{
  agents: { defaults: { model: { primary: "opencode/claude-opus-4-5" } } }
}
```

<div id="google-gemini-api-key">
  ### Google Gemini（API 密钥）
</div>

* 提供方：`google`
* 身份验证：`GEMINI_API_KEY`
* 示例模型：`google/gemini-3-pro-preview`
* CLI：`openclaw onboard --auth-choice gemini-api-key`

<div id="google-vertex-antigravity-gemini-cli">
  ### Google Vertex / Antigravity / Gemini CLI
</div>

* 提供方：`google-vertex`、`google-antigravity`、`google-gemini-cli`
* 身份验证：Vertex 使用 gcloud ADC；Antigravity/Gemini CLI 使用各自的身份验证流程
* Antigravity OAuth 作为内置插件提供（`google-antigravity-auth`，默认禁用）。
  * 启用：`openclaw plugins enable google-antigravity-auth`
  * 登录：`openclaw models auth login --provider google-antigravity --set-default`
* Gemini CLI OAuth 作为内置插件提供（`google-gemini-cli-auth`，默认禁用）。
  * 启用：`openclaw plugins enable google-gemini-cli-auth`
  * 登录：`openclaw models auth login --provider google-gemini-cli --set-default`
  * 注意：你**不需要**在 `openclaw.json` 中粘贴客户端 ID 或密钥。CLI 登录流程会将
    令牌存储在 Gateway 主机上的认证配置文件中。

<div id="zai-glm">
  ### Z.AI (GLM)
</div>

* 提供方: `zai`
* 鉴权: `ZAI_API_KEY`
* 示例模型: `zai/glm-4.7`
* CLI: `openclaw onboard --auth-choice zai-api-key`
  * 别名: `z.ai/*` 和 `z-ai/*` 会被规范化为 `zai/*`

<div id="vercel-ai-gateway">
  ### Vercel AI Gateway
</div>

* 提供方: `vercel-ai-gateway`
* 认证: 环境变量 `AI_GATEWAY_API_KEY`
* 示例模型: `vercel-ai-gateway/anthropic/claude-opus-4.5`
* CLI: `openclaw onboard --auth-choice ai-gateway-api-key`

<div id="other-built-in-providers">
  ### 其他内置提供方
</div>

* OpenRouter：`openrouter`（`OPENROUTER_API_KEY`）
* 示例模型：`openrouter/anthropic/claude-sonnet-4-5`
* xAI：`xai`（`XAI_API_KEY`）
* Groq：`groq`（`GROQ_API_KEY`）
* Cerebras：`cerebras`（`CEREBRAS_API_KEY`）
  * Cerebras 上的 GLM 模型使用 ID `zai-glm-4.7` 和 `zai-glm-4.6`。
  * 兼容 OpenAI 的基础 URL 为：`https://api.cerebras.ai/v1`。
* Mistral：`mistral`（`MISTRAL_API_KEY`）
* GitHub Copilot：`github-copilot`（`COPILOT_GITHUB_TOKEN` / `GH_TOKEN` / `GITHUB_TOKEN`）

<div id="providers-via-modelsproviders-custombase-url">
  ## 通过 `models.providers` 配置提供方（自定义/基础 URL）
</div>

使用 `models.providers`（或 `models.json`）来添加 **自定义** 提供方或
与 OpenAI/Anthropic 兼容的代理。

<div id="moonshot-ai-kimi">
  ### Moonshot AI（Kimi）
</div>

Moonshot 使用与 OpenAI 兼容的 API 端点，因此将其配置为自定义提供方：

* 提供方：`moonshot`
* 认证方式：`MOONSHOT_API_KEY`
* 示例模型：`moonshot/kimi-k2.5`
* Kimi K2 模型 ID：
  {/* moonshot-kimi-k2-model-refs:start */}
  * `moonshot/kimi-k2.5`
  * `moonshot/kimi-k2-0905-preview`
  * `moonshot/kimi-k2-turbo-preview`
  * `moonshot/kimi-k2-thinking`
  * `moonshot/kimi-k2-thinking-turbo`
  {/* moonshot-kimi-k2-model-refs:end */}

```json5
{
  agents: {
    defaults: { model: { primary: "moonshot/kimi-k2.5" } }
  },
  models: {
    mode: "merge",
    providers: {
      moonshot: {
        baseUrl: "https://api.moonshot.ai/v1",
        apiKey: "${MOONSHOT_API_KEY}",
        api: "openai-completions",
        models: [{ id: "kimi-k2.5", name: "Kimi K2.5" }]
      }
    }
  }
}
```

<div id="kimi-code">
  ### Kimi Code
</div>

Kimi Code 使用独立的 endpoint 和密钥（与 Moonshot 不同）：

* 提供方：`kimi-code`
* 鉴权：`KIMICODE_API_KEY`
* 示例模型：`kimi-code/kimi-for-coding`

```json5
{
  env: { KIMICODE_API_KEY: "sk-..." },
  agents: {
    defaults: { model: { primary: "kimi-code/kimi-for-coding" } }
  },
  models: {
    mode: "merge",
    providers: {
      "kimi-code": {
        baseUrl: "https://api.kimi.com/coding/v1",
        apiKey: "${KIMICODE_API_KEY}",
        api: "openai-completions",
        models: [{ id: "kimi-for-coding", name: "Kimi For Coding" }]
      }
    }
  }
}
```

<div id="qwen-oauth-free-tier">
  ### Qwen OAuth（免费层）
</div>

Qwen 通过设备代码（device code）流程提供对 Qwen Coder + Vision 的 OAuth 访问。
启用随附的插件，然后登录：

```bash
openclaw plugins enable qwen-portal-auth
openclaw models auth login --provider qwen-portal --set-default
```

模型引用：

* `qwen-portal/coder-model`
* `qwen-portal/vision-model`

有关配置详情和注意事项，请参见 [/providers/qwen](/zh/providers/qwen)。

<div id="synthetic">
  ### Synthetic
</div>

Synthetic 通过 `synthetic` 提供方提供与 Anthropic 兼容的模型：

* 提供方（Provider）：`synthetic`
* 认证（Auth）：`SYNTHETIC_API_KEY`
* 示例模型（Example model）：`synthetic/hf:MiniMaxAI/MiniMax-M2.1`
* CLI：`openclaw onboard --auth-choice synthetic-api-key`

```json5
{
  agents: {
    defaults: { model: { primary: "synthetic/hf:MiniMaxAI/MiniMax-M2.1" } }
  },
  models: {
    mode: "merge",
    providers: {
      synthetic: {
        baseUrl: "https://api.synthetic.new/anthropic",
        apiKey: "${SYNTHETIC_API_KEY}",
        api: "anthropic-messages",
        models: [{ id: "hf:MiniMaxAI/MiniMax-M2.1", name: "MiniMax M2.1" }]
      }
    }
  }
}
```

<div id="minimax">
  ### MiniMax
</div>

MiniMax 通过 `models.providers` 进行配置，因为它使用自定义端点：

* MiniMax（兼容 Anthropic）：`--auth-choice minimax-api`
* 身份验证：`MINIMAX_API_KEY`

参见 [/providers/minimax](/zh/providers/minimax) 以获取配置详情、模型选项和配置示例片段。

<div id="ollama">
  ### Ollama
</div>

Ollama 是一个本地 LLM 运行时，提供 OpenAI 兼容的 API：

* 提供方: `ollama`
* 认证: 无需认证（本地服务器）
* 示例模型: `ollama/llama3.3`
* 安装: https://ollama.ai

```bash
# 安装 Ollama,然后拉取一个模型:
ollama pull llama3.3
```

```json5
{
  agents: {
    defaults: { model: { primary: "ollama/llama3.3" } }
  }
}
```

当在本地运行于 `http://127.0.0.1:11434/v1` 时，Ollama 会被自动检测到。有关模型推荐和自定义配置，请参阅 [/providers/ollama](/zh/providers/ollama)。

<div id="local-proxies-lm-studio-vllm-litellm-etc">
  ### 本地代理（LM Studio、vLLM、LiteLLM 等）
</div>

示例（OpenAI 兼容）：

```json5
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.1-gs32" },
      models: { "lmstudio/minimax-m2.1-gs32": { alias: "Minimax" } }
    }
  },
  models: {
    providers: {
      lmstudio: {
        baseUrl: "http://localhost:1234/v1",
        apiKey: "LMSTUDIO_KEY",
        api: "openai-completions",
        models: [
          {
            id: "minimax-m2.1-gs32",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 200000,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

说明：

* 对于自定义提供方，`reasoning`、`input`、`cost`、`contextWindow` 和 `maxTokens` 都是可选的。
  如果省略，OpenClaw 的默认值为：
  * `reasoning: false`
  * `input: ["text"]`
  * `cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 }`
  * `contextWindow: 200000`
  * `maxTokens: 8192`
* 建议：显式设置与代理/模型限制相匹配的具体数值。

<div id="cli-examples">
  ## CLI 示例
</div>

```bash
openclaw onboard --auth-choice opencode-zen
openclaw models set opencode/claude-opus-4-5
openclaw models list
```

另请参阅 [/gateway/configuration](/zh/gateway/configuration)，了解完整的配置示例。
