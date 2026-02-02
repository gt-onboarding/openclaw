---
title: 模型
summary: "Models CLI：列出、设置、别名、回退、扫描、状态"
read_when:
  - 添加或修改 models CLI（models list/set/scan/aliases/fallbacks）
  - 更改模型回退策略或模型选择 UX
  - 更新模型扫描探测（tools/images）
---

<div id="models-cli">
  # 模型 CLI
</div>

关于认证配置文件轮换、冷却时间，以及它们如何与回退策略配合工作的说明，请参见 [/concepts/model-failover](/zh/concepts/model-failover)。
提供方的简要概览和示例：[/concepts/model-providers](/zh/concepts/model-providers)。

<div id="how-model-selection-works">
  ## 模型选择的工作方式
</div>

OpenClaw 按如下顺序选择模型：

1. **主模型**（`agents.defaults.model.primary` 或 `agents.defaults.model`）。
2. `agents.defaults.model.fallbacks` 中的 **回退模型**（按顺序）。
3. **提供方认证故障切换** 会在同一提供方内部完成，然后才会切换到下一个模型。

相关说明：

* `agents.defaults.models` 是 OpenClaw 可以使用的模型允许列表/目录（外加别名）。
* `agents.defaults.imageModel` **仅在**主模型无法接受图像时才会被使用。
* 每个智能体的默认值可以通过 `agents.list[].model` 加上绑定来覆盖 `agents.defaults.model`（参见 [/concepts/multi-agent](/zh/concepts/multi-agent)）。

<div id="quick-model-picks-anecdotal">
  ## 快速模型推荐（经验性）
</div>

* **GLM**：在写代码/调用工具方面稍微更强一些。
* **MiniMax**：更适合写作和整体氛围/体验。

<div id="setup-wizard-recommended">
  ## 设置向导（推荐）
</div>

如果你不想手动编辑配置文件，可以运行初始化向导：

```bash
openclaw onboard
```

它可以为常见的提供方设置模型和认证，包括 **OpenAI Code（Codex）
订阅**（OAuth）和 **Anthropic**（建议使用 API 密钥；也支持 `claude
setup-token`）。

<div id="config-keys-overview">
  ## 配置键（概览）
</div>

* `agents.defaults.model.primary` 和 `agents.defaults.model.fallbacks`
* `agents.defaults.imageModel.primary` 和 `agents.defaults.imageModel.fallbacks`
* `agents.defaults.models`（允许列表 + 别名 + 提供方参数）
* `models.providers`（写入 `models.json` 的自定义提供方）

模型引用会被规范化为小写。像 `z.ai/*` 这样的提供方别名会被规范化为
`zai/*`。

