---
title: 对话记录规范
summary: "参考：特定提供方的对话记录清洗与修复规则"
read_when:
  - 你正在调试由于对话记录结构导致的提供方请求拒绝问题
  - 你正在修改对话记录清洗或工具调用修复逻辑
  - 你正在排查不同提供方之间的工具调用 ID 不匹配问题
---

<div id="transcript-hygiene-provider-fixups">
  # 转录清理（提供方修正）
</div>

本文档描述在运行前（构建模型上下文时）对转录应用的**针对提供方的特定修正**。这些是为满足严格的提供方要求而进行的**仅在内存中的**调整，**不会**重写磁盘上存储的 JSONL 转录文件。

处理范围包括：

* 工具调用 ID 规范化
* 工具结果配对修复
* 对话轮次校验 / 排序
* 思考签名清理
* 图像负载净化

若需要了解转录存储的详细信息，请参见：

* [/reference/session-management-compaction](/zh/reference/session-management-compaction)

***

<div id="where-this-runs">
  ## 运行位置
</div>

所有转录清理逻辑都集中在嵌入式运行器中：

* 策略选择：`src/agents/transcript-policy.ts`
* 清理/修复执行：`src/agents/pi-embedded-runner/google.ts` 中的 `sanitizeSessionHistory`

该策略使用 `provider`、`modelApi` 和 `modelId` 来决定应应用哪些处理。

***

<div id="global-rule-image-sanitization">
  ## 全局规则：图像规范化处理
</div>

图像负载始终会经过规范化处理，以避免因大小限制（对过大的 base64 图像进行缩放/重新压缩）而在提供方端被拒绝。

实现：

* `sanitizeSessionMessagesImages` 位于 `src/agents/pi-embedded-helpers/images.ts`
* `sanitizeContentBlocksImages` 位于 `src/agents/tool-images.ts`

***

<div id="provider-matrix-current-behavior">
  ## 提供方矩阵（当前行为）
</div>

**OpenAI / OpenAI Codex**

* 仅进行图像清理。
* 在切换到 OpenAI Responses/Codex 模型时，丢弃孤立推理签名（即后面没有内容块的独立推理项）。
* 不进行工具调用 ID 清理。
* 不进行工具结果配对修复。
* 不进行轮次验证或重排。
* 不生成合成工具结果。
* 不移除思考签名。

**Google（Generative AI / Gemini CLI / Antigravity）**

* 工具调用 ID 清理：严格限制为字母数字字符。
* 工具结果配对修复并生成合成工具结果。
* 轮次验证（Gemini 风格轮次交替）。
* Google 轮次顺序修正（当历史记录以 assistant 开头时，前置一个极小的 user 引导消息）。
* Antigravity Claude：规范化思考签名；丢弃未签名的思考块。

**Anthropic / Minimax（Anthropic 兼容）**

* 工具结果配对修复并生成合成工具结果。
* 轮次验证（合并连续的 user 轮次以满足严格交替约束）。

**Mistral（包括基于 model-id 的检测）**

* 工具调用 ID 清理：strict9（长度为 9 的字母数字字符串）。

**OpenRouter Gemini**

* 思考签名清理：移除非 base64 的 `thought_signature` 值（保留 base64）。

**其他所有提供方**

* 仅进行图像清理。

***

<div id="historical-behavior-pre-2026122">
  ## 历史行为（2026.1.22 之前）
</div>

在 2026.1.22 版本发布之前，OpenClaw 会对对话转录应用多层“清理”处理：

* 在每次构建上下文时都会运行一个 **transcript-sanitize 扩展**，它可以：
  * 修复工具调用与结果的配对关系。
  * 清理工具调用 id（包括一个会保留 `_`/`-` 的非严格模式）。
* runner 还会执行针对不同提供方的专用清理逻辑，造成了重复工作。
* 在提供方策略之外还会进行额外的修改操作，包括：
  * 在持久化之前，从 assistant 文本中移除 `<final>` 标签。
  * 丢弃空的 assistant 错误轮次。
  * 在工具调用之后裁剪 assistant 内容。

这种复杂性导致了跨提供方的回归问题（尤其是 `openai-responses`
`call_id|fc_id` 配对）。2026.1.22 的清理重构移除了该扩展，在 runner 中集中实现相关逻辑，并让 OpenAI 除了图像清理之外保持 **no-touch**（不再额外改写）。