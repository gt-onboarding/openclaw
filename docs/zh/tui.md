---
title: TUI
summary: "终端 UI（TUI）：从任何机器连接到 Gateway"
read_when:
  - 你想要一份面向初学者的 TUI 入门指引
  - 你需要完整的 TUI 功能、命令和快捷键列表
---

<div id="tui-terminal-ui">
  # TUI（终端 UI）
</div>

<div id="quick-start">
  ## 快速入门
</div>

1. 启动 Gateway。

```bash
openclaw gateway
```

2. 打开 TUI 界面。

```bash
openclaw tui
```

3. 输入一条消息并按 Enter 键发送。

远程 Gateway：

```bash
openclaw tui --url ws://<host>:<port> --token <gateway-token>
```

如果你的 Gateway 启用了密码认证，请使用 `--password` 参数。

<div id="what-you-see">
  ## 你会看到什么
</div>

* 顶部栏：连接 URL、当前智能体、当前会话。
* 聊天记录：用户消息、助手回复、系统通知、工具卡片。
* 状态栏：连接/运行状态（连接中、运行中、流式输出中、空闲、错误）。
* 页脚：连接状态 + 智能体 + 会话 + 模型 + 思考/详细/推理 + token 计数 + 发送状态。
* 输入栏：带自动补全功能的文本编辑器。

<div id="mental-model-agents-sessions">
  ## 思维模型：智能体 + 会话
</div>

* 智能体由唯一的 slug 标识（例如 `main`、`research`）。Gateway 会提供智能体列表。
* 会话隶属于当前智能体。
* 会话键存储为 `agent:<agentId>:<sessionKey>`。
  * 如果你输入 `/session main`，TUI 会将其展开为 `agent:<currentAgent>:main`。
  * 如果你输入 `/session agent:other:main`，则会显式切换到该智能体的会话。
* 会话的 scope：
  * `per-sender`（默认）：每个智能体有多个会话。
  * `global`：TUI 始终使用 `global` 会话（选择器可能为空）。
* 当前智能体和会话始终显示在页脚区域。

<div id="sending-delivery">
  ## 发送与投递
</div>

* 消息会先发送到 Gateway；向提供方的投递默认关闭。
* 开启投递：
  * `/deliver on`
  * 或通过 Settings 面板
  * 或使用 `openclaw tui --deliver` 启动

<div id="pickers-overlays">
  ## 选择器与浮层
</div>

* 模型选择器：列出可用模型并设置会话级覆盖设置。
* Agent 选择器：选择其他 Agent 代理。
* 会话选择器：仅显示当前智能体的会话。
* 设置：切换消息投递、工具输出展开和思考过程可见性。

<div id="keyboard-shortcuts">
  ## 键盘快捷键
</div>

* Enter：发送消息
* Esc：终止当前运行
* Ctrl+C：清空输入（按两次退出）
* Ctrl+D：退出
* Ctrl+L：模型选择器
* Ctrl+G：智能体选择器
* Ctrl+P：会话选择器
* Ctrl+O：切换工具输出展开/折叠
* Ctrl+T：切换思考内容显示（会重新加载历史记录）

<div id="slash-commands">
  ## 斜杠命令
</div>

核心：

* `/help`
* `/status`
* `/agent <id>`（或 `/agents`）
* `/session <key>`（或 `/sessions`）
* `/model <provider/model>`（或 `/models`）

会话控制：

* `/think <off|minimal|low|medium|high>`
* `/verbose <on|full|off>`
* `/reasoning <on|off|stream>`
* `/usage <off|tokens|full>`
* `/elevated <on|off|ask|full>`（别名：`/elev`）
* `/activation <mention|always>`
* `/deliver <on|off>`

会话生命周期：

* `/new` 或 `/reset`（重置会话）
* `/abort`（中止当前执行）
* `/settings`
* `/exit`

其他 Gateway 斜杠命令（例如 `/context`）会被转发到 Gateway，并以系统输出的形式显示。参见[斜杠命令](/zh/tools/slash-commands)。

<div id="local-shell-commands">
  ## 本地 shell 命令
</div>

* 在行首添加 `!`，即可在 TUI 所在主机上运行本地 shell 命令。
* TUI 每个会话只会提示一次是否允许本地执行；如果拒绝，本会话中将保持禁用 `!`。
* 命令会在一个全新的、非交互式 shell 中，以 TUI 工作目录为当前目录运行（不会保留对 `cd`/环境变量的更改）。
* 单独输入 `!` 会被当作普通消息发送；前导空格不会触发本地执行。

<div id="tool-output">
  ## 工具输出
</div>

* 工具调用会以卡片形式展示，显示参数和结果。
* 使用 Ctrl+O 在折叠视图和展开视图之间切换。
* 工具运行期间，中间更新会以流式方式写入同一张卡片。

<div id="history-streaming">
  ## 历史记录与流式传输
</div>

* 建立连接时，TUI 会加载最新的历史记录（默认 200 条消息）。
* 流式响应会就地实时更新，直到最终完成。
* TUI 还会监听智能体工具事件，以提供更丰富的工具卡片。

<div id="connection-details">
  ## 连接详情
</div>

* TUI 会以 `mode: "tui"` 的形式向 Gateway 注册。
* 重新连接时会显示一条系统消息；事件间断会记录在日志中。

<div id="options">
  ## 选项
</div>

* `--url <url>`: Gateway WebSocket URL（默认使用配置中的值，或 `ws://127.0.0.1:<port>`）
* `--token <token>`: Gateway token（如需要）
* `--password <password>`: Gateway 密码（如需要）
* `--session <key>`: 会话 key（默认：`main`，当 scope 为 `global` 时则为 `global`）
* `--deliver`: 将助手回复转发到提供方（默认关闭）
* `--thinking <level>`: 覆盖发送时使用的思考级别
* `--timeout-ms <ms>`: Agent 代理超时时间（毫秒）（默认取自 `agents.defaults.timeoutSeconds`）

<div id="troubleshooting">
  ## 故障排查
</div>

发送消息后没有输出：

* 在 TUI 中运行 `/status`，确认 Gateway 已连接且处于空闲/忙碌状态。
* 检查 Gateway 日志：`openclaw logs --follow`。
* 确认智能体可以运行：`openclaw status` 和 `openclaw models status`。
* 如果你期望在聊天通道中看到消息，请启用消息投递（`/deliver on` 或 `--deliver`）。
* `--history-limit <n>`：要加载的历史记录条目数（默认 200）

<div id="troubleshooting">
  ## 故障排查
</div>

* `disconnected`：确保 Gateway 正在运行，并且你的 `--url/--token/--password` 参数正确。
* 选择器中没有智能体：检查 `openclaw agents list` 命令输出以及你的路由配置。
* 会话选择器为空：你可能处于全局 scope，或者当前还没有任何会话。