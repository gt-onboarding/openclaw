---
title: 入门配置协议
summary: "用于入门向导和配置架构的 RPC 协议说明"
read_when: "更改入门向导步骤或配置架构端点时阅读"
---

<div id="onboarding-config-protocol">
  # 上手与配置协议
</div>

目的：在 CLI、macOS 应用和 Web UI 间共享统一的上手与配置界面。

<div id="components">
  ## 组件
</div>

- 向导引擎（共享会话 + 提示词 + 引导状态）。
- CLI 引导采用与 UI 客户端相同的向导流程。
- Gateway RPC 暴露向导和配置 schema 端点。
- macOS 引导采用向导步骤模型。
- Web UI 基于 JSON Schema 和 UI 提示渲染配置表单。

<div id="gateway-rpc">
  ## Gateway RPC
</div>

- `wizard.start` 参数：`{ mode?: "local"|"remote", workspace?: string }`
- `wizard.next` 参数：`{ sessionId, answer?: { stepId, value? } }`
- `wizard.cancel` 参数：`{ sessionId }`
- `wizard.status` 参数：`{ sessionId }`
- `config.schema` 参数：`{}`

响应（结构）

- 向导：`{ sessionId, done, step?, status?, error? }`
- 配置 schema：`{ schema, uiHints, version, generatedAt }`

<div id="ui-hints">
  ## UI 提示
</div>

- `uiHints` 以路径为键；包含可选元数据（label/help/group/order/advanced/sensitive/placeholder）。
- 敏感字段渲染为密码输入；不提供额外的脱敏层。
- 不受支持的 schema 节点会回退为原始 JSON 编辑器。

<div id="notes">
  ## 说明
</div>

- 本文档是跟踪 Onboarding/配置 协议重构的唯一权威来源。