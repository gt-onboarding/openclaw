---
title: GLM
summary: "GLM 模型系列概览，以及如何在 OpenClaw 中使用它们"
read_when:
  - 你想在 OpenClaw 中使用 GLM 模型
  - 你需要了解模型命名规范和配置方式
---

<div id="glm-models">
  # GLM 模型
</div>

GLM 是一个可通过 Z.AI 平台使用的**模型系列**（不是公司）。在 OpenClaw 中，GLM
模型是通过 `zai` 提供方，并使用诸如 `zai/glm-4.7` 这样的模型 ID 进行访问的。

<div id="cli-setup">
  ## CLI 配置
</div>

```bash
openclaw onboard --auth-choice zai-api-key
```

<div id="config-snippet">
  ## 配置示例
</div>

```json5
{
  env: { ZAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "zai/glm-4.7" } } }
}
```

<div id="notes">
  ## 注意事项
</div>

* GLM 的版本和可用性可能会变化；请查看 Z.AI 的文档以获取最新信息。
* 模型 ID 示例包括 `glm-4.7` 和 `glm-4.6`。
* 有关该提供方的详细信息，请参阅 [/providers/zai](/zh/providers/zai)。