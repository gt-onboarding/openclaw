---
title: CLI
summary: "OpenClaw CLI 中 `openclaw` 命令、子命令和选项的参考文档"
read_when:
  - 添加或修改 CLI 命令或选项时
  - 为新的命令接口编写文档时
---

<div id="cli-reference">
  # CLI 参考
</div>

本页面描述当前 CLI 的行为。如果命令有变更，请更新本文档。

<div id="command-pages">
  ## 命令页面
</div>

* [`setup`](/zh/cli/setup)
* [`onboard`](/zh/cli/onboard)
* [`configure`](/zh/cli/configure)
* [`config`](/zh/cli/config)
* [`doctor`](/zh/cli/doctor)
* [`dashboard`](/zh/cli/dashboard)
* [`reset`](/zh/cli/reset)
* [`uninstall`](/zh/cli/uninstall)
* [`update`](/zh/cli/update)
* [`message`](/zh/cli/message)
* [`agent`](/zh/cli/agent)
* [`agents`](/zh/cli/agents)
* [`acp`](/zh/cli/acp)
* [`status`](/zh/cli/status)
* [`health`](/zh/cli/health)
* [`sessions`](/zh/cli/sessions)
* [`gateway`](/zh/cli/gateway)
* [`logs`](/zh/cli/logs)
* [`system`](/zh/cli/system)
* [`models`](/zh/cli/models)
* [`memory`](/zh/cli/memory)
* [`nodes`](/zh/cli/nodes)
* [`devices`](/zh/cli/devices)
* [`node`](/zh/cli/node)
* [`approvals`](/zh/cli/approvals)
* [`sandbox`](/zh/cli/sandbox)
* [`tui`](/zh/cli/tui)
* [`browser`](/zh/cli/browser)
* [`cron`](/zh/cli/cron)
* [`dns`](/zh/cli/dns)
* [`docs`](/zh/cli/docs)
* [`hooks`](/zh/cli/hooks)
* [`webhooks`](/zh/cli/webhooks)
* [`pairing`](/zh/cli/pairing)
* [`plugins`](/zh/cli/plugins)（插件相关命令）
* [`channels`](/zh/cli/channels)
* [`security`](/zh/cli/security)
* [`skills`](/zh/cli/skills)
* [`voicecall`](/zh/cli/voicecall)（插件；如已安装）

<div id="global-flags">
  ## 全局参数
</div>

* `--dev`：将状态隔离存放在 `~/.openclaw-dev` 下，并调整默认端口号。
* `--profile <name>`：将状态隔离存放在 `~/.openclaw-<name>` 下。
* `--no-color`：禁用 ANSI 颜色。
* `--update`：`openclaw update` 的简写（仅适用于源码安装）。
* `-V`、`--version`、`-v`：显示版本信息并退出。

<div id="output-styling">
  ## 输出样式
</div>

* ANSI 颜色和进度指示器仅在 TTY 会话中渲染。
* OSC-8 超链接会在受支持的终端中渲染为可点击链接；否则会回退为普通 URL。
* `--json`（以及在支持的情况下 `--plain`）会禁用样式，以获得干净的输出。
* `--no-color` 会禁用 ANSI 样式；同时也会遵循 `NO_COLOR=1`。
* 长时间运行的命令会显示进度指示器（在支持的情况下使用 OSC 9;4）。

<div id="color-palette">
  ## 颜色方案
</div>

OpenClaw 在 CLI 输出中使用“lobster”配色方案。

