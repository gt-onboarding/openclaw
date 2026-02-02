---
title: 本地模型
summary: "在本地部署的 LLM 上运行 OpenClaw（LM Studio、vLLM、LiteLLM、自定义 OpenAI 端点）"
read_when:
  - 你想从自己的 GPU 机器/服务器上提供模型服务
  - 你正在接入 LM Studio 或兼容 OpenAI 的代理
  - 你需要关于本地模型的最高安全实践指引
---

<div id="local-models">
  # 本地模型
</div>

本地是可行的，但 OpenClaw 需要大型上下文窗口和强有力的提示注入防御。小显存卡会截断上下文并削弱安全性。把目标定高一点：**至少 2 台顶配 Mac Studio 或等效的 GPU 设备（约 3 万美元以上）**。单块 **24 GB** GPU 只适合较轻量的提示词请求，而且延迟更高。请尽可能使用你能跑得动的**最大 / 全尺寸模型版本**；量化过狠或“精简版”的模型检查点会显著提高提示注入风险（参见 [Security](/zh/gateway/security)）。

<div id="recommended-lm-studio-minimax-m21-responses-api-full-size">
  ## 推荐：LM Studio + MiniMax M2.1（Responses API，完整模型）
</div>

当前最佳本地方案。在 LM Studio 中加载 MiniMax M2.1，启用本地服务器（默认 `http://127.0.0.1:1234`），并使用 Responses API 将推理过程与最终文本输出分离。

```json5
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.1-gs32" },
      models: {
        "anthropic/claude-opus-4-5": { alias: "Opus" },
        "lmstudio/minimax-m2.1-gs32": { alias: "Minimax" }
      }
    }
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.1-gs32",
            name: "MiniMax M2.1 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

**设置检查清单**

* 安装 LM Studio：https://lmstudio.ai
* 在 LM Studio 中下载**当前可用的最大 MiniMax M2.1 模型构建**（避免使用 “small”/高度量化的变体），启动服务器，确认在 `http://127.0.0.1:1234/v1/models` 中能看到该模型。
* 保持模型常驻内存；冷加载会增加启动延迟。
* 如果你的 LM Studio 构建不同，请调整 `contextWindow`/`maxTokens`。
* 对于 WhatsApp，请始终使用 Responses API，这样只会发送最终文本。

即使运行本地模型，也要保持托管模型处于已配置状态；使用 `models.mode: "merge"` 以便回退模型始终可用。

<div id="hybrid-config-hosted-primary-local-fallback">
  ### 混合配置：以云端为主，本地兜底
</div>

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-sonnet-4-5",
        fallbacks: ["lmstudio/minimax-m2.1-gs32", "anthropic/claude-opus-4-5"]
      },
      models: {
        "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
        "lmstudio/minimax-m2.1-gs32": { alias: "MiniMax Local" },
        "anthropic/claude-opus-4-5": { alias: "Opus" }
      }
    }
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.1-gs32",
            name: "MiniMax M2.1 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

<div id="local-first-with-hosted-safety-net">
  ### 以本地为主，云端作为安全兜底
</div>

将主用和备用的顺序对调；保持相同的 providers 配置块和 `models.mode: "merge"` 设置，这样当本地机器不可用时，你可以回退到 Sonnet 或 Opus。

<div id="regional-hosting-data-routing">
  ### 区域托管 / 数据路由
</div>

* OpenRouter 上也提供托管的 MiniMax/Kimi/GLM 变体，并带有固定区域的端点（例如美国区域托管）。在那里选择对应的区域变体，可以在继续使用 `models.mode: "merge"` 实现 Anthropic/OpenAI 回退的同时，将流量限制在你选定的司法管辖区内。
* 仅本地运行依然是隐私最强的方案；当你既需要提供方功能、又希望控制数据流向时，区域托管路由是一个折中的选择。

<div id="other-openai-compatible-local-proxies">
  ## 其他兼容 OpenAI 的本地代理
</div>

如果 vLLM、LiteLLM、OAI-proxy 或自定义网关暴露了符合 OpenAI 风格的 `/v1` 端点，它们都可以正常工作。将上面的提供方配置块替换为你的端点和模型 ID：

```json5
{
  models: {
    mode: "merge",
    providers: {
      local: {
        baseUrl: "http://127.0.0.1:8000/v1",
        apiKey: "sk-local",
        api: "openai-responses",
        models: [
          {
            id: "my-local-model",
            name: "Local Model",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 120000,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

保持 `models.mode: "merge"` 不变，这样托管模型仍然可以作为备用模型使用。

<div id="troubleshooting">
  ## 故障排查
</div>

* Gateway 能访问该代理服务吗？`curl http://127.0.0.1:1234/v1/models`。
* LM Studio 中的模型被卸载了吗？请重新加载；冷启动是导致“卡住”的常见原因。
* 出现上下文相关错误？降低 `contextWindow` 或提高服务器的限制上限。
* 安全性：本地模型不会经过提供方侧的安全过滤；请尽量缩小智能体的职责范围，并开启压缩以限制提示注入的影响范围。