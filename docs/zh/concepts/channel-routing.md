---
title: 渠道路由
summary: "各渠道的路由规则（WhatsApp、Telegram、Discord、Slack）及共享上下文"
read_when:
  - 更改渠道路由或收件箱行为
---

<div id="channels-routing">
  # 渠道与路由
</div>

OpenClaw 会将回复**路由回消息来源的同一渠道**。模型本身不会选择渠道；路由是确定性的，由宿主配置加以控制。

<div id="key-terms">
  ## 关键术语
</div>

* **Channel**：`whatsapp`、`telegram`、`discord`、`slack`、`signal`、`imessage`、`webchat`。
* **AccountId**：按 Channel 划分的账号实例（在该 Channel 支持的前提下）。
* **AgentId**：一个隔离的工作区 + 会话存储（“大脑”）。
* **SessionKey**：用于存储上下文并控制并发的桶键。

<div id="session-key-shapes-examples">
  ## 会话键结构（示例）
</div>

私信会折叠到该智能体的**主**会话中：

* `agent:<agentId>:<mainKey>`（默认：`agent:main:main`）

群组和频道按频道维度相互隔离：

* 群组：`agent:<agentId>:<channel>:group:<id>`
* 频道/房间：`agent:<agentId>:<channel>:channel:<id>`

线程：

* Slack/Discord 线程会在基础键后追加 `:thread:<threadId>`。
* Telegram 论坛话题会将 `:topic:<topicId>` 嵌入到群组键中。

示例：

* `agent:main:telegram:group:-1001234567890:topic:42`
* `agent:main:discord:channel:123456:thread:987654`

<div id="routing-rules-how-an-agent-is-chosen">
  ## 路由规则（如何选择智能体）
</div>

路由会为每条收到的消息选择 **一个智能体**：

1. **精确对端匹配**（`bindings` 中的 `peer.kind` + `peer.id`）。
2. **Guild 匹配**（Discord）通过 `guildId`。
3. **Team 匹配**（Slack）通过 `teamId`。
4. **账号匹配**（channel 上的 `accountId`）。
5. **渠道匹配**（该渠道上的任意账号）。
6. **默认智能体**（`agents.list[].default`；否则列表中的第一个条目；再否则回退到 `main`）。

匹配到的智能体决定要使用哪个工作区和会话存储。

<div id="broadcast-groups-run-multiple-agents">
  ## 广播组（运行多个 Agent）
</div>

广播组允许你在 **OpenClaw 通常会回复的时机**，为同一个会话对象同时运行 **多个智能体**（例如：在 WhatsApp 群组中、经过提及/激活门控之后）。

配置：

```json5
{
  broadcast: {
    strategy: "parallel",
    "120363403215116621@g.us": ["alfred", "baerbel"],
    "+15555550123": ["support", "logger"]
  }
}
```

请参阅：[Broadcast Groups](/zh/broadcast-groups)。

<div id="config-overview">
  ## 配置概览
</div>

* `agents.list`：具名智能体定义（工作区、模型等）。
* `bindings`：将入站通道/账号/对端映射到智能体的映射表。

示例：

```json5
{
  agents: {
    list: [
      { id: "support", name: "Support", workspace: "~/.openclaw/workspace-support" }
    ]
  },
  bindings: [
    { match: { channel: "slack", teamId: "T123" }, agentId: "support" },
    { match: { channel: "telegram", peer: { kind: "group", id: "-100123" } }, agentId: "support" }
  ]
}
```

<div id="session-storage">
  ## 会话存储
</div>

会话存储位于 state 目录下（默认 `~/.openclaw`）：

* `~/.openclaw/agents/<agentId>/sessions/sessions.json`
* JSONL 会话记录文件与存储文件放在同一目录下

你可以通过 `session.store` 并在路径中使用 `{agentId}` 模板占位符来覆盖默认存储路径。

<div id="webchat-behavior">
  ## WebChat 行为
</div>

WebChat 会绑定到**所选 Agent 代理**，并默认使用该 Agent 的主会话。正因如此，WebChat 允许你在同一处查看该 Agent 跨渠道的上下文。

<div id="reply-context">
  ## 回复上下文
</div>

入站回复包括：

* 在可用时包含 `ReplyToId`、`ReplyToBody` 和 `ReplyToSender`。
* 引用上下文会作为一个 `[Replying to ...]` 块附加到 `Body` 中。

这一行为在各个通道中是一致的。