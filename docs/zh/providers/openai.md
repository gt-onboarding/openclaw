---
title: OpenAI
summary: "在 OpenClaw 中通过 API 密钥或 Codex 订阅使用 OpenAI"
read_when:
  - 你想在 OpenClaw 中使用 OpenAI 模型
  - 你想通过 Codex 订阅进行认证，而不是使用 API 密钥
---

<div id="openai">
  # OpenAI
</div>

OpenAI 为 GPT 模型提供开发者 API。Codex 支持使用 **ChatGPT 登录** 获取订阅访问权限，或使用 **API key** 登录获取按用量计费的访问权限。Codex 云端要求使用 ChatGPT 登录。

<div id="option-a-openai-api-key-openai-platform">
  ## 选项 A：OpenAI API 密钥（OpenAI 平台）
</div>

**适用于：** 直接通过 API 访问并按用量计费的场景。\
在 OpenAI 控制台获取你的 API 密钥。

<div id="cli-setup">
  ### CLI 配置
</div>

```bash
openclaw onboard --auth-choice openai-api-key
# 或者使用非交互式方式
openclaw onboard --openai-api-key "$OPENAI_API_KEY"
```

<div id="config-snippet">
  ### 配置示例片段
</div>

```json5
{
  env: { OPENAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "openai/gpt-5.2" } } }
}
```

<div id="option-b-openai-code-codex-subscription">
  ## 选项 B：OpenAI Code（Codex）订阅
</div>

**最适用于：** 通过 ChatGPT/Codex 订阅使用服务，而不是使用 API 密钥。
Codex 云服务需要通过 ChatGPT 登录，而 Codex CLI 支持通过 ChatGPT 或 API 密钥登录。

<div id="cli-setup">
  ### CLI 配置
</div>

```bash
# 在向导中运行 Codex OAuth
openclaw onboard --auth-choice openai-codex

# Or run OAuth directly
openclaw models auth login --provider openai-codex
```

### 配置示例

```json5
{
  agents: { defaults: { model: { primary: "openai-codex/gpt-5.2" } } }
}
```

<div id="notes">
  ## 注意事项
</div>

* 模型引用始终使用 `provider/model`（参见 [/concepts/models](/zh/concepts/models)）。
* 认证配置和复用规则详见 [/concepts/oauth](/zh/concepts/oauth)。