---
title: 会话工具
summary: "Agent 代理的会话工具，用于列出会话、获取历史记录以及跨会话发送消息"
read_when:
  - 在添加或修改会话工具时
---

<div id="session-tools">
  # 会话工具
</div>

目标：提供一组小巧且不易被误用的工具，使智能体可以列出会话、获取会话历史，并将内容发送到另一个会话。

<div id="tool-names">
  ## 工具名称
</div>

- `sessions_list`
- `sessions_history`
- `sessions_send`
- `sessions_spawn`

<div id="key-model">
  ## 键模型
</div>

- 主直接聊天存储桶始终使用字面量键 `"main"`（解析为当前智能体的主键）。
- 群聊使用 `agent:<agentId>:<channel>:group:<id>` 或 `agent:<agentId>:<channel>:channel:<id>`（传入完整键名）。
- Cron 作业使用 `cron:<job.id>`。
- Hooks 使用 `hook:<uuid>`，除非显式设置。
- 节点会话使用 `node-<nodeId>`，除非显式设置。

`global` 和 `unknown` 是保留值，永远不会被列出。如果 `session.scope = "global"`，我们会在所有工具中将它别名为 `main`，这样调用方永远不会看到 `global`。

<div id="sessions_list">
  ## sessions_list
</div>

以行数组形式列出会话。

参数：

- `kinds?: string[]` 过滤器：`"main" | "group" | "cron" | "hook" | "node" | "other"` 中的任意值
- `limit?: number` 最大行数（默认：使用服务端默认值，例如限制为 200）
- `activeMinutes?: number` 只包含在最近 N 分钟内有更新的会话
- `messageLimit?: number` 0 = 不包含消息（默认 0）；>0 = 包含最近的 N 条消息

行为：

- 当 `messageLimit > 0` 时，会为每个会话获取 `chat.history` 并包含最近 N 条消息。
- 工具结果会从列表输出中被过滤掉；若需要工具消息，请使用 `sessions_history`。
- 在**沙箱化的**智能体会话中运行时，会话工具默认采用**仅对衍生会话可见（spawned-only）**的可见性策略（见下文）。

行结构（JSON）：

- `key`: 会话键（字符串）
- `kind`: `main | group | cron | hook | node | other`
- `channel`: `whatsapp | telegram | discord | signal | imessage | webchat | internal | unknown`
- `displayName`（如果可用，为群组显示标签）
- `updatedAt`（毫秒）
- `sessionId`
- `model`, `contextTokens`, `totalTokens`
- `thinkingLevel`, `verboseLevel`, `systemSent`, `abortedLastRun`
- `sendPolicy`（如果设置，则为会话级覆写策略）
- `lastChannel`, `lastTo`
- `deliveryContext`（在可用时为归一化的 `{ channel, to, accountId }`）
- `transcriptPath`（基于存储目录 + sessionId 推导出的尽力路径）
- `messages?`（仅当 `messageLimit > 0` 时存在）

<div id="sessions_history">
  ## sessions_history
</div>

获取单个会话的对话记录。

参数：

- `sessionKey`（必填；接受会话 key 或来自 `sessions_list` 的 `sessionId`）
- `limit?: number` 消息条数上限（由服务器强制限制）
- `includeTools?: boolean`（默认 false）

行为：

- 当 `includeTools=false` 时，会过滤掉 `role: "toolResult"` 的消息。
- 以原始对话记录格式返回消息数组。
- 当提供 `sessionId` 时，OpenClaw 会将其解析为对应的会话 key（如果 id 不存在则返回错误）。

<div id="sessions_send">
  ## sessions_send
</div>

向另一个会话发送一条消息。

参数：

- `sessionKey`（必填；接受会话 key 或来自 `sessions_list` 的 `sessionId`）
- `message`（必填）
- `timeoutSeconds?: number`（默认值 >0；0 = fire-and-forget）

行为：

