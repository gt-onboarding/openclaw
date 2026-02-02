---
title: 工具
summary: "OpenClaw 的 Agent 代理工具能力范围（browser、canvas、nodes、message、cron），用于替代旧版 `openclaw-*` 技能"
read_when:
  - 添加或修改智能体工具时
  - 废弃或更改 `openclaw-*` 技能时
---

<div id="tools-openclaw">
  # 工具（OpenClaw）
</div>

OpenClaw 为浏览器、画布、节点和 cron 提供了**一等公民级的智能体工具**。
这些工具取代了旧的 `openclaw-*` 技能：工具具备明确的类型定义、不会通过 shell 调用外部命令，
并且智能体应当直接依赖并使用这些工具。

<div id="disabling-tools">
  ## 禁用工具
</div>

你可以在 `openclaw.json` 中通过设置 `tools.allow` / `tools.deny` 来全局允许或禁止工具（`deny` 优先生效）。这样可以防止被禁用的工具被发送给模型提供方。

```json5
{
  tools: { deny: ["browser"] }
}
```

注意：

* 匹配时不区分大小写。
* 支持 `*` 通配符（`"*"` 表示所有工具）。
* 如果 `tools.allow` 只引用未知或未加载的插件工具名称，OpenClaw 会输出一条警告并忽略该允许列表，以确保核心工具仍然可用。

<div id="tool-profiles-base-allowlist">
  ## 工具配置文件（基础允许列表）
</div>

`tools.profile` 会在 `tools.allow`/`tools.deny` 之前设置一个**基础工具允许列表**。
智能体级覆盖：`agents.list[].tools.profile`。

配置文件：

* `minimal`：仅 `session_status`
* `coding`：`group:fs`、`group:runtime`、`group:sessions`、`group:memory`、`image`
* `messaging`：`group:messaging`、`sessions_list`、`sessions_history`、`sessions_send`、`session_status`
* `full`：无限制（与未设置相同）

示例（默认仅限消息工具，同时也允许 Slack 和 Discord 工具）：

```json5
{
  tools: {
    profile: "messaging",
    allow: ["slack", "discord"]
  }
}
```

示例（面向编码的配置模板，但在所有场景一律禁止 exec/process）：

```json5
{
  tools: {
    profile: "coding",
    deny: ["group:runtime"]
  }
}
```

示例（全局编码配置文件、仅支持消息的客服智能体）：

```json5
{
  tools: { profile: "coding" },
  agents: {
    list: [
      {
        id: "support",
        tools: { profile: "messaging", allow: ["slack"] }
      }
    ]
  }
}
```

<div id="provider-specific-tool-policy">
  ## 特定提供方的工具策略
</div>

使用 `tools.byProvider` 为特定提供方
（或单个 `provider/model`）**进一步收紧** 可用工具，而不更改全局默认配置。
按智能体覆盖：`agents.list[].tools.byProvider`。

该配置在基础工具配置**之后**、allow/deny 列表**之前**生效，
因此它只能缩小可用工具集合。
提供方键既可以是 `provider`（例如 `google-antigravity`），也可以是
`provider/model`（例如 `openai/gpt-5.2`）。

示例（保持全局编码配置，但对 Google Antigravity 仅启用最少工具）：

```json5
{
  tools: {
    profile: "coding",
    byProvider: {
      "google-antigravity": { profile: "minimal" }
    }
  }
}
```

示例（针对不稳定端点的提供方/模型专用允许列表）：

```json5
{
  tools: {
    allow: ["group:fs", "group:runtime", "sessions_list"],
    byProvider: {
      "openai/gpt-5.2": { allow: ["group:fs", "sessions_list"] }
    }
  }
}
```

示例（针对单个提供方的智能体级别覆盖配置）：

```json5
{
  agents: {
    list: [
      {
        id: "support",
        tools: {
          byProvider: {
            "google-antigravity": { allow: ["message", "sessions_list"] }
          }
        }
      }
    ]
  }
}
```

<div id="tool-groups-shorthands">
  ## 工具分组（简写）
</div>

