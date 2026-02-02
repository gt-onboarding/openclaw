---
title: Venice
summary: "在 OpenClaw 中使用 Venice AI 的隐私优先模型"
read_when:
  - 你希望在 OpenClaw 中使用注重隐私的推理能力
  - 你需要 Venice AI 的安装与配置指南
---

<div id="venice-ai-venice-highlight">
  # Venice AI（Venice 重点介绍）
</div>

**Venice** 是我们为 Venice 提供的重点部署方案，在隐私优先的前提下进行推理，并可选择以匿名方式访问专有模型。

Venice AI 提供以隐私为核心的 AI 推理能力，支持未审查模型，并可通过其匿名代理访问主流专有模型。所有推理默认都是私有的——不会基于你的数据进行训练，也不会记录日志。

<div id="why-venice-in-openclaw">
  ## 为什么在 OpenClaw 中使用 Venice
</div>

- 面向开源模型的**私有推理**（不记录日志）。
- 在需要时可使用**无审查模型**。
- 在质量优先的场景下，可**匿名访问**专有模型（Opus/GPT/Gemini）。
- 兼容 OpenAI 的 `/v1` 端点。

<div id="privacy-modes">
  ## 隐私模式
</div>

Venice 提供两种隐私级别——理解这两种模式有助于你选择合适的模型：

| 模式 | 描述 | 模型 |
|------|-------------|--------|
| **私密** | 完全私密。提示词/回复内容**永不被存储或记录**，仅作临时处理。 | Llama、Qwen、DeepSeek、Venice Uncensored 等 |
| **匿名化** | 通过 Venice 代理，并剥离元数据。底层提供方（OpenAI、Anthropic）只会看到匿名化请求。 | Claude、GPT、Gemini、Grok、Kimi、MiniMax |

<div id="features">
  ## 功能
</div>

- **注重隐私**：可在 "private"（完全私密）和 "anonymized"（经代理匿名）模式之间选择
- **未做内容审查的模型**：可访问无内容限制的模型
- **主流模型访问**：通过 Venice 的匿名代理使用 Claude、GPT-5.2、Gemini、Grok
- **兼容 OpenAI 的 API**：提供标准 `/v1` 端点，便于集成
- **流式输出**：✅ 所有模型均支持
- **函数调用**：✅ 部分模型支持（请查看模型功能）
- **视觉能力**：✅ 具备视觉能力的模型支持
- **无硬性速率限制**：极端用量情况下可能会触发公平使用限流

<div id="setup">
  ## 配置
</div>

<div id="1-get-api-key">
  ### 1. 获取 API 密钥
</div>