提供方配置示例（包括 OpenCode Zen）位于
[/gateway/configuration](/zh/gateway/configuration#opencode-zen-multi-model-proxy) 中。

<div id="model-is-not-allowed-and-why-replies-stop">
  ## “模型不被允许使用”（以及为什么回复会中断）
</div>

如果设置了 `agents.defaults.models`，它就会成为 `/model` 和
会话覆盖配置所使用的**允许列表**。当用户选择了一个不在该允许列表中的模型时，
OpenClaw 会返回：

```
Model "provider/model" is not allowed. Use /model to list available models.
```

这种情况发生在生成正常回复**之前**，因此这条消息可能会让人感觉好像“没有回应”。可以通过以下任一方式解决：

* 将该模型添加到 `agents.defaults.models` 中，或
* 清空允许列表（移除 `agents.defaults.models`），或
* 从 `/model list` 中选择一个模型。

允许列表配置示例：

```json5
{
  agent: {
    model: { primary: "anthropic/claude-sonnet-4-5" },
    models: {
      "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
      "anthropic/claude-opus-4-5": { alias: "Opus" }
    }
  }
}
```

<div id="switching-models-in-chat-model">
  ## 在聊天中切换模型（`/model`）
</div>

你可以在不重启的情况下为当前会话切换模型：

```
/model
/model list
/model 3
/model openai/gpt-5.2
/model status
```

说明：

* `/model`（以及 `/model list`）是一个紧凑的、带编号的选择器（模型系列 + 可用提供方）。
* `/model <#>` 从该选择器中选取对应项。
* `/model status` 是详细视图（认证候选项，以及在已配置时，提供方端点的 `baseUrl` + `api` 模式）。
* 模型引用通过在**第一个** `/` 处分割进行解析。输入 `/model <ref>` 时使用 `provider/model` 格式。
* 如果模型 ID 本身包含 `/`（OpenRouter 风格），你必须包含提供方前缀（示例：`/model openrouter/moonshotai/kimi-k2`）。
* 如果你省略提供方，OpenClaw 会将输入视为别名或**默认提供方**的模型（仅当模型 ID 中没有 `/` 时有效）。

完整命令行为/配置：[Slash commands](/zh/tools/slash-commands)。

<div id="cli-commands">
  ## CLI 命令
</div>

```bash
openclaw models list
openclaw models status
openclaw models set <provider/model>
openclaw models set-image <provider/model>

openclaw models aliases list
openclaw models aliases add <alias> <provider/model>
openclaw models aliases remove <alias>

openclaw models fallbacks list
openclaw models fallbacks add <provider/model>
openclaw models fallbacks remove <provider/model>
openclaw models fallbacks clear

openclaw models image-fallbacks list
openclaw models image-fallbacks add <provider/model>
openclaw models image-fallbacks remove <provider/model>
openclaw models image-fallbacks clear
```

`openclaw models`（不带子命令）是 `models status` 的快捷方式。

<div id="models-list">
  ### `models list`
</div>

默认显示已配置的模型。常用选项：

* `--all`: 显示全部模型列表
* `--local`: 仅显示本地提供方
* `--provider <name>`: 按提供方过滤
* `--plain`: 每行一个模型
* `--json`: 机器可解析输出

<div id="models-status">
  ### `models status`
</div>

显示解析后的主模型、备用模型、图像模型，以及已配置提供方的认证概览。
它还会展示在认证存储中找到的 OAuth 配置文件的过期状态（默认在到期前 24 小时内发出警告）。`--plain` 仅输出解析后的主模型。
OAuth 状态始终会显示（并包含在 `--json` 输出中）。如果某个已配置的提供方没有凭据，`models status` 会打印一个 **Missing auth**（缺失认证）部分。
JSON 会包含 `auth.oauth`（预警时间窗口 + 配置文件）以及 `auth.providers`（每个提供方的有效认证）。
在自动化场景中使用 `--check`（当认证缺失/已过期时退出状态码为 `1`，即将过期时为 `2`）。

推荐的 Anthropic 认证方式是使用 Claude Code CLI 的 setup-token（可在任意位置运行；如有需要，在 Gateway 主机上粘贴生成的令牌）：

```bash
claude setup-token
openclaw models status
```

<div id="scanning-openrouter-free-models">
  ## 扫描（OpenRouter 免费模型）
</div>

`openclaw models scan` 会检查 OpenRouter 的**免费模型目录**，并且可以选择性地探测模型对工具和图像的支持情况。

关键参数：

* `--no-probe`：跳过在线探测（仅获取元数据）
* `--min-params <b>`：最小参数规模（以十亿为单位）
* `--max-age-days <days>`：跳过较旧的模型
* `--provider <name>`：提供方前缀过滤
* `--max-candidates <n>`：回退候选列表大小
* `--set-default`：将 `agents.defaults.model.primary` 设为第一个选择结果
* `--set-image`：将 `agents.defaults.imageModel.primary` 设为第一个图像模型选择结果

进行探测需要 OpenRouter API 密钥（来自认证配置文件或环境变量 `OPENROUTER_API_KEY`）。如果没有密钥，使用 `--no-probe` 仅列出候选模型。

扫描结果按以下顺序排序：

1. 图像支持情况
2. 工具延迟
3. 上下文长度
4. 参数数量

输入

* OpenRouter `/models` 列表（过滤 `:free`）
* 需要在认证配置文件中配置 OpenRouter API 密钥或通过环境变量 `OPENROUTER_API_KEY` 提供（参见 [/environment](/zh/environment)）
* 可选过滤器：`--max-age-days`、`--min-params`、`--provider`、`--max-candidates`
* 探测控制参数：`--timeout`、`--concurrency`

在 TTY 中运行时，你可以以交互方式选择回退模型。在非交互模式下，传入 `--yes` 以接受默认值。

<div id="models-registry-modelsjson">
  ## 模型注册表（`models.json`）
</div>

`models.providers` 中的自定义提供方会写入到智能体目录下的 `models.json` 文件（默认路径为 `~/.openclaw/agents/<agentId>/models.json`）。在默认情况下，此文件会被合并，除非将 `models.mode` 设置为 `replace`。