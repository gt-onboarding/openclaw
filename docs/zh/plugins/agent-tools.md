---
title: Agent 工具
summary: "在插件中编写 Agent 工具（schema、可选工具、允许列表）"
read_when:
  - 你想在插件中添加一个新的 Agent 工具
  - 你需要通过允许列表将某个工具配置为用户主动启用（opt-in）
---

<div id="plugin-agent-tools">
  # 插件 Agent 工具
</div>

OpenClaw 插件可以注册 **Agent 工具**（JSON‑schema 函数），在智能体运行时提供给 LLM 使用。工具可以是 **必需的**（始终可用）或 **可选的**（按需启用）。

Agent 工具可以在主配置中的 `tools` 下配置，或在每个智能体的 `agents.list[].tools` 中单独配置。allowlist/denylist 策略用于控制智能体可以调用哪些工具。

<div id="basic-tool">
  ## 基础工具
</div>

```ts
import { Type } from "@sinclair/typebox";

export default function (api) {
  api.registerTool({
    name: "my_tool",
    description: "Do a thing",
    parameters: Type.Object({
      input: Type.String(),
    }),
    async execute(_id, params) {
      return { content: [{ type: "text", text: params.input }] };
    },
  });
}
```


<div id="optional-tool-optin">
  ## 可选工具（按需启用）
</div>

可选工具**永远不会**被自动启用。用户必须将它们显式添加到智能体的允许列表中。

```ts
export default function (api) {
  api.registerTool(
    {
      name: "workflow_tool",
      description: "Run a local workflow",
      parameters: {
        type: "object",
        properties: {
          pipeline: { type: "string" },
        },
        required: ["pipeline"],
      },
      async execute(_id, params) {
        return { content: [{ type: "text", text: params.pipeline }] };
      },
    },
    { optional: true },
  );
}
```

在 `agents.list[].tools.allow`（或全局 `tools.allow`）中启用可选的工具：

```json5
{
  agents: {
    list: [
      {
        id: "main",
        tools: {
          allow: [
            "workflow_tool",  // 特定工具名称
            "workflow",       // 插件 ID(启用该插件的所有工具)
            "group:plugins"   // 所有插件工具
          ]
        }
      }
    ]
  }
}
```

影响工具可用性的其他配置项：

* 只包含插件工具名称的允许列表会被视为对插件工具的按需启用；核心工具仍然
  保持启用状态，除非你也在允许列表中包含核心工具或核心工具分组。
* `tools.profile` / `agents.list[].tools.profile`（基础允许列表）
* `tools.byProvider` / `agents.list[].tools.byProvider`（特定提供方的允许/拒绝）
* `tools.sandbox.tools.*`（启用沙箱时的沙箱工具策略）


<div id="rules-tips">
  ## 规则与提示
</div>

- 工具名称**不得**与核心工具名称冲突；发生冲突的工具会被跳过。
- 在允许列表中使用的插件 ID 不得与核心工具名称冲突。
- 对会触发副作用或需要额外二进制文件/凭证的工具，优先将其设置为 `optional: true`。