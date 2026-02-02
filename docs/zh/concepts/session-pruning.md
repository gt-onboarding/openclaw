---
title: 会话裁剪
summary: "会话裁剪：通过裁剪工具结果来减少上下文膨胀"
read_when:
  - 你希望减少由于工具输出导致的 LLM 上下文增长
  - 你正在调整 agents.defaults.contextPruning 参数
---

<div id="session-pruning">
  # 会话修剪
</div>

会话修剪会在每次调用 LLM 之前，从内存中的上下文里删除**旧的工具结果**。它**不会**修改磁盘上的会话历史（`*.jsonl`）。

<div id="when-it-runs">
  ## 何时运行
</div>

* 当启用 `mode: "cache-ttl"` 且该会话上次对 Anthropic 的调用早于 `ttl` 时。
* 只影响该请求中发送给模型的消息。
* 仅对 Anthropic API 调用（以及 OpenRouter Anthropic 模型）生效。
* 为获得最佳效果，请将 `ttl` 与模型的 `cacheControlTtl` 保持一致。
* 剪枝完成后，TTL 窗口会重置，因此后续请求会继续使用缓存，直到 `ttl` 再次过期。

<div id="smart-defaults-anthropic">
  ## 智能默认值（Anthropic）
</div>

* 使用 **OAuth 或 setup-token** 的配置：启用 `cache-ttl` 清理，并将心跳间隔设置为 `1h`。
* 使用 **API key** 的配置：启用 `cache-ttl` 清理，将心跳间隔设置为 `30m`，并将 Anthropic 模型上的 `cacheControlTtl` 默认设为 `1h`。
* 如果你显式设置了这些值中的任一项，OpenClaw **不会** 覆盖它们。

<div id="what-this-improves-cost-cache-behavior">
  ## 这样能改善什么（成本 + 缓存行为）
</div>

* **为什么要裁剪：**Anthropic 的 prompt 缓存只在 TTL 内生效。如果一个会话长时间空闲，超过 TTL，下一次请求会重新缓存完整 prompt，除非你先把它裁剪掉。
* **哪里会更省钱：**裁剪会减小 TTL 过期后第一次请求的 **cacheWrite** 大小。
* **为什么 TTL 重置很重要：**一旦裁剪执行，缓存窗口会重置，因此后续请求可以复用刚刚缓存好的 prompt，而不是再次缓存完整历史。
* **它不会做什么：**裁剪不会增加 token 数量，也不会让成本“翻倍”；它只会改变在 TTL 过期后的第一次请求中，哪些内容会被缓存。

<div id="what-can-be-pruned">
  ## 可以被裁剪的内容
</div>

* 只会裁剪 `toolResult` 类型的消息。
* 用户和助手消息**绝不会**被修改。
* 最后 `keepLastAssistants` 条助手消息是受保护的；在该截断点之后的工具结果不会被裁剪。
* 如果助手消息不足以确定截断点，则跳过裁剪。
* 包含**图像块**的工具结果会被跳过（永不裁剪或清除）。

<div id="context-window-estimation">
  ## 上下文窗口估算
</div>

裁剪会使用一个估算的上下文窗口（字符数 ≈ token 数 × 4）。窗口大小按如下顺序确定：

1. 模型定义中的 `contextWindow`（来自模型注册表）。
2. `models.providers.*.models[].contextWindow` 覆盖值。
3. `agents.defaults.contextTokens`。
4. 默认 `200000` token。

<div id="mode">
  ## 模式
</div>

<div id="cache-ttl">
  ### cache-ttl
</div>

* 仅当上一次 Anthropic 调用早于设定的 `ttl`（默认 `5m`）时才会执行清理。
* 运行时：行为与之前相同，仍为软裁剪 + 硬清除。

<div id="soft-vs-hard-pruning">
  ## 软裁剪 vs 硬清理
</div>

* **Soft-trim（软裁剪）**：仅用于过大的工具输出。
  * 保留开头和结尾，中间插入 `...`，并在末尾附上一条包含原始大小的说明。
  * 跳过包含图像块的结果。
* **Hard-clear（硬清理）**：将整个工具输出替换为 `hardClear.placeholder`。

<div id="tool-selection">
  ## 工具选择
</div>

* `tools.allow` / `tools.deny` 支持 `*` 通配符。
* 拒绝规则优先。
* 匹配不区分大小写。
* 允许列表为空 =&gt; 所有工具均被允许。

<div id="interaction-with-other-limits">
  ## 与其他限制的交互
</div>

* 内置工具已经会截断自己的输出；会话修剪是额外的一层，用于防止长时间运行的会话在模型上下文中累积过多的工具输出。
* 压缩是独立的：压缩会对内容进行总结并持久化存储，而修剪则是按请求进行的临时操作。参见 [/concepts/compaction](/zh/concepts/compaction)。

<div id="defaults-when-enabled">
  ## 默认值（启用时）
</div>

* `ttl`: `"5m"`
* `keepLastAssistants`: `3`
* `softTrimRatio`: `0.3`
* `hardClearRatio`: `0.5`
* `minPrunableToolChars`: `50000`
* `softTrim`: `{ maxChars: 4000, headChars: 1500, tailChars: 1500 }`
* `hardClear`: `{ enabled: true, placeholder: "[旧工具结果内容已清除]" }`

<div id="examples">
  ## 示例
</div>

默认（关闭）：

```json5
{
  agent: {
    contextPruning: { mode: "off" }
  }
}
```

启用支持 TTL 的清理：

```json5
{
  agent: {
    contextPruning: { mode: "cache-ttl", ttl: "5m" }
  }
}
```

仅对特定工具进行修剪：

```json5
{
  agent: {
    contextPruning: {
      mode: "cache-ttl",
      tools: { allow: ["exec", "read"], deny: ["*image*"] }
    }
  }
}
```

请参阅配置参考：[Gateway 配置](/zh/gateway/configuration)
