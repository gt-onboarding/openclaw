---
title: 斜杠命令
summary: "斜杠命令：文本模式 vs 原生模式、配置及支持的命令"
read_when:
  - 使用或配置聊天命令时
  - 调试命令路由或权限时
---

<div id="slash-commands">
  # 斜杠命令
</div>

命令由 Gateway 处理。大多数命令必须作为以 `/` 开头的**独立**消息发送。
仅主机可用的 bash 聊天命令使用 `! <cmd>`（`/bash <cmd>` 为其别名）。

有两个相关的系统：

* **Commands（命令）**：独立的 `/...` 消息。
* **Directives（指令）**：`/think`、`/verbose`、`/reasoning`、`/elevated`、`/exec`、`/model`、`/queue`。
  * 在模型看到消息之前，会将指令从消息中移除。
  * 在普通聊天消息中（非仅包含指令），它们被视为“内联提示”，并且**不会**持久化会话设置。
  * 在仅包含指令的消息中（消息只包含指令），它们会持久化到会话中，并返回一条确认回复。
  * 指令仅对**已授权发送方**生效（频道允许列表/配对加上 `commands.useAccessGroups`）。
    未授权发送方会看到指令被当作普通文本处理。

另外还有一些**内联快捷方式**（仅限于允许列表中的/已授权发送方）：`/help`、`/commands`、`/status`、`/whoami`（`/id`）。
它们会立即运行，在模型看到消息前被移除，剩余文本则按正常流程继续处理。

<div id="config">
  ## 配置
</div>

```json5
{
  commands: {
    native: "auto",
    nativeSkills: "auto",
    text: true,
    bash: false,
    bashForegroundMs: 2000,
    config: false,
    debug: false,
    restart: false,
    useAccessGroups: true
  }
}
```

* `commands.text`（默认值为 `true`）启用在聊天消息中解析 `/...`。
  * 在没有原生命令支持的平台（WhatsApp/WebChat/Signal/iMessage/Google Chat/MS Teams）上，即使你将其设为 `false`，文本命令仍然可用。
* `commands.native`（默认值为 `"auto"`）注册原生命令。
  * Auto：在 Discord/Telegram 上启用；在 Slack 上禁用（直到你添加 slash commands）；对不支持原生命令的提供方将被忽略。
  * 设置 `channels.discord.commands.native`、`channels.telegram.commands.native` 或 `channels.slack.commands.native` 可按提供方分别覆盖该设置（布尔值或 `"auto"`）。
  * 将其设为 `false` 会在启动时清除 Discord/Telegram 上之前注册的命令。Slack 命令在 Slack app 中管理，不会被自动移除。
* `commands.nativeSkills`（默认值为 `"auto"`）在支持的情况下以原生方式注册**技能**命令。
  * Auto：在 Discord/Telegram 上启用；在 Slack 上禁用（Slack 需要为每个技能单独创建一个 slash command）。
  * 设置 `channels.discord.commands.nativeSkills`、`channels.telegram.commands.nativeSkills` 或 `channels.slack.commands.nativeSkills` 可按提供方分别覆盖该设置（布尔值或 `"auto"`）。
* `commands.bash`（默认值为 `false`）启用通过 `! <cmd>` 运行宿主 shell 命令（`/bash <cmd>` 是别名；需要在 `tools.elevated` 允许列表中放行）。
* `commands.bashForegroundMs`（默认值为 `2000`）控制 bash 在切换到后台模式前等待的时间（`0` 表示立即转入后台）。
* `commands.config`（默认值为 `false`）启用 `/config`（读写 `openclaw.json`）。
* `commands.debug`（默认值为 `false`）启用 `/debug`（仅在运行时生效的临时覆盖配置）。
* `commands.useAccessGroups`（默认值为 `true`）对命令强制应用允许列表/策略。

<div id="command-list">
  ## 命令列表
</div>

文本 + 原生命令（启用时）：

