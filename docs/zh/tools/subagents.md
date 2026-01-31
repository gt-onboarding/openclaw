---
title: 子智能体
summary: "子智能体：启动隔离的智能体运行，并将结果回传给发起请求的聊天"
read_when:
  - 你希望通过智能体在后台/并行执行任务
  - 你正在修改 sessions_spawn 或子智能体的工具策略
---

<div id="sub-agents">
  # 子智能体
</div>

子智能体是在现有智能体运行过程中派生出的后台运行智能体实例。它们在各自的会话（`agent:<agentId>:subagent:<uuid>`）中运行，并在完成后，将其结果**发布**回请求方的聊天通道。

<div id="slash-command">
  ## 斜杠命令
</div>

使用 `/subagents` 来检查或控制**当前会话**中的子智能体运行：

- `/subagents list`
- `/subagents stop <id|#|all>`
- `/subagents log <id|#> [limit] [tools]`
- `/subagents info <id|#>`
- `/subagents send <id|#> <message>`

`/subagents info` 会显示运行元数据（状态、时间戳、会话 id、对话记录路径、清理信息）。

主要目标：

- 将“调研 / 长耗时任务 / 慢工具”并行化，而不阻塞主运行。
- 默认保持子智能体隔离（会话分离 + 可选沙箱隔离）。
- 让工具暴露面尽量不易被误用：子智能体默认**不会**获得会话工具。
- 避免嵌套扇出：子智能体不能再生成子智能体。

费用提示：每个子智能体都有**自己的**上下文和 token 消耗。对于重型或重复
任务，可以为子智能体设置更便宜的模型，同时让你的主 Agent 代理使用更高质量的模型。
你可以通过 `agents.defaults.subagents.model` 或按智能体进行覆盖配置。

<div id="tool">
  ## 工具
</div>

使用 `sessions_spawn`：

- 启动一次子智能体运行（`deliver: false`，全局 lane：`subagent`）
- 然后执行一个 announce 步骤，并将 announce 的回复发送到请求方的聊天渠道
- 默认模型：继承调用方，除非你设置了 `agents.defaults.subagents.model`（或为单个智能体设置 `agents.list[].subagents.model`）；显式指定的 `sessions_spawn.model` 仍然具有最高优先级。

工具参数：

- `task`（必填）
- `label?`（可选）
- `agentId?`（可选；如果允许，则在另一个智能体 id 下启动）
- `model?`（可选；覆盖子智能体的模型；无效值会被忽略，子智能体将使用默认模型运行，并在工具结果中给出警告）
- `thinking?`（可选；覆盖该子智能体运行的思考级别）
- `runTimeoutSeconds?`（默认 `0`；设置后，子智能体运行会在 N 秒后被中止）
- `cleanup?`（`delete|keep`，默认 `keep`）

允许列表：

- `agents.list[].subagents.allowAgents`：可以通过 `agentId` 作为目标的智能体 id 列表（`["*"]` 表示允许任意）。默认：仅请求方智能体。

发现：

- 使用 `agents_list` 查看当前哪些智能体 id 被允许用于 `sessions_spawn`。

自动归档：

- 子智能体会话会在 `agents.defaults.subagents.archiveAfterMinutes` 指定的时间后自动归档（默认：60）。
- 归档通过 `sessions.delete` 实现，并将会话记录重命名为 `*.deleted.<timestamp>`（同一文件夹）。
- `cleanup: "delete"` 会在 announce 之后立即归档（仍然通过重命名保留会话记录）。
- 自动归档是尽力而为的机制；如果 Gateway 重启，尚未触发的定时器会丢失。
- `runTimeoutSeconds` **不会** 触发自动归档；它只会停止运行。会话会一直保留到自动归档为止。

<div id="authentication">
  ## 认证
</div>

子 Agent 代理的认证是由 **agent id** 决定的，而不是由会话类型决定：

- 子 Agent 会话键为 `agent:<agentId>:subagent:<uuid>`。
- 认证存储从该智能体的 `agentDir` 加载。
- 主智能体的认证配置文件会作为**后备**被合并进来；在发生冲突时，智能体自己的配置文件会覆盖主配置文件。

注意：合并是叠加式的，因此主配置文件始终可以作为后备使用。目前尚不支持为每个智能体提供完全隔离的认证。

<div id="announce">
  ## Announce
</div>

子智能体通过一个 announce 步骤回传结果：

- announce 步骤在子智能体会话内运行（而不是在请求方的会话中）。
- 如果子智能体的回复恰好是 `ANNOUNCE_SKIP`，则不会发布任何内容。
- 否则，announce 回复会通过后续的 `agent` 调用（`deliver=true`）发布到请求方的聊天通道。
- Announce 回复在可用时会保留线程/话题路由（Slack 线程、Telegram 话题、Matrix 线程）。
- Announce 消息会被规范化为一个稳定的模板：
  - `Status:` 从运行结果推导而来（`success`、`error`、`timeout` 或 `unknown`）。
  - `Result:` 来自 announce 步骤的摘要内容（如果缺失则为 `(not available)`）。
  - `Notes:` 错误详情及其他有用的上下文信息。
- `Status` 不会从模型输出中推断；它来自运行时结果信号。

Announce 负载在末尾会包含一行统计信息（即使被包裹在其他消息中也是如此）：

- 运行时间（例如 `runtime 5m12s`）
- Token 使用量（输入/输出/总计）
- 在配置了模型定价时的预估成本（`models.providers.*.models[].cost`）
- `sessionKey`、`sessionId` 和对话记录路径（这样主智能体可以通过 `sessions_history` 获取历史，或直接在磁盘上检查该文件）

<div id="tool-policy-sub-agent-tools">
  ## 工具策略（子智能体工具）
</div>

默认情况下，子智能体会获得**除会话工具之外的所有工具**：

* `sessions_list`
* `sessions_history`
* `sessions_send`
* `sessions_spawn`

可以通过配置进行重写：

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxConcurrent: 1
      }
    }
  },
  tools: {
    subagents: {
      tools: {
        // deny 优先
        deny: ["gateway", "cron"],
        // 如果设置了 allow,则变为仅允许模式(deny 仍优先)
        // allow: ["read", "exec", "process"]
      }
    }
  }
}
```


<div id="concurrency">
  ## 并发
</div>

子 Agent 代理使用一个专用的进程内队列通道：

- 通道名称：`subagent`
- 并发数：`agents.defaults.subagents.maxConcurrent`（默认值为 `8`）

<div id="stopping">
  ## 停止
</div>

- 在请求方的聊天中发送 `/stop` 会中止该请求方会话，并停止由该会话衍生出的所有正在运行的子智能体。

<div id="limitations">
  ## 限制
</div>

- 子 Agent 代理的 announce 是**尽力而为**机制。如果 Gateway 重启，所有挂起的“announce back”工作都会丢失。
- 子 Agent 代理仍然共享同一个 Gateway 进程资源；将 `maxConcurrent` 视为安全阀。
- `sessions_spawn` 始终为非阻塞：它会立即返回 `{ status: "accepted", runId, childSessionKey }`。
- 子 Agent 代理上下文仅会注入 `AGENTS.md` + `TOOLS.md`（不会注入 `SOUL.md`、`IDENTITY.md`、`USER.md`、`HEARTBEAT.md` 或 `BOOTSTRAP.md`）。