* `accent` (#FF5A2D)：标题、标签、主要高亮。
* `accentBright` (#FF7A3D)：命令名称、强调内容。
* `accentDim` (#D14A22)：次要高亮文本。
* `info` (#FF8A5B)：信息类值。
* `success` (#2FBF71)：成功状态。
* `warn` (#FFB020)：警告、回退策略、需要注意的内容。
* `error` (#E23D2D)：错误、失败。
* `muted` (#8B7F77)：弱化显示、元数据。

调色板唯一权威来源：`src/terminal/palette.ts`（又名 “lobster seam”）。

<div id="command-tree">
  ## 命令树
</div>

```
openclaw [--dev] [--profile <name>] <command>
  setup
  onboard
  configure
  config
    get
    set
    unset
  doctor
  security
    audit
  reset
  uninstall
  update
  channels
    list
    status
    logs
    add
    remove
    login
    logout
  skills
    list
    info
    check
  plugins
    list
    info
    install
    enable
    disable
    doctor
  memory
    status
    index
    search
  message
  agent
  agents
    list
    add
    delete
  acp
  status
  health
  sessions
  gateway
    call
    health
    status
    probe
    discover
    install
    uninstall
    start
    stop
    restart
    run
  logs
  system
    event
    heartbeat last|enable|disable
    presence
  models
    list
    status
    set
    set-image
    aliases list|add|remove
    fallbacks list|add|remove|clear
    image-fallbacks list|add|remove|clear
    scan
    auth add|setup-token|paste-token
    auth order get|set|clear
  sandbox
    list
    recreate
    explain
  cron
    status
    list
    add
    edit
    rm
    enable
    disable
    runs
    run
  nodes
  devices
  node
    run
    status
    install
    uninstall
    start
    stop
    restart
  approvals
    get
    set
    allowlist add|remove
  browser
    status
    start
    stop
    reset-profile
    tabs
    open
    focus
    close
    profiles
    create-profile
    delete-profile
    screenshot
    snapshot
    navigate
    resize
    click
    type
    press
    hover
    drag
    select
    upload
    fill
    dialog
    wait
    evaluate
    console
    pdf
  hooks
    list
    info
    check
    enable
    disable
    install
    update
  webhooks
    gmail setup|run
  pairing
    list
    approve
  docs
  dns
    setup
  tui
```

注意：插件可以添加新的顶级命令（例如 `openclaw voicecall`）。

<div id="security">
  ## 安全
</div>

* `openclaw security audit` — 审计配置和本地状态，检查常见的安全踩坑/隐患。
* `openclaw security audit --deep` — 尽力对正在运行的 Gateway 进行实时探测。
* `openclaw security audit --fix` — 收紧安全默认值并调整状态/配置文件的权限（chmod）。

<div id="plugins">
  ## 插件
</div>

管理扩展及其配置：

* `openclaw plugins list` — 发现插件（使用 `--json` 获取机器可读输出）。
* `openclaw plugins info <id>` — 显示某个插件的详细信息。
* `openclaw plugins install <path|.tgz|npm-spec>` — 安装插件（或将插件路径添加到 `plugins.load.paths`）。
* `openclaw plugins enable <id>` / `disable <id>` — 切换 `plugins.entries.<id>.enabled`。
* `openclaw plugins doctor` — 诊断并报告插件加载错误。

大多数插件变更需要重启 Gateway。参见 [插件](/zh/plugin)。

<div id="memory">
  ## Memory
</div>

对 `MEMORY.md` 和 `memory/*.md` 进行向量搜索：

* `openclaw memory status` — 显示索引统计信息。
* `openclaw memory index` — 重新为 memory 目录中的文件建立索引。
* `openclaw memory search "<query>"` — 对记忆数据执行语义搜索。

<div id="chat-slash-commands">
  ## 聊天斜杠命令
</div>

聊天消息支持 `/...` 命令（文本和原生）。参见 [/tools/slash-commands](/zh/tools/slash-commands)。

要点：

* 使用 `/status` 进行快速诊断。
* 使用 `/config` 进行持久化配置更改。
* 使用 `/debug` 进行仅在运行时生效的配置覆盖（只存于内存，不写入磁盘；需要 `commands.debug: true`）。

<div id="setup-onboarding">
  ## 安装与初始化
</div>

<div id="setup">
  ### `setup`
</div>

初始化配置和工作区。

选项：

* `--workspace <dir>`: 智能体工作区路径（默认 `~/.openclaw/workspace`）。
* `--wizard`: 运行上手向导。
* `--non-interactive`: 在无交互提示的情况下运行向导。
* `--mode <local|remote>`: 向导模式。
* `--remote-url <url>`: 远程 Gateway URL。
* `--remote-token <token>`: 远程 Gateway 令牌。

当存在任意与向导相关的标志（`--non-interactive`、`--mode`、`--remote-url`、`--remote-token`）时，将自动运行向导。

<div id="onboard">
  ### `onboard`
</div>

用于以交互式向导方式设置 Gateway、工作区和技能。

选项：

* `--workspace <dir>`
* `--reset`（在运行向导前重置配置、凭据、会话和工作区）
* `--non-interactive`
* `--mode <local|remote>`
* `--flow <quickstart|advanced|manual>`（manual 是 advanced 的别名）
* `--auth-choice <setup-token|token|chutes|openai-codex|openai-api-key|openrouter-api-key|ai-gateway-api-key|moonshot-api-key|kimi-code-api-key|synthetic-api-key|venice-api-key|gemini-api-key|zai-api-key|apiKey|minimax-api|minimax-api-lightning|opencode-zen|skip>`
* `--token-provider <id>`（非交互模式；与 `--auth-choice token` 一起使用）
* `--token <token>`（非交互模式；与 `--auth-choice token` 一起使用）
* `--token-profile-id <id>`（非交互模式；默认值：`<provider>:manual`）
* `--token-expires-in <duration>`（非交互模式；例如 `365d`、`12h`）
* `--anthropic-api-key <key>`
* `--openai-api-key <key>`
* `--openrouter-api-key <key>`
* `--ai-gateway-api-key <key>`
* `--moonshot-api-key <key>`
* `--kimi-code-api-key <key>`
* `--gemini-api-key <key>`
* `--zai-api-key <key>`
* `--minimax-api-key <key>`
* `--opencode-zen-api-key <key>`
* `--gateway-port <port>`
* `--gateway-bind <loopback|lan|tailnet|auto|custom>`
* `--gateway-auth <token|password>`
* `--gateway-token <token>`
* `--gateway-password <password>`
* `--remote-url <url>`
* `--remote-token <token>`
* `--tailscale <off|serve|funnel>`
* `--tailscale-reset-on-exit`
* `--install-daemon`
* `--no-install-daemon`（别名：`--skip-daemon`）
* `--daemon-runtime <node|bun>`
* `--skip-channels`
* `--skip-skills`
* `--skip-health`
* `--skip-ui`
* `--node-manager <npm|pnpm|bun>`（推荐使用 pnpm；不推荐将 bun 用于 Gateway 运行时）
* `--json`

<div id="configure">
  ### `configure`
</div>

交互式配置向导（模型、渠道、技能、Gateway）。

<div id="config">
  ### `config`
</div>

非交互式配置辅助命令（get/set/unset）。在没有子命令的情况下运行 `openclaw config` 会启动向导。

子命令：

* `config get <path>`：输出配置值（点/方括号表示的路径）。
* `config set <path> <value>`：设置一个值（JSON5 或原始字符串）。
* `config unset <path>`：删除一个值。

<div id="doctor">
  ### `doctor`
</div>

健康检查与快速修复（配置、Gateway 和旧版服务）。

选项：

* `--no-workspace-suggestions`：禁用工作区记忆提示。
* `--yes`：在不提示的情况下接受默认值（无头模式）。
* `--non-interactive`：跳过交互提示；仅应用安全迁移。
* `--deep`：扫描系统服务，查找额外的 Gateway 安装。

<div id="channel-helpers">
  ## 频道辅助命令
</div>

<div id="channels">
  ### `channels`
</div>

管理聊天渠道账号（WhatsApp/Telegram/Discord/Google Chat/Slack/Mattermost（插件）/Signal/iMessage/MS Teams）。

子命令：

* `channels list`：显示已配置的渠道和认证配置。
* `channels status`：检查 Gateway 连通性和渠道健康状况（`--probe` 会执行额外检查；Gateway 健康检查请使用 `openclaw health` 或 `openclaw status --deep`）。
* 提示：`channels status` 在检测到常见错误配置时，会输出带有修复建议的警告（并引导你使用 `openclaw doctor`）。
* `channels logs`：从 Gateway 日志文件中显示最近的渠道日志。
* `channels add`：在未传入任何 flag 时使用向导模式进行设置；传入 flag 时切换为非交互模式。
* `channels remove`：默认仅禁用；传入 `--delete` 可在无确认提示的情况下删除配置条目。
* `channels login`：交互式渠道登录（仅支持 WhatsApp Web）。
* `channels logout`：登出某个渠道会话（如果该渠道支持）。

通用选项：

* `--channel <name>`：`whatsapp|telegram|discord|googlechat|slack|mattermost|signal|imessage|msteams`
* `--account <id>`：渠道账号 ID（默认 `default`）
* `--name <label>`：该账号的显示名称

`channels login` 选项：

* `--channel <channel>`（默认 `whatsapp`；支持 `whatsapp`/`web`）
* `--account <id>`
* `--verbose`

`channels logout` 选项：

* `--channel <channel>`（默认 `whatsapp`）
* `--account <id>`

`channels list` 选项：

* `--no-usage`：跳过模型提供方的用量/配额快照（仅限基于 OAuth/API 的提供方）。
* `--json`：输出 JSON（默认包含用量信息，除非设置了 `--no-usage`）。

`channels logs` 选项：

* `--channel <name|all>`（默认 `all`）
* `--lines <n>`（默认 `200`）
* `--json`

更多详情：[/concepts/oauth](/zh/concepts/oauth)

示例：

```bash
openclaw channels add --channel telegram --account alerts --name "Alerts Bot" --token $TELEGRAM_BOT_TOKEN
openclaw channels add --channel discord --account work --name "Work Bot" --token $DISCORD_BOT_TOKEN
openclaw channels remove --channel discord --account work --delete
openclaw channels status --probe
openclaw status --deep
```

<div id="skills">
  ### `skills`
</div>

列出并检视可用技能及其就绪状态信息。

子命令：

* `skills list`：列出技能（无子命令时的默认行为）。
* `skills info <name>`：显示单个技能的详细信息。
* `skills check`：汇总已就绪与缺失要求的情况。

选项：

* `--eligible`：仅显示已就绪的技能。
* `--json`：以 JSON 输出（无样式）。
* `-v`, `--verbose`：包含缺失要求的详细信息。

提示：使用 `npx clawhub` 来搜索、安装和同步技能。

<div id="pairing">
  ### `pairing`
</div>

在各通道中审批私信配对请求。

子命令：

* `pairing list <channel> [--json]`
* `pairing approve <channel> <code> [--notify]`

<div id="webhooks-gmail">
  ### `webhooks gmail`
</div>

Gmail Pub/Sub 钩子配置与运行器。参见 [/automation/gmail-pubsub](/zh/automation/gmail-pubsub)。

子命令：

* `webhooks gmail setup`（需要 `--account <email>`，支持 `--project`、`--topic`、`--subscription`、`--label`、`--hook-url`、`--hook-token`、`--push-token`、`--bind`、`--port`、`--path`、`--include-body`、`--max-bytes`、`--renew-minutes`、`--tailscale`、`--tailscale-path`、`--tailscale-target`、`--push-endpoint`、`--json`）
* `webhooks gmail run`（针对上述相同标志提供运行时覆盖）

<div id="dns-setup">
  ### `dns setup`
</div>

广域发现 DNS 辅助工具（CoreDNS + Tailscale）。参见 [/gateway/discovery](/zh/gateway/discovery)。

选项：

* `--apply`: 安装/更新 CoreDNS 配置（需要 sudo；仅限 macOS）。

<div id="messaging-agent">
  ## 消息与智能体
</div>

<div id="message">
  ### `message`
</div>

统一的出站消息和频道操作接口。

参见：[/cli/message](/zh/cli/message)

子命令：

* `message send|poll|react|reactions|read|edit|delete|pin|unpin|pins|permissions|search|timeout|kick|ban`
* `message thread <create|list|reply>`
* `message emoji <list|upload>`
* `message sticker <send|upload>`
* `message role <info|add|remove>`
* `message channel <info|list>`
* `message member info`
* `message voice status`
* `message event <list|create>`

示例：

* `openclaw message send --target +15555550123 --message "Hi"`
* `openclaw message poll --channel discord --target channel:123 --poll-question "Snack?" --poll-option Pizza --poll-option Sushi`

<div id="agent">
  ### `agent`
</div>

通过 Gateway（或使用 `--local` 本地嵌入模式）执行智能体的一次轮次。

必需参数：

* `--message <text>`

可选参数：

* `--to <dest>`（用于会话 key 和可选消息投递）
* `--session-id <id>`
* `--thinking <off|minimal|low|medium|high|xhigh>`（仅适用于 GPT-5.2 和 Codex 模型）
* `--verbose <on|full|off>`
* `--channel <whatsapp|telegram|discord|slack|mattermost|signal|imessage|msteams>`
* `--local`
* `--deliver`
* `--json`
* `--timeout <seconds>`

<div id="agents">
  ### `agents`
</div>

管理彼此隔离的智能体（工作区 + 认证 + 路由）。

<div id="agents-list">
  #### `agents list`
</div>

列出所有已配置的智能体。

选项：

* `--json`
* `--bindings`

<div id="agents-add-name">
  #### `agents add [name]`
</div>

添加一个新的隔离智能体。如果未传入任何标志（或 `--non-interactive`），则会运行引导式向导；在非交互模式下，必须提供 `--workspace`。

选项：

* `--workspace &lt;dir&gt;`
* `--model &lt;id&gt;`
* `--agent-dir &lt;dir&gt;`
* `--bind &lt;channel[:accountId]&gt;`（可重复）
* `--non-interactive`
* `--json`

绑定格式为 `channel[:accountId]`。当用于 WhatsApp 且省略 `accountId` 时，将使用默认账号 ID。

<div id="agents-delete-id">
  #### `agents delete <id>`
</div>

删除一个智能体，并清理其工作区和状态数据。

选项：

* `--force`
* `--json`

<div id="acp">
  ### `acp`
</div>

运行 ACP 桥接程序，将 IDE 连接到 Gateway。

完整选项和示例请参见 [`acp`](/zh/cli/acp)。

<div id="status">
  ### `status`
</div>

显示已关联的会话健康状况和最近的接收方。

选项：

* `--json`
* `--all`（完整诊断；只读，可粘贴）
* `--deep`（探测通道）
* `--usage`（显示模型提供方的用量/配额）
* `--timeout <ms>`
* `--verbose`
* `--debug`（`--verbose` 的别名）

注意：

* 在可用时，概览还会包含 Gateway 和节点宿主服务的状态。

<div id="usage-tracking">
  ### 使用情况追踪
</div>

在可用 OAuth/API 凭据的情况下，OpenClaw 可以展示提供方的使用量/配额。

展示位置：

* `/status`（在可用时添加一行简短的提供方使用信息）
* `openclaw status --usage`（输出提供方使用情况的完整明细）
* macOS 菜单栏（Context 下的 Usage 区域）

说明：

* 数据直接来自提供方的使用情况接口（非估算值）。
* 提供方：Anthropic、GitHub Copilot、OpenAI Codex OAuth，以及在启用相应提供方插件时的 Gemini CLI/Antigravity。
* 如果不存在匹配的凭据，则不会显示使用情况。
* 详细信息：参见[使用情况追踪](/zh/concepts/usage-tracking)。

<div id="health">
  ### `health`
</div>

从运行中的 Gateway 获取健康检查信息。

选项：

* `--json`
* `--timeout &lt;ms&gt;`
* `--verbose`

<div id="sessions">
  ### `sessions`
</div>

列出已保存的会话。

选项：

* `--json`
* `--verbose`
* `--store <path>`
* `--active <minutes>`

<div id="reset-uninstall">
  ## 重置 / 卸载
</div>

<div id="reset">
  ### `reset`
</div>

重置本地配置和状态（保留已安装的 CLI）。

选项：

* `--scope <config|config+creds+sessions|full>`
* `--yes`
* `--non-interactive`
* `--dry-run`

说明：

* `--non-interactive` 需要同时指定 `--scope` 和 `--yes`。

<div id="uninstall">
  ### `uninstall`
</div>

卸载 Gateway 服务及本地数据（CLI 仍会保留）。

选项：

* `--service`
* `--state`
* `--workspace`
* `--app`
* `--all`
* `--yes`
* `--non-interactive`
* `--dry-run`

注意：

* `--non-interactive` 必须同时指定 `--yes` 和显式的 scope（或 `--all`）。

## Gateway

<div id="gateway">
  ### `gateway`
</div>

运行 WebSocket Gateway。

选项：

* `--port <port>`
* `--bind <loopback|tailnet|lan|auto|custom>`
* `--token <token>`
* `--auth <token|password>`
* `--password <password>`
* `--tailscale <off|serve|funnel>`
* `--tailscale-reset-on-exit`
* `--allow-unconfigured`
* `--dev`
* `--reset`（重置开发配置 + 凭据 + 会话 + 工作区）
* `--force`（强制终止当前端口上的现有监听器）
* `--verbose`
* `--claude-cli-logs`
* `--ws-log <auto|full|compact>`
* `--compact`（`--ws-log compact` 的别名）
* `--raw-stream`
* `--raw-stream-path <path>`

<div id="gateway-service">
  ### `gateway service`
</div>

管理 Gateway 服务（launchd/systemd/schtasks）。

子命令：

* `gateway status`（默认探测 Gateway RPC）
* `gateway install`（安装服务）
* `gateway uninstall`
* `gateway start`
* `gateway stop`
* `gateway restart`

说明：

* `gateway status` 默认使用服务解析出的端口/配置探测 Gateway RPC（可通过 `--url/--token/--password` 覆盖）。
* `gateway status` 支持 `--no-probe`、`--deep` 和 `--json`，便于脚本化使用。
* 当能够检测到遗留或额外的 Gateway 服务时，`gateway status` 也会将其展示出来（`--deep` 会增加系统级扫描）。以 profile 命名的 OpenClaw 服务被视为一等公民，不会被标记为“extra”。
* `gateway status` 会打印 CLI 实际使用的配置路径与服务可能使用的配置路径（服务环境）分别是什么，以及解析出的探测目标 URL。
* `gateway install|uninstall|start|stop|restart` 支持 `--json` 以便脚本化使用（默认输出保持对人类友好）。
* `gateway install` 默认使用 Node 运行时；**不推荐** 使用 bun（会导致 WhatsApp/Telegram 相关问题）。
* `gateway install` 支持的选项：`--port`、`--runtime`、`--token`、`--force`、`--json`。

<div id="logs">
  ### `logs`
</div>

通过 RPC 实时查看 Gateway 的文件日志。

注意：

* 在 TTY 会话中会渲染带颜色的结构化视图；非 TTY 则回退为纯文本。
* 使用 `--json` 会输出按行分隔的 JSON（每行一个日志事件）。

示例：

```bash
openclaw logs --follow
openclaw logs --limit 200
openclaw logs --plain
openclaw logs --json
openclaw logs --no-color
```

<div id="gateway-subcommand">
  ### `gateway <subcommand>`
</div>

Gateway CLI 帮助命令（在 RPC 子命令中使用 `--url`、`--token`、`--password`、`--timeout`、`--expect-final`）。

子命令：

* `gateway call <method> [--params <json>]`
* `gateway health`
* `gateway status`
* `gateway probe`
* `gateway discover`
* `gateway install|uninstall|start|stop|restart`
* `gateway run`

常见 RPC 调用：

* `config.apply`（校验 + 写入配置 + 重启 + 唤醒）
* `config.patch`（合并部分更新 + 重启 + 唤醒）
* `update.run`（执行更新 + 重启 + 唤醒）

提示：直接调用 `config.set` / `config.apply` / `config.patch` 时，如果已有配置，请传入来自
`config.get` 的 `baseHash`。

<div id="models">
  ## 模型
</div>

有关回退行为和扫描策略，请参见 [/concepts/models](/zh/concepts/models)。

首选 Anthropic 身份验证方式（setup-token）：

```bash
claude setup-token
openclaw models auth setup-token --provider anthropic
openclaw models status
```

<div id="models-root">
  ### `models`（根级命令）
</div>

`openclaw models` 是 `models status` 的别名命令。

根级选项：

* `--status-json`（`models status --json` 的别名）
* `--status-plain`（`models status --plain` 的别名）

<div id="models-list">
  ### `models list`
</div>

选项：

* `--all`
* `--local`
* `--provider <name>`
* `--json`
* `--plain`

<div id="models-status">
  ### `models status`
</div>

选项：

* `--json`
* `--plain`
* `--check`（退出码 1=已过期/缺失，2=即将过期）
* `--probe`（对已配置的认证配置文件进行实时探测）
* `--probe-provider <name>`
* `--probe-profile <id>`（可重复或用逗号分隔）
* `--probe-timeout <ms>`
* `--probe-concurrency <n>`
* `--probe-max-tokens <n>`

始终包含身份验证概览以及认证存储中各配置文件的 OAuth 过期状态。
`--probe` 会执行实时请求（可能消耗 tokens 并触发速率限制）。

<div id="models-set-model">
  ### `models set <model>`
</div>

设置 `agents.defaults.model.primary`。

<div id="models-set-image-model">
  ### `models set-image <model>`
</div>

将 `agents.defaults.imageModel.primary` 设置为指定的 `<model>`。

<div id="models-aliases-listaddremove">
  ### `models aliases list|add|remove`
</div>

选项：

* `list`: `--json`, `--plain`
* `add <alias> <model>`
* `remove <alias>`

<div id="models-fallbacks-listaddremoveclear">
  ### `models fallbacks list|add|remove|clear`
</div>

选项：

* `list`: `--json`, `--plain`
* `add <model>`
* `remove <model>`
* `clear`

<div id="models-image-fallbacks-listaddremoveclear">
  ### `models image-fallbacks list|add|remove|clear`
</div>

选项：

* `list`: `--json`, `--plain`
* `add <model>`
* `remove <model>`
* `clear`

<div id="models-scan">
  ### `models scan`
</div>

选项：

* `--min-params <b>`
* `--max-age-days <days>`
* `--provider <name>`
* `--max-candidates <n>`
* `--timeout <ms>`
* `--concurrency <n>`
* `--no-probe`
* `--yes`
* `--no-input`
* `--set-default`
* `--set-image`
* `--json`

<div id="models-auth-addsetup-tokenpaste-token">
  ### `models auth add|setup-token|paste-token`
</div>

选项：

* `add`: 交互式认证辅助工具
* `setup-token`: `--provider <name>`（默认为 `anthropic`），`--yes`
* `paste-token`: `--provider <name>`，`--profile-id <id>`，`--expires-in <duration>`

<div id="models-auth-order-getsetclear">
  ### `models auth order get|set|clear`
</div>

选项：

* `get`: `--provider <name>`, `--agent <id>`, `--json`
* `set`: `--provider <name>`, `--agent <id>`, `<profileIds...>`
* `clear`: `--provider <name>`, `--agent <id>`

<div id="system">
  ## 系统
</div>

<div id="system-event">
  ### `system event`
</div>

将系统事件加入队列，并可选触发一次心跳（Gateway RPC）。

必需参数：

* `--text <text>`

可选参数：

* `--mode <now|next-heartbeat>`
* `--json`
* `--url`, `--token`, `--timeout`, `--expect-final`

<div id="system-heartbeat-lastenabledisable">
  ### `system heartbeat last|enable|disable`
</div>

心跳控制（Gateway RPC）。

选项：

* `--json`
* `--url`, `--token`, `--timeout`, `--expect-final`

<div id="system-presence">
  ### `system presence`
</div>

列出系统在线状态条目（Gateway RPC）。

选项：

* `--json`
* `--url`、`--token`、`--timeout`、`--expect-final`

<div id="cron">
  ## Cron
</div>

管理定时任务（Gateway RPC）。参见 [/automation/cron-jobs](/zh/automation/cron-jobs)。

子命令：

* `cron status [--json]`
* `cron list [--all] [--json]`（默认以表格形式输出；使用 `--json` 获取原始输出）
* `cron add`（别名：`create`；需要 `--name`，且 `--at` | `--every` | `--cron` 三者中**恰好一个**，并且在 `--system-event` | `--message` 中**恰好一个**作为负载）
* `cron edit <id>`（修改字段）
* `cron rm <id>`（别名：`remove`、`delete`）
* `cron enable <id>`
* `cron disable <id>`
* `cron runs --id <id> [--limit <n>]`
* `cron run <id> [--force]`

所有 `cron` 命令都支持 `--url`、`--token`、`--timeout`、`--expect-final`。

<div id="node-host">
  ## 节点主机
</div>

`node` 用于运行一个**无头节点主机**，或将其作为后台服务进行管理。参见
[`openclaw node`](/zh/cli/node)。

子命令：

* `node run --host <gateway-host> --port 18789`
* `node status`
* `node install [--host <gateway-host>] [--port <port>] [--tls] [--tls-fingerprint <sha256>] [--node-id <id>] [--display-name <name>] [--runtime <node|bun>] [--force]`
* `node uninstall`
* `node stop`
* `node restart`

<div id="nodes">
  ## 节点
</div>

`nodes` 与 Gateway 通信，并针对已配对的节点执行操作。参见 [/nodes](/zh/nodes)。

通用选项：

* `--url`、`--token`、`--timeout`、`--json`

子命令：

* `nodes status [--connected] [--last-connected <duration>]`
* `nodes describe --node <id|name|ip>`
* `nodes list [--connected] [--last-connected <duration>]`
* `nodes pending`
* `nodes approve <requestId>`
* `nodes reject <requestId>`
* `nodes rename --node <id|name|ip> --name <displayName>`
* `nodes invoke --node <id|name|ip> --command <command> [--params <json>] [--invoke-timeout <ms>] [--idempotency-key <key>]`
* `nodes run --node <id|name|ip> [--cwd <path>] [--env KEY=VAL] [--command-timeout <ms>] [--needs-screen-recording] [--invoke-timeout <ms>] <command...>`（mac 节点或无头节点宿主）
* `nodes notify --node <id|name|ip> [--title <text>] [--body <text>] [--sound <name>] [--priority <passive|active|timeSensitive>] [--delivery <system|overlay|auto>] [--invoke-timeout <ms>]`（仅限 macOS）

摄像头：

* `nodes camera list --node <id|name|ip>`
* `nodes camera snap --node <id|name|ip> [--facing front|back|both] [--device-id <id>] [--max-width <px>] [--quality <0-1>] [--delay-ms <ms>] [--invoke-timeout <ms>]`
* `nodes camera clip --node <id|name|ip> [--facing front|back] [--device-id <id>] [--duration <ms|10s|1m>] [--no-audio] [--invoke-timeout <ms>]`

画布 + 屏幕：

* `nodes canvas snapshot --node <id|name|ip> [--format png|jpg|jpeg] [--max-width <px>] [--quality <0-1>] [--invoke-timeout <ms>]`
* `nodes canvas present --node <id|name|ip> [--target <urlOrPath>] [--x <px>] [--y <px>] [--width <px>] [--height <px>] [--invoke-timeout <ms>]`
* `nodes canvas hide --node <id|name|ip> [--invoke-timeout <ms>]`
* `nodes canvas navigate <url> --node <id|name|ip> [--invoke-timeout <ms>]`
* `nodes canvas eval [<js>] --node <id|name|ip> [--js <code>] [--invoke-timeout <ms>]`
* `nodes canvas a2ui push --node <id|name|ip> (--jsonl <path> | --text <text>) [--invoke-timeout <ms>]`
* `nodes canvas a2ui reset --node <id|name|ip> [--invoke-timeout <ms>]`
* `nodes screen record --node <id|name|ip> [--screen <index>] [--duration <ms|10s>] [--fps <n>] [--no-audio] [--out <path>] [--invoke-timeout <ms>]`

位置信息：

* `nodes location get --node <id|name|ip> [--max-age <ms>] [--accuracy <coarse|balanced|precise>] [--location-timeout <ms>] [--invoke-timeout <ms>]`

<div id="browser">
  ## 浏览器
</div>

浏览器控制 CLI 工具（专用 Chrome/Brave/Edge/Chromium）。参见 [`openclaw browser`](/zh/cli/browser) 和 [Browser 工具](/zh/tools/browser)。

常用选项：

* `--url`, `--token`, `--timeout`, `--json`
* `--browser-profile <name>`

管理：

* `browser status`
* `browser start`
* `browser stop`
* `browser reset-profile`
* `browser tabs`
* `browser open <url>`
* `browser focus <targetId>`
* `browser close [targetId]`
* `browser profiles`
* `browser create-profile --name <name> [--color <hex>] [--cdp-url <url>]`
* `browser delete-profile --name <name>`

检查：

* `browser screenshot [targetId] [--full-page] [--ref <ref>] [--element <selector>] [--type png|jpeg]`
* `browser snapshot [--format aria|ai] [--target-id <id>] [--limit <n>] [--interactive] [--compact] [--depth <n>] [--selector <sel>] [--out <path>]`

操作：

* `browser navigate <url> [--target-id <id>]`
* `browser resize <width> <height> [--target-id <id>]`
* `browser click <ref> [--double] [--button <left|right|middle>] [--modifiers <csv>] [--target-id <id>]`
* `browser type <ref> <text> [--submit] [--slowly] [--target-id <id>]`
* `browser press <key> [--target-id <id>]`
* `browser hover <ref> [--target-id <id>]`
* `browser drag <startRef> <endRef> [--target-id <id>]`
* `browser select <ref> <values...> [--target-id <id>]`
* `browser upload <paths...> [--ref <ref>] [--input-ref <ref>] [--element <selector>] [--target-id <id>] [--timeout-ms <ms>]`
* `browser fill [--fields <json>] [--fields-file <path>] [--target-id <id>]`
* `browser dialog --accept|--dismiss [--prompt <text>] [--target-id <id>] [--timeout-ms <ms>]`
* `browser wait [--time <ms>] [--text <value>] [--text-gone <value>] [--target-id <id>]`
* `browser evaluate --fn <code> [--ref <ref>] [--target-id <id>]`
* `browser console [--level <error|warn|info>] [--target-id <id>]`
* `browser pdf [--target-id <id>]`

<div id="docs-search">
  ## 文档搜索
</div>

<div id="docs-query">
  ### `docs [query...]`
</div>

搜索在线文档索引。

## TUI

<div id="tui">
  ### `tui`
</div>

打开与 Gateway 相连的终端 UI。

选项：

* `--url <url>`
* `--token <token>`
* `--password <password>`
* `--session <key>`
* `--deliver`
* `--thinking <level>`
* `--message <text>`
* `--timeout-ms <ms>`（默认为 `agents.defaults.timeoutSeconds`）
* `--history-limit <n>`