- `timeoutSeconds = 0`：入队并返回 `{ runId, status: "accepted" }`。
- `timeoutSeconds > 0`：最多等待 N 秒完成，然后返回 `{ runId, status: "ok", reply }`。
- 如果等待超时：`{ runId, status: "timeout", error }`。运行会继续；稍后调用 `sessions_history`。
- 如果运行失败：`{ runId, status: "error", error }`。
- 通知投递运行在主运行完成后触发，且为尽力而为；`status: "ok"` 并不保证通知一定成功投递。
- 通过 Gateway 的 `agent.wait`（服务端）进行等待，因此重连不会中断等待。
- 主运行会注入智能体到智能体的消息上下文。
- 主运行完成后，OpenClaw 会运行一个**回复回传循环（reply-back loop）**：
  - 第 2 轮及之后在请求方和目标智能体之间交替进行。
  - 精确回复 `REPLY_SKIP` 以停止乒乓往返。
  - 最大轮数为 `session.agentToAgent.maxPingPongTurns`（0–5，默认 5）。
- 一旦循环结束，OpenClaw 会运行**智能体到智能体通知步骤（announce step）**（仅在目标智能体上执行）：
  - 精确回复 `ANNOUNCE_SKIP` 以保持静默。
  - 任何其他回复都会被发送到目标通道。
  - 通知步骤中会包含原始请求 + 第 1 轮回复 + 最新的乒乓回复。

<div id="channel-field">
  ## 频道字段
</div>

- 对于群组，`channel` 是记录在会话条目上的频道。
- 对于私聊，`channel` 由 `lastChannel` 映射而来。
- 对于 cron/hook/node，`channel` 为 `internal`。
- 如果缺失，`channel` 为 `unknown`。

<div id="security-send-policy">
  ## 安全 / 发送策略
</div>

基于策略，对不同渠道/聊天类型进行阻断（而非按会话 ID）。

```json
{
  "session": {
    "sendPolicy": {
      "rules": [
        {
          "match": { "channel": "discord", "chatType": "group" },
          "action": "deny"
        }
      ],
      "default": "allow"
    }
  }
}
```

运行时覆盖（按会话项）：

* `sendPolicy: "allow" | "deny"`（未设置 = 继承配置）
* 可通过 `sessions.patch` 或所有者专用的 `/send on|off|inherit`（独立消息）进行设置。

执行点：

* `chat.send` / 智能体（Gateway）
* 自动回复发送逻辑


<div id="sessions_spawn">
  ## sessions_spawn
</div>

在隔离会话中启动一次子智能体运行，并将结果回传到请求方的聊天通道。

参数：

- `task`（必填）
- `label?`（可选；用于日志/UI）
- `agentId?`（可选；如果允许，则以另一个智能体 id 启动）
- `model?`（可选；覆盖子智能体使用的模型；无效值会报错）
- `runTimeoutSeconds?`（默认 0；设置后，会在运行 N 秒后中止子智能体）
- `cleanup?`（`delete|keep`，默认 `keep`）

允许列表：

- `agents.list[].subagents.allowAgents`：通过 `agentId` 允许的智能体 id 列表（`["*"]` 表示允许任意）。默认：仅请求方智能体。

发现：

- 使用 `agents_list` 来发现哪些智能体 id 被允许用于 `sessions_spawn`。

行为：

- 以 `deliver: false` 启动一个新的 `agent:<agentId>:subagent:<uuid>` 会话。
- 子智能体默认拥有完整工具集，**但不包括会话工具**（可通过 `tools.subagents.tools` 配置）。
- 子智能体不允许调用 `sessions_spawn`（禁止子智能体 → 子智能体的嵌套启动）。
- 始终为非阻塞调用：会立即返回 `{ status: "accepted", runId, childSessionKey }`。
- 完成后，OpenClaw 会执行一次子智能体的 **announce 步骤**，并将结果发送到请求方聊天通道。
- 在 announce 步骤中精确回复 `ANNOUNCE_SKIP` 可保持静默。
- announce 回复会被规范化为 `Status`/`Result`/`Notes`；`Status` 源自运行时结果（而非模型文本）。
- 子智能体会话会在 `agents.defaults.subagents.archiveAfterMinutes` 指定时间后自动归档（默认：60）。
- announce 回复中会包含一行统计信息（运行时长、tokens、sessionKey/sessionId、对话记录路径，以及可选成本）。

<div id="sandbox-session-visibility">
  ## 沙箱会话可见性
</div>

处于沙箱中的会话可以使用会话工具，但默认情况下，它们只能看到自己通过 `sessions_spawn` 创建的会话。

配置：

```json5
{
  agents: {
    defaults: {
      sandbox: {
        // 默认: "spawned"
        sessionToolsVisibility: "spawned" // or "all"
      }
    }
  }
}
```
