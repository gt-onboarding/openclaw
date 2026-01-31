---
title: 高权限
summary: "高权限执行模式和 /elevated 指令"
read_when:
  - 调整高权限模式的默认设置、允许列表或斜杠命令行为
---

<div id="elevated-mode-elevated-directives">
  # 提权模式（/elevated 指令）
</div>

<div id="what-it-does">
  ## 它的作用
</div>

- `/elevated on` 在 Gateway 主机上运行，并保留执行审批（效果与 `/elevated ask` 相同）。
- `/elevated full` 在 Gateway 主机上运行，**并且** 自动批准执行（跳过执行审批）。
- `/elevated ask` 在 Gateway 主机上运行，但保留执行审批（效果与 `/elevated on` 相同）。
- `on`/`ask` **不会** 强制设置 `exec.security=full`；已配置的安全/询问策略仍然生效。
- 仅在智能体运行于**沙箱**中时才会改变行为（否则 exec 本就已在主机上运行）。
- 指令形式：`/elevated on|off|ask|full`, `/elev on|off|ask|full`。
- 只接受 `on|off|ask|full`；其他任何内容都会返回提示，且不会改变状态。

<div id="what-it-controls-and-what-it-doesnt">
  ## 它控制什么（以及不控制什么）
</div>

- **可用性门控**：`tools.elevated` 是全局基线。`agents.list[].tools.elevated` 可以在智能体级别进一步收紧提升权限（两者都必须允许）。
- **按会话的状态**：`/elevated on|off|ask|full` 为当前会话对应的 session key 设置提升权限级别。
- **内联指令**：消息内的 `/elevated on|ask|full` 仅对该条消息生效。
- **群组**：在群聊中，仅当提到该智能体时，提升指令才会生效。仅包含命令、从而绕过“必须提及”要求的消息，视为已提及。
- **宿主执行**：提升权限会强制在 Gateway 宿主机上执行 `exec`；`full` 还会设置 `security=full`。
- **审批**：`full` 会跳过 exec 审批；当允许列表/ask 规则要求审批时，`on`/`ask` 会遵循审批流程。
- **未沙箱化的智能体**：对执行位置是 no-op，只影响门控、日志记录和状态。
- **仍然受工具策略约束**：如果工具策略拒绝 `exec`，则无法使用提升权限。
- **与 `/exec` 独立**：`/exec` 为已授权发送方调整每会话默认值，并且不需要提升权限。

<div id="resolution-order">
  ## 生效顺序
</div>

1. 消息上的内联指令（仅对该消息生效）。
2. 会话级覆盖（通过发送仅包含指令的消息进行设置）。
3. 全局默认值（配置中的 `agents.defaults.elevatedDefault`）。

<div id="setting-a-session-default">
  ## 设置会话默认值
</div>

- 发送一条**仅**包含指令的消息（允许有空白字符），例如 `/elevated full`。
- 系统会发回确认回复（`Elevated mode set to full...` / `Elevated mode disabled.`）。
- 如果提升模式访问被禁用，或发送方不在已批准的允许列表中，该指令会返回一条可操作的错误信息，并且不会更改会话状态。
- 发送不带任何参数的 `/elevated`（或 `/elevated:`）即可查看当前的提升模式级别。

<div id="availability-allowlists">
  ## 可用性与允许列表
</div>

- 功能开关：`tools.elevated.enabled`（即使代码支持，也可以通过配置将默认值设为关闭）。
- 发送方允许列表：`tools.elevated.allowFrom`，支持按提供方设置允许列表（例如 `discord`、`whatsapp`）。
- 按智能体开关：`agents.list[].tools.elevated.enabled`（可选；只能在此基础上进一步收紧权限）。
- 按智能体允许列表：`agents.list[].tools.elevated.allowFrom`（可选；一旦设置，发送方必须**同时**匹配全局和按智能体的允许列表）。
- Discord 回退：如果省略 `tools.elevated.allowFrom.discord`，则会回退使用 `channels.discord.dm.allowFrom` 列表。设置 `tools.elevated.allowFrom.discord`（即使是 `[]`）即可覆盖回退逻辑。按智能体的允许列表**不会**使用该回退。
- 所有开关都必须通过检查；否则将视为 elevated 模式不可用。

<div id="logging-status">
  ## 日志与状态
</div>

- 提权执行调用会以 info 级别记录到日志中。
- 会话状态中会包含提权模式（例如 `elevated=ask`、`elevated=full`）。