---
title: LLM 任务
summary: "面向工作流的仅 JSON LLM 任务（可选插件工具）"
read_when:
  - 当你想在工作流中添加仅输出 JSON 的 LLM 步骤时
  - 当你需要用于自动化的、经过模式校验的 LLM 输出时
---

<div id="llm-task">
  # LLM 任务
</div>

`llm-task` 是一个**可选的插件工具**，用于运行仅处理 JSON 的 LLM 任务，并
返回结构化输出（并可选择根据 JSON Schema 进行校验）。

这非常适合像 Lobster 这样的工作流引擎：你可以只添加一个 LLM 步骤，
而无需为每个工作流编写自定义的 OpenClaw 代码。

<div id="enable-the-plugin">
  ## 启用插件
</div>

1. 启用插件：

```json
{
  "plugins": {
    "entries": {
      "llm-task": { "enabled": true }
    }
  }
}
```

2. 将此工具加入允许列表（它被注册为 `optional: true`）：

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "tools": { "allow": ["llm-task"] }
      }
    ]
  }
}
```


<div id="config-optional">
  ## 配置（可选）
</div>

```json
{
  "plugins": {
    "entries": {
      "llm-task": {
        "enabled": true,
        "config": {
          "defaultProvider": "openai-codex",
          "defaultModel": "gpt-5.2",
          "defaultAuthProfileId": "main",
          "allowedModels": ["openai-codex/gpt-5.2"],
          "maxTokens": 800,
          "timeoutMs": 30000
        }
      }
    }
  }
}
```

`allowedModels` 是一个由 `provider/model` 字符串组成的允许列表。配置该字段后，任何不在该列表中的请求都会被拒绝。


<div id="tool-parameters">
  ## 工具参数
</div>

- `prompt`（字符串，必需）
- `input`（任意类型，可选）
- `schema`（对象，可选，JSON Schema）
- `provider`（字符串，可选）
- `model`（字符串，可选）
- `authProfileId`（字符串，可选）
- `temperature`（数值，可选）
- `maxTokens`（数值，可选）
- `timeoutMs`（数值，可选）

<div id="output">
  ## 输出
</div>

返回包含已解析 JSON 的 `details.json`（在提供 `schema` 时会进行校验）。

<div id="example-lobster-workflow-step">
  ## Lobster 工作流步骤示例
</div>

```lobster
openclaw.invoke --tool llm-task --action json --args-json '{
  "prompt": "Given the input email, return intent and draft.",
  "input": {
    "subject": "Hello",
    "body": "Can you help?"
  },
  "schema": {
    "type": "object",
    "properties": {
      "intent": { "type": "string" },
      "draft": { "type": "string" }
    },
    "required": ["intent", "draft"],
    "additionalProperties": false
  }
}'
```


<div id="safety-notes">
  ## 安全注意事项
</div>

- 该工具**仅使用 JSON**，并强制模型只输出 JSON（不允许使用
  代码块，也不允许添加任何说明性文字）。
- 本次运行不会向模型暴露任何工具。
- 将输出视为不可信数据，除非你通过 `schema` 进行了验证。
- 在执行任何会产生副作用的操作（send、post、exec）之前，必须先获得批准。