工具策略（全局、智能体、沙箱）支持 `group:*` 条目，这些条目会展开为多个工具。
可以在 `tools.allow` / `tools.deny` 中使用它们。

可用分组：

* `group:runtime`: `exec`, `bash`, `process`
* `group:fs`: `read`, `write`, `edit`, `apply_patch`
* `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
* `group:memory`: `memory_search`, `memory_get`
* `group:web`: `web_search`, `web_fetch`
* `group:ui`: `browser`, `canvas`
* `group:automation`: `cron`, `gateway`
* `group:messaging`: `message`
* `group:nodes`: `nodes`
* `group:openclaw`: 所有内置的 OpenClaw 工具（不包括提供方插件）

示例（仅允许文件工具和浏览器工具）：

```json5
{
  tools: {
    allow: ["group:fs", "browser"]
  }
}
```

<div id="plugins-tools">
  ## 插件 + 工具
</div>

插件可以注册除核心工具集之外的**额外工具**（以及 CLI 命令）。
安装和配置请参见 [Plugins](/zh/plugin)，关于如何将工具使用说明注入到提示词中，请参见 [Skills](/zh/tools/skills)。
某些插件会在工具之外同时附带自己的技能（例如 voice-call 插件）。

可选插件工具：

* [Lobster](/zh/tools/lobster)：具有可恢复审批的类型化工作流运行时（需要在 Gateway 主机上安装 Lobster CLI）。
* [LLM Task](/zh/tools/llm-task)：用于结构化工作流输出的仅 JSON 的 LLM 步骤（可选 schema 校验）。

<div id="tool-inventory">
  ## 工具清单
</div>

<div id="apply_patch">
  ### `apply_patch`
</div>

跨一个或多个文件应用结构化补丁。适用于包含多个 hunk 的编辑。
实验性功能：通过 `tools.exec.applyPatch.enabled` 启用（仅适用于 OpenAI 模型）。

<div id="exec">
  ### `exec`
</div>

在工作区中运行 shell 命令。

核心参数：

* `command`（必填）
* `yieldMs`（在超时后自动转为后台运行，默认 10000）
* `background`（立即后台运行）
* `timeout`（单位为秒；超时则杀死进程，默认 1800）
* `elevated`（布尔值；如果已启用/允许提权模式，则在宿主机上运行；仅当智能体在沙箱中时才会改变行为）
* `host` (`sandbox | gateway | node`)
* `security` (`deny | allowlist | full`)
* `ask` (`off | on-miss | always`)
* `node`（当 `host=node` 时使用的节点 ID/名称）
* 需要真实的 TTY？设置 `pty: true`。

注意事项：

* 在转为后台运行时会返回 `status: "running"`，并附带一个 `sessionId`。
* 使用 `process` 来轮询/记录日志/写入/终止/清理后台会话。
* 如果不允许 `process`，`exec` 将同步运行，并忽略 `yieldMs`/`background`。
* `elevated` 受 `tools.elevated` 和任意 `agents.list[].tools.elevated` 覆盖项共同控制（两者都必须允许），并且等价于 `host=gateway` + `security=full`。
* `elevated` 仅在智能体处于沙箱中时才会改变行为（否则是空操作，无效果）。
* `host=node` 可以指向 macOS 配套应用或无头节点宿主机（`openclaw node run`）。
* Gateway/node 审批与允许列表：[Exec approvals](/zh/tools/exec-approvals)。

<div id="process">
  ### `process`
</div>

管理后台执行会话。

核心操作：

* `list`, `poll`, `log`, `write`, `kill`, `clear`, `remove`

注意：

* `poll` 在会话完成时返回新增输出和退出状态。
* `log` 支持基于行的 `offset`/`limit`（省略 `offset` 以获取最后的 N 行）。
* `process` 按智能体维度生效；来自其他智能体的会话不可见。

<div id="web_search">
  ### `web_search`
</div>

使用 Brave 搜索 API 进行网页搜索。

核心参数：

* `query`（必填）
* `count`（1–10；默认取自 `tools.web.search.maxResults`）

说明：

* 需要 Brave API 密钥（推荐：运行 `openclaw configure --section web`，或设置 `BRAVE_API_KEY`）。
* 通过 `tools.web.search.enabled` 启用。
* 响应会被缓存（默认 15 分钟）。
* 配置说明参见 [Web tools](/zh/tools/web)。

<div id="web_fetch">
  ### `web_fetch`
</div>

从 URL 获取并提取可读内容（HTML → markdown/text）。

核心参数：

* `url`（必填）
* `extractMode`（`markdown` | `text`）
* `maxChars`（截断超长页面）

注意：

* 通过 `tools.web.fetch.enabled` 启用。
* 响应会被缓存（默认 15 分钟）。
* 对于严重依赖 JS 的站点，优先使用 `browser` 工具。
* 参见 [Web tools](/zh/tools/web) 了解配置方法。
* 参见 [Firecrawl](/zh/tools/firecrawl) 了解可选的反爬虫回退方案。

<div id="browser">
  ### `browser`
</div>

控制由 OpenClaw 管理的专用浏览器。

核心操作：

* `status`, `start`, `stop`, `tabs`, `open`, `focus`, `close`
* `snapshot`（aria/ai）
* `screenshot`（返回图像块 + `MEDIA:<path>`）
* `act`（UI 操作：click/type/press/hover/drag/select/fill/resize/wait/evaluate）
* `navigate`, `console`, `pdf`, `upload`, `dialog`

配置文件管理：

* `profiles` — 列出所有浏览器配置文件及其状态
* `create-profile` — 创建具有自动分配端口（或 `cdpUrl`）的新配置文件
* `delete-profile` — 停止浏览器、删除用户数据，并从配置中移除（仅本地）
* `reset-profile` — 终止占用该配置文件端口的孤立进程（仅本地）

常用参数：

* `profile`（可选；默认为 `browser.defaultProfile`）
* `target`（`sandbox` | `host` | `node`）
* `node`（可选；指定特定的节点 id/name）

注意事项：

* 需要 `browser.enabled=true`（默认值为 `true`；设为 `false` 以禁用）。
* 所有操作都接受可选的 `profile` 参数，以支持多实例。
* 当省略 `profile` 时，会使用 `browser.defaultProfile`（默认为 &quot;chrome&quot;）。
* 配置文件名称：仅限小写字母、数字和连字符（最长 64 个字符）。
* 端口范围：18800-18899（最多约 100 个配置文件）。
* 远程配置文件仅支持附加（attach），不支持 start/stop/reset。
* 如果已连接具备浏览器能力的节点，工具可能会自动路由到该节点（除非你固定 `target`）。
* 当安装了 Playwright 时，`snapshot` 默认使用 `ai`；使用 `aria` 可获取可访问性树。
* `snapshot` 还支持角色快照选项（`interactive`、`compact`、`depth`、`selector`），会返回类似 `e12` 的引用。
* `act` 需要来自 `snapshot` 的 `ref`（AI 快照中的数字 `12`，或角色快照中的 `e12`）；对于少数需要 CSS 选择器的场景，请使用 `evaluate`。
* 默认应避免在 `act` 中使用 `wait`；仅在极少数无法依赖 UI 状态进行等待的情况下使用。
* `upload` 可以选择性传入 `ref`，在“武装”完成后自动点击目标元素。
* `upload` 还支持 `inputRef`（aria 引用）或 `element`（CSS 选择器），以直接设置 `<input type="file">`。

<div id="canvas">
  ### `canvas`
</div>

控制节点 Canvas（present、eval、snapshot、A2UI）。

核心操作：

* `present`、`hide`、`navigate`、`eval`
* `snapshot`（返回图像块 + `MEDIA:<path>`）
* `a2ui_push`、`a2ui_reset`

注意事项：

* 底层通过 Gateway 的 `node.invoke` 实现。
* 如果未提供 `node`，该工具会选择一个默认节点（单个已连接节点或本地 macOS 节点）。
* A2UI 目前仅支持 v0.8（不支持 `createSurface`）；CLI 会将 v0.9 JSONL 逐行报错并拒绝。
* 快速冒烟测试：`openclaw nodes canvas a2ui push --node <id> --text "Hello from A2UI"`。

<div id="nodes">
  ### `nodes`
</div>

发现并选取已配对节点；发送通知；捕获相机/屏幕。

核心操作：

* `status`, `describe`
* `pending`, `approve`, `reject`（配对）
* `notify`（macOS `system.notify`）
* `run`（macOS `system.run`）
* `camera_snap`, `camera_clip`, `screen_record`
* `location_get`

注意：

* 相机/屏幕命令要求节点应用在前台运行。
* 图片返回图像块 + `MEDIA:<path>`。
* 视频返回 `FILE:<path>`（mp4）。
* 位置返回一个 JSON 载荷（lat/lon/accuracy/timestamp）。
* `run` 参数：`command` argv 数组；可选 `cwd`、`env`（`KEY=VAL`）、`commandTimeoutMs`、`invokeTimeoutMs`、`needsScreenRecording`。

示例（`run`）：

```json
{
  "action": "run",
  "node": "office-mac",
  "command": ["echo", "Hello"],
  "env": ["FOO=bar"],
  "commandTimeoutMs": 12000,
  "invokeTimeoutMs": 45000,
  "needsScreenRecording": false
}
```

<div id="image">
  ### `image`
</div>

使用已配置的图像模型分析图像。

核心参数：

* `image`（必需，路径或 URL）
* `prompt`（可选；默认为“Describe the image.”）
* `model`（可选，用于覆盖默认模型）
* `maxBytesMb`（可选，大小上限）

注意：

* 仅在已配置 `agents.defaults.imageModel`（主模型或回退模型），或可从你的默认模型 + 已配置的认证信息中推断出隐式图像模型时可用（尽力进行配对）。
* 直接使用图像模型（独立于主聊天模型）。

<div id="message">
  ### `message`
</div>

在 Discord/Google Chat/Slack/Telegram/WhatsApp/Signal/iMessage/MS Teams 上发送消息和频道级操作。

核心操作：

* `send`（文本 + 可选媒体；MS Teams 还支持用于 Adaptive Cards 的 `card`）
* `poll`（WhatsApp/Discord/MS Teams 投票）
* `react` / `reactions` / `read` / `edit` / `delete`
* `pin` / `unpin` / `list-pins`
* `permissions`
* `thread-create` / `thread-list` / `thread-reply`
* `search`
* `sticker`
* `member-info` / `role-info`
* `emoji-list` / `emoji-upload` / `sticker-upload`
* `role-add` / `role-remove`
* `channel-info` / `channel-list`
* `voice-status`
* `event-list` / `event-create`
* `timeout` / `kick` / `ban`

注意：

* `send` 在 WhatsApp 上通过 Gateway 路由；其他频道则直接发送。
* `poll` 对 WhatsApp 和 MS Teams 使用 Gateway；Discord 投票则直接发送。
* 当 `message` 工具调用绑定到一个活动聊天会话时，发送会被限制在该会话的目标内，以避免跨上下文泄漏。

<div id="cron">
  ### `cron`
</div>

管理 Gateway 的 cron 任务和唤醒。

核心操作：

* `status`, `list`
* `add`, `update`, `remove`, `run`, `runs`
* `wake`（将系统事件入队 + 可选的即时心跳）

说明：

* `add` 需要一个完整的 cron 任务对象（与 `cron.add` RPC 使用相同的 schema）。
* `update` 使用 `{ id, patch }`。

<div id="gateway">
  ### `gateway`
</div>

对正在运行的 Gateway 进程执行重启或应用更新（就地操作）。

核心操作：

* `restart`（完成授权并发送 `SIGUSR1` 以在进程内重启；`openclaw gateway` 就地重启）
* `config.get` / `config.schema`
* `config.apply`（校验并写入配置 + 重启 + 唤醒）
* `config.patch`（合并部分更新 + 重启 + 唤醒）
* `update.run`（执行更新 + 重启 + 唤醒）

说明：

* 使用 `delayMs`（默认 2000）以避免中断正在进行中的回复。
* `restart` 默认禁用；可通过 `commands.restart: true` 启用。

<div id="sessions_list-sessions_history-sessions_send-sessions_spawn-session_status">
  ### `sessions_list` / `sessions_history` / `sessions_send` / `sessions_spawn` / `session_status`
</div>

列出会话、检查对话历史，或向另一个会话发送消息。

核心参数：

* `sessions_list`: `kinds?`, `limit?`, `activeMinutes?`, `messageLimit?`（0 = 不限制）
* `sessions_history`: `sessionKey`（或 `sessionId`）、`limit?`、`includeTools?`
* `sessions_send`: `sessionKey`（或 `sessionId`）、`message`、`timeoutSeconds?`（0 = fire-and-forget，发送后不等待结果）
* `sessions_spawn`: `task`, `label?`, `agentId?`, `model?`, `runTimeoutSeconds?`, `cleanup?`
* `session_status`: `sessionKey?`（默认为当前会话；也接受 `sessionId`）、`model?`（`default` 会清除模型覆盖）

说明：

* `main` 是规范的直接聊天会话键名；global/unknown 会被隐藏。
* 当 `messageLimit > 0` 时，为每个会话获取最近的 N 条消息（会过滤工具消息）。
* `sessions_send` 在 `timeoutSeconds > 0` 时会等待最终完成。
* 消息投递 / announce 会在完成后进行，且是尽力而为；`status: "ok"` 仅表示智能体运行已结束，并不保证 announce 已成功投递。
* `sessions_spawn` 会启动一次子智能体运行，并向请求方的聊天中发送一个 announce 回复。
* `sessions_spawn` 为非阻塞调用，会立即返回 `status: "accepted"`。
* `sessions_send` 会执行一个往返式的回复 ping‑pong（回复 `REPLY_SKIP` 以停止；最大轮数由 `session.agentToAgent.maxPingPongTurns` 控制，范围 0–5）。
* 在 ping‑pong 结束后，目标智能体会执行一次 **announce 步骤**；回复 `ANNOUNCE_SKIP` 可抑制该 announce。

<div id="agents_list">
  ### `agents_list`
</div>

列出当前会话可以通过 `sessions_spawn` 作为目标的智能体 ID 列表。

注意：

* 结果会受每个智能体允许列表（`agents.list[].subagents.allowAgents`）的限制。
* 当配置为 `["*"]` 时，该工具会包含所有已配置的智能体，并标记 `allowAny: true`。

<div id="parameters-common">
  ## 参数（通用）
</div>

由 Gateway 提供支持的工具（`canvas`、`nodes`、`cron`）：

* `gatewayUrl`（默认值为 `ws://127.0.0.1:18789`）
* `gatewayToken`（如果启用了认证）
* `timeoutMs`

