---
title: 清单
summary: "插件清单 + JSON Schema 要求（严格的配置校验）"
read_when:
  - 你正在构建 OpenClaw 插件
  - 你需要发布插件配置 Schema，或者调试插件校验错误
---

<div id="plugin-manifest-openclawpluginjson">
  # 插件清单（openclaw.plugin.json）
</div>

每个插件**必须**在其**插件根目录**中包含一个 `openclaw.plugin.json` 文件。
OpenClaw 使用该清单在**不执行插件代码**的情况下验证配置。缺失或无效的清单会被视为插件错误，并导致配置校验无法通过。

查看完整的插件系统指南：[插件](/zh/plugin)。

<div id="required-fields">
  ## 必填字段
</div>

```json
{
  "id": "voice-call",
  "configSchema": {
    "type": "object",
    "additionalProperties": false,
    "properties": {}
  }
}
```

必需键：

* `id` (string)：插件的规范 ID。
* `configSchema` (object)：插件配置的 JSON Schema（内联）。

可选键：

* `kind` (string)：插件类型（示例：`"memory"`）。
* `channels` (array)：由此插件注册的通道 ID（示例：`["matrix"]`）。
* `providers` (array)：由此插件注册的提供方 ID。
* `skills` (array)：要加载的技能目录（相对于插件根目录）。
* `name` (string)：插件的显示名称。
* `description` (string)：插件的简短概述。
* `uiHints` (object)：用于 UI 渲染的配置字段标签/占位符/敏感标记。
* `version` (string)：插件版本（仅供参考）。

<div id="json-schema-requirements">
  ## JSON Schema 要求
</div>

* **每个插件都必须提供一个 JSON Schema**，即使它不需要任何配置。
* 允许使用空 Schema（例如，`{ "type": "object", "additionalProperties": false }`）。
* Schema 会在配置读/写时进行验证，而不是在运行时。

<div id="validation-behavior">
  ## 验证行为
</div>

* 未知的 `channels.*` 键会被视为**错误**，除非该 channel ID 由某个插件清单声明。
* `plugins.entries.<id>`、`plugins.allow`、`plugins.deny` 和 `plugins.slots.*`
  必须引用**可发现的**插件 ID。未知 ID 会被视为**错误**。
* 如果插件已安装，但其清单或 schema 损坏或缺失，则验证失败，Doctor 会报告该插件错误。
* 如果存在插件配置但该插件被**禁用**，则会保留该配置，并在 Doctor 和日志中生成**警告**。

<div id="notes">
  ## 备注
</div>

* 清单对于**所有插件**都是必需的，包括本地文件系统加载。
* 运行时仍会单独加载插件模块；清单仅用于
  发现 + 验证。
* 如果您的插件依赖于原生模块，请记录构建步骤以及任何
  包管理器的允许列表要求（例如，pnpm `allow-build-scripts`
  * `pnpm rebuild <package>`）。