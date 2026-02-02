---
title: "MiniMax"
summary: "在 OpenClaw 中使用 MiniMax M2.1"
read_when:
  - 你希望在 OpenClaw 中使用 MiniMax 模型
  - 你需要 MiniMax 的设置指南
---

<div id="minimax">
  # MiniMax
</div>

MiniMax 是一家构建 **M2/M2.1** 模型家族的 AI 公司。当前
面向编码场景的版本为 **MiniMax M2.1**（2025 年 12 月 23 日），专为
真实世界的复杂任务而打造。

来源：[MiniMax M2.1 发布说明](https://www.minimax.io/news/minimax-m21)

<div id="model-overview-m21">
  ## 模型概览（M2.1）
</div>

MiniMax 强调 M2.1 带来了以下改进：

* 更强的**多语言编码**能力（Rust、Java、Go、C++、Kotlin、Objective-C、TS/JS）。
* 更出色的**网页/应用开发**支持，以及更高的美学输出质量（包括原生移动应用）。
* 改进的**复合指令**处理能力，更适用于办公类工作流，基于交错思维和集成约束执行。
* **回答更简洁**，同时降低 token 使用量并加快迭代节奏。
* 更强的**工具/智能体框架**兼容性和上下文管理能力（Claude Code、Droid/Factory AI、Cline、Kilo Code、Roo Code、BlackBox）。
* 更高质量的**对话与技术写作**输出。

<div id="minimax-m21-vs-minimax-m21-lightning">
  ## MiniMax M2.1 与 MiniMax M2.1 Lightning 对比
</div>

* **速度：**Lightning 是 MiniMax 定价文档中的「快速」版本。
* **成本：**定价显示输入单价相同，但 Lightning 的输出单价更高。
* **编码套餐路由：**Lightning 后端在 MiniMax 的编码套餐中无法直接使用。MiniMax 会将大部分请求自动路由到 Lightning，但在流量高峰时会退回到常规的 M2.1 后端。

<div id="choose-a-setup">
  ## 选择配置方案
</div>

<div id="minimax-m21-recommended">
  ### MiniMax M2.1 — 推荐使用
</div>

**最佳适用场景：** 使用具备 Anthropic 兼容 API 的托管 MiniMax。

通过 CLI 配置：

* 运行 `openclaw configure`
* 选择 **Model/auth**
* 选择 **MiniMax M2.1**

```json5
{
  env: { MINIMAX_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "minimax/MiniMax-M2.1" } } },
  models: {
    mode: "merge",
    providers: {
      minimax: {
        baseUrl: "https://api.minimax.io/anthropic",
        apiKey: "${MINIMAX_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 15, output: 60, cacheRead: 2, cacheWrite: 10 },
            contextWindow: 200000,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

<div id="minimax-m21-as-fallback-opus-primary">
  ### 将 MiniMax M2.1 作为回退（Opus 为主）
</div>

**适用场景：** 以 Opus 4.5 为主模型，在故障时自动切换到 MiniMax M2.1。

```json5
{
  env: { MINIMAX_API_KEY: "sk-..." },
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-5": { alias: "opus" },
        "minimax/MiniMax-M2.1": { alias: "minimax" }
      },
      model: {
        primary: "anthropic/claude-opus-4-5",
        fallbacks: ["minimax/MiniMax-M2.1"]
      }
    }
  }
}
```

<div id="optional-local-via-lm-studio-manual">
  ### 可选：通过 LM Studio 本地运行（手动）
</div>

**最适合：** 使用 LM Studio 进行本地推理。
在高性能硬件（例如台式机 / 服务器）上，通过 LM Studio 的本地服务器运行 MiniMax M2.1 时，我们观察到效果非常理想。

通过 `openclaw.json` 手动配置：

```json5
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.1-gs32" },
      models: { "lmstudio/minimax-m2.1-gs32": { alias: "Minimax" } }
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

<div id="configure-via-openclaw-configure">
  ## 通过 `openclaw configure` 进行配置
</div>

使用交互式配置向导来设置 MiniMax，而无需手动编辑 JSON：

1. 运行 `openclaw configure`。
2. 选择 **Model/auth**。
3. 选择 **MiniMax M2.1**。
4. 在出现提示时选择默认模型。

<div id="configuration-options">
  ## 配置选项
</div>

* `models.providers.minimax.baseUrl`：优先使用 `https://api.minimax.io/anthropic`（兼容 Anthropic）；`https://api.minimax.io/v1` 可选，用于兼容 OpenAI 的请求体。
* `models.providers.minimax.api`：优先使用 `anthropic-messages`；`openai-completions` 可选，用于兼容 OpenAI 的请求体。
* `models.providers.minimax.apiKey`：MiniMax API 密钥（`MINIMAX_API_KEY`）。
* `models.providers.minimax.models`：定义 `id`、`name`、`reasoning`、`contextWindow`、`maxTokens`、`cost`。
* `agents.defaults.models`：为你希望加入允许列表的模型创建别名。
* `models.mode`：如果你希望在内置模型之外额外添加 MiniMax，请保持设置为 `merge`。

<div id="notes">
  ## 注意事项
</div>

* 模型引用格式为 `minimax/<model>`。
* Coding Plan 用量 API：`https://api.minimaxi.com/v1/api/openplatform/coding_plan/remains`（需要 Coding Plan 密钥）。
* 如果需要精确的成本追踪，请在 `models.json` 中更新价格数值。
* MiniMax Coding Plan 推荐链接（九折优惠）：https://platform.minimax.io/subscribe/coding-plan?code=DbXJTRClnb&amp;source=link
* 有关提供方规则，请参见 [/concepts/model-providers](/zh/concepts/model-providers)。
* 使用 `openclaw models list` 和 `openclaw models set minimax/MiniMax-M2.1` 进行切换。

<div id="troubleshooting">
  ## 故障排查
</div>

<div id="unknown-model-minimaxminimax-m21">
  ### “未知模型：minimax/MiniMax-M2.1”
</div>

这通常意味着**MiniMax 提供方尚未配置**（没有提供方条目，且未找到 MiniMax 认证配置文件/环境变量键）。针对该检测的修复包含在
**2026.1.12** 版本中（在撰写本文时尚未发布）。可以通过以下方式修复：

* 升级到 **2026.1.12**（或从源码的 `main` 分支运行），然后重启 Gateway。
* 运行 `openclaw configure` 并选择 **MiniMax M2.1**，或
* 手动添加 `models.providers.minimax` 配置块，或
* 设置 `MINIMAX_API_KEY`（或配置一个 MiniMax 认证配置文件），以便该提供方可以被注入。

请确保模型 ID **区分大小写**：

* `minimax/MiniMax-M2.1`
* `minimax/MiniMax-M2.1-lightning`

然后再次检查：

```bash
openclaw models list
```