* `/help`
* `/commands`
* `/skill <name> [input]`（按名称运行 skill）
* `/status`（显示当前状态；在可用时包含当前模型提供方的使用量/配额）
* `/allowlist`（列出/添加/移除允许列表条目）
* `/approve <id> allow-once|allow-always|deny`（处理执行审批提示）
* `/context [list|detail|json]`（解释“context”；`detail` 显示按文件 + 按工具 + 按 skill + system prompt 的大小）
* `/whoami`（显示你的发送方 ID；别名：`/id`）
* `/subagents list|stop|log|info|send`（检查、停止、记录或向当前会话的子智能体运行实例发送消息）
* `/config show|get|set|unset`（将配置持久化到磁盘，仅所有者；需要 `commands.config: true`）
* `/debug show|set|unset|reset`（运行时覆写配置，仅所有者；需要 `commands.debug: true`）
* `/usage off|tokens|full|cost`（在每次响应后显示使用情况页脚或本地成本汇总）
* `/tts off|always|inbound|tagged|status|provider|limit|summary|audio`（控制 TTS；参见 [/tts](/zh/tts)）
  * Discord：原生命令为 `/voice`（Discord 保留了 `/tts`）；文本命令 `/tts` 仍然有效。
* `/stop`
* `/restart`
* `/dock-telegram`（别名：`/dock_telegram`）（将回复切换到 Telegram）
* `/dock-discord`（别名：`/dock_discord`）（将回复切换到 Discord）
* `/dock-slack`（别名：`/dock_slack`）（将回复切换到 Slack）
* `/activation mention|always`（仅群组）
* `/send on|off|inherit`（仅所有者）
* `/reset` 或 `/new [model]`（可选模型提示；其余内容将透传）
* `/think <off|minimal|low|medium|high|xhigh>`（由模型/提供方动态决定的选项；别名：`/thinking`、`/t`）
* `/verbose on|full|off`（别名：`/v`）
* `/reasoning on|off|stream`（别名：`/reason`；开启时会发送一条以 `Reasoning:` 为前缀的单独消息；`stream` = 仅 Telegram 草稿）
* `/elevated on|off|ask|full`（别名：`/elev`；`full` 会跳过执行审批）
* `/exec host=<sandbox|gateway|node> security=<deny|allowlist|full> ask=<off|on-miss|always> node=<id>`（发送 `/exec` 查看当前设置）
* `/model <name>`（别名：`/models`；或使用来自 `agents.defaults.models.*.alias` 的 `/<alias>`）
* `/queue <mode>`（加上类似 `debounce:2s cap:25 drop:summarize` 的选项；发送 `/queue` 查看当前设置）
* `/bash <command>`（仅主机（host-only）；`! <command>` 的别名；需要 `commands.bash: true` + `tools.elevated` 允许列表）

仅文本：

* `/compact [instructions]`（参见 [/concepts/compaction](/zh/concepts/compaction)）
* `! <command>`（仅主机；一次一个；对长时间运行的任务使用 `!poll` + `!stop`）
* `!poll`（检查输出/状态；接受可选 `sessionId`；`/bash poll` 也可用）
* `!stop`（停止正在运行的 bash 任务；接受可选 `sessionId`；`/bash stop` 也可用）

注意：

* 命令在命令名和参数之间可以选择性地使用一个 `:`（例如 `/think: high`、`/send: on`、`/help:`）。
* `/new <model>` 接受模型别名、`provider/model`，或提供方名称（模糊匹配）；如果未匹配到，则文本会被视为消息正文。
* 若要查看完整的提供方用量拆解，请使用 `openclaw status --usage`。
* `/allowlist add|remove` 需要 `commands.config=true`，并遵循频道的 `configWrites` 设置。
* `/usage` 控制每条回复末尾的用量附注；`/usage cost` 会基于 OpenClaw 会话日志打印本地成本汇总。
* `/restart` 默认禁用；将 `commands.restart: true` 设为 true 以启用。
* `/verbose` 主要用于调试和增加可见性；在正常使用中请保持其为**关闭**。
* `/reasoning`（以及 `/verbose`）在群组环境中存在风险：它们可能会泄露你并不打算公开的内部推理或工具输出。建议保持关闭，尤其是在群聊中。
* **快速路径：** 来自允许列表发送方且仅包含命令的消息会被立即处理（绕过队列和模型）。
* **群组提及门控：** 来自允许列表发送方且仅包含命令的消息会绕过“需要被提及”的限制。
* **内联快捷方式（仅限允许列表发送方）：** 某些命令在嵌入到普通消息中时同样生效，并会在模型看到剩余文本之前被剥离。
  * 示例：`hey /status` 会触发状态回复，其余文本按正常流程继续处理。