1. 在 [venice.ai](https://venice.ai) 注册账号
2. 进入 **Settings → API Keys → Create new key**
3. 复制你的 API 密钥（格式：`vapi_xxxxxxxxxxxx`）

<div id="2-configure-openclaw">
  ### 2. 配置 OpenClaw
</div>

**选项 A：环境变量**

```bash
export VENICE_API_KEY="vapi_xxxxxxxxxxxx"
```

**选项 B：交互式安装（推荐）**

```bash
openclaw onboard --auth-choice venice-api-key
```

这将会：

1. 提示你输入你的 API 密钥（或使用已有的 `VENICE_API_KEY`）
2. 列出所有可用的 Venice 模型
3. 让你选择默认模型
4. 自动配置提供方

**选项 C：非交互式模式**

```bash
openclaw onboard --non-interactive \
  --auth-choice venice-api-key \
  --venice-api-key "vapi_xxxxxxxxxxxx"
```


<div id="3-verify-setup">
  ### 3. 验证配置
</div>

```bash
openclaw chat --model venice/llama-3.3-70b "Hello, are you working?"
```


<div id="model-selection">
  ## 模型选择
</div>

完成配置后，OpenClaw 会显示所有可用的 Venice 模型。请根据你的需求进行选择：

* **默认（我们的推荐）**：`venice/llama-3.3-70b`，适用于注重隐私且性能均衡的场景。
* **整体质量最佳**：`venice/claude-opus-45`，适用于高难度任务（Opus 依然是最强模型）。
* **隐私**：选择 “private” 模型以获得完全私密的推理。
* **能力**：选择 “anonymized” 模型，通过 Venice 的代理访问 Claude、GPT、Gemini 等。

你可以随时更改默认模型：

```bash
openclaw models set venice/claude-opus-45
openclaw models set venice/llama-3.3-70b
```

列出所有可用的模型：

```bash
openclaw models list | grep venice
```


<div id="configure-via-openclaw-configure">
  ## 使用 `openclaw configure` 进行配置
</div>

1. 运行 `openclaw configure`
2. 选择 **Model/auth**
3. 选择 **Venice AI**

<div id="which-model-should-i-use">
  ## 我应该使用哪个模型？
</div>

| 使用场景 | 推荐模型 | 原因 |
|----------|-------------------|-----|
| **通用对话** | `llama-3.3-70b` | 综合表现优秀，完全私有 |
| **整体质量最佳** | `claude-opus-45` | Opus 在处理高难度任务时依然最强 |
| **隐私 + Claude 质量** | `claude-opus-45` | 通过匿名代理提供最强推理能力 |
| **编程** | `qwen3-coder-480b-a35b-instruct` | 针对代码优化，支持 262k 上下文 |
| **视觉任务** | `qwen3-vl-235b-a22b` | 最佳私有视觉模型 |
| **无审查** | `venice-uncensored` | 无内容限制 |
| **快速 + 低成本** | `qwen3-4b` | 轻量但依然够用 |
| **复杂推理** | `deepseek-v3.2` | 推理能力强，且支持私有部署 |

<div id="available-models-25-total">
  ## 可用模型（共 25 个）
</div>

<div id="private-models-15-fully-private-no-logging">
  ### 私有模型（15） — 完全私有，无日志
</div>

| Model ID | 名称 | 上下文长度（tokens） | 特性 |
|----------|------|----------------------|----------|
| `llama-3.3-70b` | Llama 3.3 70B | 131k | 通用 |
| `llama-3.2-3b` | Llama 3.2 3B | 131k | 快速、轻量 |
| `hermes-3-llama-3.1-405b` | Hermes 3 Llama 3.1 405B | 131k | 复杂任务 |
| `qwen3-235b-a22b-thinking-2507` | Qwen3 235B Thinking | 131k | 推理 |
| `qwen3-235b-a22b-instruct-2507` | Qwen3 235B Instruct | 131k | 通用 |
| `qwen3-coder-480b-a35b-instruct` | Qwen3 Coder 480B | 262k | 代码 |
| `qwen3-next-80b` | Qwen3 Next 80B | 262k | 通用 |
| `qwen3-vl-235b-a22b` | Qwen3 VL 235B | 262k | 视觉 |
| `qwen3-4b` | Venice 小型（Qwen3 4B） | 32k | 快速、推理 |
| `deepseek-v3.2` | DeepSeek V3.2 | 163k | 推理 |
| `venice-uncensored` | Venice 未审查版 | 32k | 无内容审查 |
| `mistral-31-24b` | Venice 中型（Mistral） | 131k | 视觉 |
| `google-gemma-3-27b-it` | Gemma 3 27B Instruct | 202k | 视觉 |
| `openai-gpt-oss-120b` | OpenAI GPT OSS 120B | 131k | 通用 |
| `zai-org-glm-4.7` | GLM 4.7 | 202k | 推理、多语言 |

<div id="anonymized-models-10-via-venice-proxy">
  ### 匿名模型 (10) — 通过 Venice 代理
</div>

| Model ID | 原始名称 | 上下文（tokens） | 特性 |
|----------|----------|------------------|----------|
| `claude-opus-45` | Claude Opus 4.5 | 202k | 推理、视觉 |
| `claude-sonnet-45` | Claude Sonnet 4.5 | 202k | 推理、视觉 |
| `openai-gpt-52` | GPT-5.2 | 262k | 推理 |
| `openai-gpt-52-codex` | GPT-5.2 Codex | 262k | 推理、视觉 |
| `gemini-3-pro-preview` | Gemini 3 Pro | 202k | 推理、视觉 |
| `gemini-3-flash-preview` | Gemini 3 Flash | 262k | 推理、视觉 |
| `grok-41-fast` | Grok 4.1 Fast | 262k | 推理、视觉 |
| `grok-code-fast-1` | Grok Code Fast 1 | 262k | 推理、代码 |
| `kimi-k2-thinking` | Kimi K2 Thinking | 262k | 推理 |
| `minimax-m21` | MiniMax M2.1 | 202k | 推理 |

<div id="model-discovery">
  ## 模型发现
</div>

当设置了 `VENICE_API_KEY` 时，OpenClaw 会自动从 Venice API 中发现可用模型。如果 API 无法访问，则会退回使用静态模型目录。

`/models` 端点是公开的（列出模型不需要认证），但进行推理需要有效的 API 密钥。

<div id="streaming-tool-support">
  ## 流式与工具支持
</div>

| 功能 | 支持情况 |
|---------|---------|
| **Streaming** | ✅ 所有模型 |
| **函数调用（Function calling）** | ✅ 大多数模型（在 API 中查看 `supportsFunctionCalling`） |
| **视觉/图像（Vision/Images）** | ✅ 标记有 “Vision” 功能的模型 |
| **JSON 模式** | ✅ 通过 `response_format` 支持 |

<div id="pricing">
  ## 定价
</div>

Venice 使用基于积分的计费系统。当前费率请查看 [venice.ai/pricing](https://venice.ai/pricing)：

- **私有模型**：通常费用更低
- **匿名化模型**：价格与直接使用 api 相近，外加少量 Venice 手续费

<div id="comparison-venice-vs-direct-api">
  ## 对比：Venice vs 直接 API
</div>

| 方面 | Venice（匿名化） | 直接 API |
|--------|---------------------|------------|
| **隐私** | 元数据会被剥离并匿名处理 | 与你的账号直接关联 |
| **延迟** | +10-50ms（经由代理） | 直连 |
| **特性** | 支持大多数功能 | 全部功能 |
| **计费** | Venice 额度 | 提供方计费 |

<div id="usage-examples">
  ## 使用示例
</div>

```bash
# Use default private model
openclaw chat --model venice/llama-3.3-70b

# 通过 Venice 使用 Claude(匿名)
openclaw chat --model venice/claude-opus-45

# Use uncensored model
openclaw chat --model venice/venice-uncensored

# Use vision model with image
openclaw chat --model venice/qwen3-vl-235b-a22b

# Use coding model
openclaw chat --model venice/qwen3-coder-480b-a35b-instruct
```


<div id="troubleshooting">
  ## 疑难解答
</div>

<div id="api-key-not-recognized">
  ### API 密钥无法识别
</div>

```bash
echo $VENICE_API_KEY
openclaw models list | grep venice
```

确保密钥以 `vapi_` 开头。


<div id="model-not-available">
  ### 模型不可用
</div>

Venice 模型目录会动态更新。运行 `openclaw models list` 查看当前可用的模型。某些模型可能会暂时下线。

<div id="connection-issues">
  ### 连接问题
</div>

Venice API 位于 `https://api.venice.ai/api/v1`。确保你的网络允许发起 HTTPS 连接。

<div id="config-file-example">
  ## 配置文件示例
</div>

```json5
{
  env: { VENICE_API_KEY: "vapi_..." },
  agents: { defaults: { model: { primary: "venice/llama-3.3-70b" } } },
  models: {
    mode: "merge",
    providers: {
      venice: {
        baseUrl: "https://api.venice.ai/api/v1",
        apiKey: "${VENICE_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "llama-3.3-70b",
            name: "Llama 3.3 70B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 131072,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```


<div id="links">
  ## 链接
</div>

- [Venice AI](https://venice.ai)
- [API 文档](https://docs.venice.ai)
- [定价](https://venice.ai/pricing)
- [状态](https://status.venice.ai)