浏览器工具：

* `profile`（可选；默认为 `browser.defaultProfile`）
* `target`（`sandbox` | `host` | `node`）
* `node`（可选；绑定到某个特定的节点 ID/名称）

<div id="recommended-agent-flows">
  ## 推荐的智能体工作流
</div>

浏览器自动化：

1. `browser` → `status` / `start`
2. `snapshot`（ai 或 aria）
3. `act`（click/type/press）
4. 如需进行画面确认，执行 `screenshot`

画布渲染：

1. `canvas` → `present`
2. `a2ui_push`（可选）
3. `snapshot`

节点定向：

1. `nodes` → `status`
2. 在选定节点上执行 `describe`
3. `notify` / `run` / `camera_snap` / `screen_record`

<div id="safety">
  ## 安全性
</div>

* 避免直接使用 `system.run`；仅在获得用户明确同意后，改用 `nodes` → `run` 执行。
* 尊重用户对摄像头/屏幕捕获的授权。
* 在调用媒体命令前，使用 `status/describe` 确认权限。

<div id="how-tools-are-presented-to-the-agent">
  ## 工具是如何呈现给智能体的
</div>

工具通过两个并行通道暴露给智能体：

1. **系统提示文本**：人类可读的工具列表和使用指引。
2. **工具 schema**：发送给模型 API 的结构化函数定义。

这意味着智能体既能看到“有哪些工具”，也能知道“如何调用它们”。如果某个工具
没有出现在系统提示或 schema 中，模型就无法调用它。