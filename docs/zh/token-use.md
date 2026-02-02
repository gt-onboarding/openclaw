---
title: Token 使用
summary: "OpenClaw 如何构建提示上下文并报告 Token 使用量和成本"
read_when:
  - 解释 Token 使用情况、成本或上下文窗口
  - 调试上下文增长或压缩行为
---

<div id="token-use-costs">
  # Token 使用情况与成本
</div>

OpenClaw 跟踪的是 **token** 数量，而不是字符数。Token 的定义因模型而异，但大多数
OpenAI 风格的模型在英文文本中平均大约每个 token 对应 4 个字符。

<div id="how-the-system-prompt-is-built">
  ## 系统提示词是如何构建的
</div>

OpenClaw 在每次运行时都会组装自己的系统提示词。它包含：

* 工具列表 + 简短描述
* 技能列表（仅元数据；说明会在需要时使用 `read` 动态加载）
* 自更新指令
* 工作区 + 引导文件（`AGENTS.md`、`SOUL.md`、`TOOLS.md`、`IDENTITY.md`、`USER.md`、`HEARTBEAT.md`、`BOOTSTRAP.md`（新建时））。大文件会被 `agents.defaults.bootstrapMaxChars` 截断（默认：20000）。
* 时间（UTC + 用户所在时区）
* 回复标签 + 心跳行为
* 运行时元数据（主机/操作系统/模型/思考状态）

完整拆解见 [系统提示词](/zh/concepts/system-prompt)。

<div id="what-counts-in-the-context-window">
  ## 上下文窗口中会计入哪些内容
</div>

模型接收到的所有内容都会计入上下文上限：

* System prompt（上面列出的所有部分）
* 对话历史（用户 + 助手消息）
* 工具调用和工具结果
* 附件/转录内容（图像、音频、文件）
* 压缩摘要和裁剪产生的内容
* 提供方包装层或安全头信息（对用户不可见，但仍会被计入）

若要查看实际占用明细（按注入的文件、工具、技能以及 System prompt 大小），使用 `/context list` 或 `/context detail`。参见 [上下文](/zh/concepts/context)。

<div id="how-to-see-current-token-usage">
  ## 如何查看当前 token 使用情况
</div>

在聊天中使用以下命令：

* `/status` → **富表情状态卡片**，显示会话模型、上下文用量、
  上一次回复的输入/输出 token，以及**预估费用**（仅限 API key）。
* `/usage off|tokens|full` → 在每条回复后附加**按回复统计的使用信息页脚**。
  * 在每个会话内持久保存该设置（存储为 `responseUsage`）。
  * 使用 OAuth 认证时会**隐藏费用**（仅显示 token）。
* `/usage cost` → 基于 OpenClaw 会话日志显示本地费用汇总。

其他界面：

* **TUI/Web TUI：** 支持 `/status` 和 `/usage`。
* **CLI：** `openclaw status --usage` 和 `openclaw channels list` 会显示
  提供方配额窗口（不是按回复的费用）。

<div id="cost-estimation-when-shown">
  ## 成本预估（如有显示）
</div>

成本是基于你的模型定价配置估算的：

```
models.providers.<provider>.models[].cost
```

这些价格以美元 (USD) 计，按 **每 100 万 tokens** 计算，适用于 `input`、`output`、`cacheRead` 和 `cacheWrite`。如果价格信息缺失，OpenClaw 只会显示 token 数量。OAuth tokens 永远不会显示对应的美元成本。

<div id="cache-ttl-and-pruning-impact">
  ## 缓存 TTL 与修剪的影响
</div>

提供方的提示缓存仅在缓存 TTL 窗口内生效。OpenClaw 可以选择启用
**cache-ttl 修剪**：一旦缓存 TTL 过期，就会修剪会话，然后重置缓存窗口，
这样后续请求就可以复用新近缓存的上下文，而不是重新缓存完整历史。这样可以在
会话在超过 TTL 后进入空闲时，降低缓存写入成本。

在 [Gateway configuration](/zh/gateway/configuration) 中进行配置，并在
[Session pruning](/zh/concepts/session-pruning) 中查看具体行为细节。

心跳可以在空闲间隙期间保持缓存**处于“温热（warm）”状态**。如果你的模型缓存 TTL 为 `1h`，
将心跳间隔设置为略低于该值（例如 `55m`），可以避免重新缓存完整提示，从而降低
缓存写入成本。

对于 Anthropic API 的定价，缓存读取比输入 token 便宜得多，而缓存写入则按更高
的倍数计费。参阅 Anthropic 的提示缓存定价以获取最新费率和 TTL 倍数：
https://docs.anthropic.com/docs/build-with-claude/prompt-caching

<div id="example-keep-1h-cache-warm-with-heartbeat">
  ### 示例：使用心跳保持 1 小时缓存处于热状态
</div>

```yaml
agents:
  defaults:
    model:
      primary: "anthropic/claude-opus-4-5"
    models:
      "anthropic/claude-opus-4-5":
        params:
          cacheControlTtl: "1h"
    heartbeat:
      every: "55m"
```

<div id="tips-for-reducing-token-pressure">
  ## 减少 token 压力的技巧
</div>

* 使用 `/compact` 来压缩/总结较长的会话。
* 在工作流中截断体积较大的工具输出。
* 保持技能描述简短（技能列表会被注入到提示中）。
* 在冗长的探索性工作中优先选择较小的模型。

参见 [Skills](/zh/tools/skills) 了解技能列表带来的额外开销的具体公式。