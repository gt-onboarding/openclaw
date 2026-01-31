---
title: Ollama
summary: "使用 Ollama（本地 LLM 运行时）来运行 OpenClaw"
read_when:
  - 你想通过 Ollama 使用本地模型来运行 OpenClaw
  - 你需要关于 Ollama 的安装和配置指导
---

<div id="ollama">
  # Ollama
</div>

Ollama 是一个本地 LLM 运行时，使你能够在自己的机器上轻松运行开源模型。OpenClaw 通过 Ollama 的 OpenAI 兼容 api 进行集成，并且当你启用 `OLLAMA_API_KEY`（或某个身份验证配置档案）且未显式定义 `models.providers.ollama` 条目时，可以**自动发现具备工具调用能力的模型**。

<div id="quick-start">
  ## 快速开始
</div>

1. 安装 Ollama：https://ollama.ai

2. 拉取模型：

```bash
ollama pull llama3.3
# 或
ollama pull qwen2.5-coder:32b
# 或
ollama pull deepseek-r1:32b
```

3. 为 OpenClaw 启用 Ollama（随便填一个值即可；Ollama 不需要真实密钥）：

```bash
# 设置环境变量
export OLLAMA_API_KEY="ollama-local"

# 或在配置文件中进行配置
openclaw config set models.providers.ollama.apiKey "ollama-local"
```

4. 使用 Ollama 模型：

```json5
{
  agents: {
    defaults: {
      model: { primary: "ollama/llama3.3" }
    }
  }
}
```

<div id="model-discovery-implicit-provider">
  ## 模型发现（隐式提供方）
</div>

当你设置了 `OLLAMA_API_KEY`（或一个认证配置文件）且**没有**定义 `models.providers.ollama` 时，OpenClaw 会从本地 Ollama 实例 `http://127.0.0.1:11434` 自动发现模型：

* 查询 `/api/tags` 和 `/api/show`
* 只保留报告具备 `tools` 能力的模型
* 当模型报告 `thinking` 时，将其标记为具备 `reasoning` 能力
* 如可用，从 `model_info["<arch>.context_length"]` 读取 `contextWindow`
* 将 `maxTokens` 设置为上下文窗口大小的 10 倍
* 将所有费用设置为 `0`

这样可以避免手动维护模型条目，同时让模型目录与 Ollama 的能力保持一致。

要查看可用的模型：

```bash
ollama list
openclaw models list
```

要添加新模型，只需使用 Ollama 拉取该模型：

```bash
ollama pull mistral
```

新模型会被自动发现并可直接使用。

如果你显式设置了 `models.providers.ollama`，则会跳过自动发现，你必须手动定义模型（见下文）。

<div id="configuration">
  ## 配置
</div>

<div id="basic-setup-implicit-discovery">
  ### 基础配置（隐式发现）
</div>

启用 Ollama 的最简单方式是通过设置环境变量：

```bash
export OLLAMA_API_KEY="ollama-local"
```

<div id="explicit-setup-manual-models">
  ### 显式设置（手动配置模型）
</div>

在以下情况下使用显式配置：

* Ollama 运行在其他主机或端口上。
* 你想强制使用特定的上下文窗口或模型列表。
* 你想包含不会报告工具支持情况的模型。

```json5
{
  models: {
    providers: {
      ollama: {
        // 使用包含 /v1 的主机地址以兼容 OpenAI API
        baseUrl: "http://ollama-host:11434/v1",
        apiKey: "ollama-local",
        api: "openai-completions",
        models: [
          {
            id: "llama3.3",
            name: "Llama 3.3",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 8192,
            maxTokens: 8192 * 10
          }
        ]
      }
    }
  }
}
```

如果设置了 `OLLAMA_API_KEY`，你可以在提供方条目中省略 `apiKey`，OpenClaw 会在进行可用性检查时自动填入该值。

<div id="custom-base-url-explicit-config">
  ### 自定义基础 URL（显式配置）
</div>

如果 Ollama 运行在不同的主机或端口上（显式配置会禁用自动发现，因此需要手动配置模型）：

```json5
{
  models: {
    providers: {
      ollama: {
        apiKey: "ollama-local",
        baseUrl: "http://ollama-host:11434/v1"
      }
    }
  }
}
```

<div id="model-selection">
  ### 模型选择
</div>

完成配置后，你的所有 Ollama 模型就都可以使用：

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "ollama/llama3.3",
        fallback: ["ollama/qwen2.5-coder:32b"]
      }
    }
  }
}
```

<div id="advanced">
  ## 高级用法
</div>

<div id="reasoning-models">
  ### 推理模型
</div>

当 Ollama 在 `/api/show` 中返回 `thinking` 时，OpenClaw 会将模型标记为支持推理：

```bash
ollama pull deepseek-r1:32b
```

<div id="model-costs">
  ### 模型费用
</div>

Ollama 是免费的并在本地运行，因此所有模型费用都为 0 美元。

<div id="context-windows">
  ### 上下文窗口
</div>

对于自动检测的模型，如果可用，OpenClaw 会使用 Ollama 报告的上下文窗口，否则默认使用 `8192`。你可以在提供方的显式配置中自定义 `contextWindow` 和 `maxTokens`。

<div id="troubleshooting">
  ## 故障排查
</div>

<div id="ollama-not-detected">
  ### 未检测到 Ollama
</div>

请确保 Ollama 正在运行，并且你已经设置了 `OLLAMA_API_KEY`（或一个认证配置文件），并确认你**没有**显式定义 `models.providers.ollama` 条目：

```bash
ollama serve
```

并且可以访问 API：

```bash
curl http://localhost:11434/api/tags
```

<div id="no-models-available">
  ### 没有可用模型
</div>

OpenClaw 只会自动发现声明支持工具的模型。如果你的模型没有出现在列表中，可以：

* 拉取一个支持工具的模型，或
* 在 `models.providers.ollama` 中显式定义该模型。

要添加模型：

```bash
ollama list  # 查看已安装的内容
ollama pull llama3.3  # 拉取模型
```

<div id="connection-refused">
  ### 连接被拒绝
</div>

检查 Ollama 是否在正确的端口上运行：

```bash
# 检查 Ollama 是否正在运行
ps aux | grep ollama

# 或重启 Ollama
ollama serve
```

<div id="see-also">
  ## 另请参阅
</div>

* [Model Providers](/zh/concepts/model-providers) - 所有模型提供方的概览
* [Model Selection](/zh/concepts/models) - 如何选择模型
* [Configuration](/zh/gateway/configuration) - 完整配置参考