* 当前支持：`/help`、`/commands`、`/status`、`/whoami`（`/id`）。
* 未授权的“仅命令”消息会被静默忽略，内联的 `/...` 标记会被视为普通文本。
* **技能命令：** `user-invocable` 技能会以斜杠命令的形式暴露。名称会被规范化为 `a-z0-9_`（最多 32 个字符）；发生冲突时会添加数字后缀（例如 `_2`）。
  * `/skill <name> [input]` 按名称运行某个技能（当原生命令数量限制导致无法为每个技能单独创建命令时非常有用）。
  * 默认情况下，技能命令会作为普通请求转发给模型。
  * 技能可以选择声明 `command-dispatch: tool`，将命令直接路由到某个工具（确定性执行，无需模型）。
  * 示例：`/prose`（OpenProse 插件）——参见 [OpenProse](/zh/prose)。
* **原生命令参数：** Discord 使用自动补全来提供动态选项（当你省略必填参数时则使用按钮菜单）。Telegram 和 Slack 在命令支持选项且你省略参数时会显示按钮菜单。

<div id="usage-surfaces-what-shows-where">
  ## 使用呈现位置（在哪显示什么）
</div>

* **提供方用量/配额**（示例：“Claude 剩余 80%”）在启用用量跟踪后，会显示在当前模型提供方的 `/status` 中。
* **每次回复的 tokens/费用** 由 `/usage off|tokens|full` 命令控制（作为附加信息跟在正常回复后面）。
* `/model status` 是关于 **模型/认证/端点** 的，而不是用量。

<div id="model-selection-model">
  ## 模型选择（`/model`）
</div>

`/model` 作为一个指令实现。

示例：

```
/model
/model list
/model 3
/model openai/gpt-5.2
/model opus@anthropic:default
/model status
```

Notes:

* `/model` 和 `/model list` 会显示一个紧凑的编号式选择器（模型系列 + 可用提供方）。
* `/model &lt;#&gt;` 会从该选择器中选定对应项（并在可能时优先使用当前提供方）。
* `/model status` 会显示详细视图，包括已配置的提供方端点（`baseUrl`）和 API 模式（`api`，如果可用）。

<div id="debug-overrides">
  ## 调试覆盖配置
</div>

`/debug` 允许你设置**仅在运行时生效**的配置覆盖（只存于内存，不写入磁盘）。仅所有者可用。默认禁用；通过 `commands.debug: true` 启用。

示例：

```
/debug show
/debug set messages.responsePrefix="[openclaw]"
/debug set channels.whatsapp.allowFrom=["+1555","+4477"]
/debug unset messages.responsePrefix
/debug reset
```

注意：

* 覆盖设置会在新的配置读取中立即生效，但**不会**写入 `openclaw.json`。
* 使用 `/debug reset` 清除所有覆盖设置，并恢复为磁盘上的配置。

<div id="config-updates">
  ## 配置更新
</div>

`/config` 会将配置写入磁盘上的配置文件（`openclaw.json`）。仅限所有者使用。默认关闭；可通过 `commands.config: true` 启用。

示例：

```
/config show
/config show messages.responsePrefix
/config get messages.responsePrefix
/config set messages.responsePrefix="[openclaw]"
/config unset messages.responsePrefix
```

备注：

* 在写入前会先对配置进行校验；无效变更会被拒绝。
* `/config` 的更新在重启后仍会保留。

<div id="surface-notes">
  ## 概览说明
</div>

* **文本命令（Text commands）** 在普通聊天会话中运行（私信 DM 共享 `main`，群组各自拥有独立会话）。
* **原生命令（Native commands）** 使用独立会话：
  * Discord：`agent:<agentId>:discord:slash:<userId>`
  * Slack：`agent:<agentId>:slack:slash:<userId>`（前缀可通过 `channels.slack.slashCommand.sessionPrefix` 配置）
  * Telegram：`telegram:slash:<userId>`（通过 `CommandTargetSessionKey` 定位到对应聊天会话）
* **`/stop`** 作用于当前活动聊天会话，用于中止当前运行。
* **Slack：** 仍然支持基于 `channels.slack.slashCommand` 的单一 `/openclaw` 风格命令。如果你启用了 `commands.native`，必须为每个内置命令分别创建一个 Slack 斜杠命令（名称与 `/help` 中相同）。Slack 的命令参数菜单通过短暂可见（ephemeral）的 Block Kit 按钮提供。