---
title: 节点
summary: "节点：配对、功能、权限，以及用于 canvas/camera/screen/system 的 CLI 辅助工具"
read_when:
  - 将 iOS/Android 节点与 Gateway 配对
  - 使用节点 canvas/camera 为智能体提供上下文
  - 添加新的节点命令或 CLI 辅助工具
---

<div id="nodes">
  # 节点
</div>

**节点** 是一台配套设备（macOS/iOS/Android/无头）通过 Gateway 的 **WebSocket**（与运维端使用相同端口）连接，使用 `role: "node"`，并通过 `node.invoke` 暴露命令接口（例如 `canvas.*`、`camera.*`、`system.*`）。协议详情见：[Gateway protocol](/zh/gateway/protocol)。

旧版传输方式：[Bridge protocol](/zh/gateway/bridge-protocol)（TCP JSONL；已弃用并从当前节点中移除）。

macOS 也可以以**节点模式**运行：菜单栏 App 会连接到 Gateway 的 WS 服务器，并将其本地的 canvas/camera 命令作为一个节点暴露出来（因此可以针对这台 Mac 使用 `openclaw nodes …`）。

注意：

* 节点是**外设**，不是 Gateway。它们不会运行 Gateway 服务。
* Telegram/WhatsApp 等渠道的消息会到达**Gateway**，而不是节点。

<div id="pairing-status">
  ## 配对与状态
</div>

**WS 节点使用设备配对。** 节点在 `connect` 时会提供设备标识；Gateway
会为 `role: node` 创建一个设备配对请求。请在设备 CLI（或 UI）中批准。

CLI 快速操作：

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
```

注意：

* 当设备的配对角色包含 `node` 时，`nodes status` 会将该节点标记为已**配对**。
* `node.pair.*`（CLI：`openclaw nodes pending/approve/reject`）是一个独立的、由 Gateway 管理的
  节点配对存储；它**不会**作为 WS `connect` 握手的准入条件。

<div id="remote-node-host-systemrun">
  ## 远程节点主机（system.run）
</div>

当你的 Gateway 运行在一台机器上，而你希望在另一台机器上执行命令时，应使用 **节点主机**。模型仍然与 **Gateway** 通信；当选择 `host=node` 时，Gateway 会将 `exec` 调用转发到 **节点主机**。

<div id="what-runs-where">
  ### 各组件的运行位置
</div>

* **Gateway 主机**：接收消息、运行模型并路由工具调用。
* **节点主机**：在节点机器上执行 `system.run`/`system.which`。
* **审批**：在节点主机上通过 `~/.openclaw/exec-approvals.json` 进行强制控制。

<div id="start-a-node-host-foreground">
  ### 以前台方式启动节点主机
</div>

在运行节点的机器上：

```bash
openclaw node run --host <gateway-host> --port 18789 --display-name "Build Node"
```

<div id="start-a-node-host-service">
  ### 启动节点主机（服务）
</div>

```bash
openclaw node install --host <gateway-host> --port 18789 --display-name "Build Node"
openclaw node restart
```

<div id="pair-name">
  ### 配对并命名
</div>

在 Gateway 主机上：

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
openclaw nodes list
```

命名选项：

* 在执行 `openclaw node run` / `openclaw node install` 时使用 `--display-name`（会持久化到该节点上的 `~/.openclaw/node.json` 中）。
* 使用 `openclaw nodes rename --node <id|name|ip> --name "Build Node"`（在 Gateway 侧进行覆盖）。

<div id="allowlist-the-commands">
  ### 将命令加入允许列表
</div>

执行（exec）审批是**按节点主机**生效的。请从 Gateway 添加允许列表条目：

```bash
openclaw approvals allowlist add --node <id|name|ip> "/usr/bin/uname"
openclaw approvals allowlist add --node <id|name|ip> "/usr/bin/sw_vers"
```

审批记录位于节点主机上的 `~/.openclaw/exec-approvals.json` 文件中。

<div id="point-exec-at-the-node">
  ### 将 exec 指向该节点
</div>

配置默认值（Gateway 配置）：

```bash
openclaw config set tools.exec.host node
openclaw config set tools.exec.security allowlist
openclaw config set tools.exec.node "<id-or-name>"
```

或者按会话划分：

```
/exec host=node security=allowlist node=<id-or-name>
```

一旦设置完成，任何带有 `host=node` 的 `exec` 调用都会在该节点主机上运行（受节点允许列表和审批的约束）。

相关内容：

* [节点主机 CLI](/zh/cli/node)
* [Exec 工具](/zh/tools/exec)
* [Exec 审批](/zh/tools/exec-approvals)

<div id="invoking-commands">
  ## 调用命令
</div>

底层（原始 RPC）：

```bash
openclaw nodes invoke --node <idOrNameOrIp> --command canvas.eval --params '{"javaScript":"location.href"}'
```

针对常见的「向 Agent 代理提供 MEDIA 附件」类工作流，提供了更高层级的辅助函数。

<div id="screenshots-canvas-snapshots">
  ## 截图（Canvas 快照）
</div>

如果节点当前正在显示 Canvas（WebView），`canvas.snapshot` 会返回 `{ format, base64 }`。

CLI 辅助工具（写入一个临时文件并输出 `MEDIA:<path>`）：

```bash
openclaw nodes canvas snapshot --node <idOrNameOrIp> --format png
openclaw nodes canvas snapshot --node <idOrNameOrIp> --format jpg --max-width 1200 --quality 0.9
```

<div id="canvas-controls">
  ### 画布控制
</div>

```bash
openclaw nodes canvas present --node <idOrNameOrIp> --target https://example.com
openclaw nodes canvas hide --node <idOrNameOrIp>
openclaw nodes canvas navigate https://example.com --node <idOrNameOrIp>
openclaw nodes canvas eval --node <idOrNameOrIp> --js "document.title"
```

Notes:

* `canvas present` 接受 URL 或本地文件路径（`--target`），可选使用 `--x/--y/--width/--height` 来指定位置和尺寸。
* `canvas eval` 接受内联 JS（`--js`）或位置参数。

<div id="a2ui-canvas">
  ### A2UI（画布）
</div>

```bash
openclaw nodes canvas a2ui push --node <idOrNameOrIp> --text "你好"
openclaw nodes canvas a2ui push --node <idOrNameOrIp> --jsonl ./payload.jsonl
openclaw nodes canvas a2ui reset --node <idOrNameOrIp>
```

注意事项：

* 仅支持 A2UI v0.8 JSONL 格式（v0.9/createSurface 会被拒绝）。

<div id="photos-videos-node-camera">
  ## 照片 + 视频（节点摄像头）
</div>

照片（`jpg`）：

```bash
openclaw nodes camera list --node <idOrNameOrIp>
openclaw nodes camera snap --node <idOrNameOrIp>            # 默认：两个朝向（2 条 MEDIA 行）
openclaw nodes camera snap --node <idOrNameOrIp> --facing front
```

视频片段（`mp4`）：

```bash
openclaw nodes camera clip --node <idOrNameOrIp> --duration 10s
openclaw nodes camera clip --node <idOrNameOrIp> --duration 3000 --no-audio
```

注意：

* 对于 `canvas.*` 和 `camera.*`，节点必须处于**前台**（在后台发起的调用会返回 `NODE_BACKGROUND_UNAVAILABLE`）。
* 为避免 base64 负载过大，会对剪辑时长进行限制（目前为 `<= 60s`）。
* 在可能的情况下，Android 会弹出 `CAMERA` / `RECORD_AUDIO` 权限请求；若权限被拒绝，则会以 `*_PERMISSION_REQUIRED` 错误失败。

<div id="screen-recordings-nodes">
  ## 屏幕录制（节点）
</div>

节点提供 `screen.record`（mp4）。示例：

```bash
openclaw nodes screen record --node <idOrNameOrIp> --duration 10s --fps 10
openclaw nodes screen record --node <idOrNameOrIp> --duration 10s --fps 10 --no-audio
```

Notes:

* `screen.record` 需要节点应用处于前台（活动）状态。
* Android 会在开始录制前显示系统屏幕捕获提示。
* 屏幕录制时长限制为 `<= 60s`。
* `--no-audio` 会禁用麦克风录音（在 iOS/Android 上支持；macOS 使用系统级音频捕获）。
* 当存在多个屏幕时，使用 `--screen <index>` 选择要录制的显示器。

<div id="location-nodes">
  ## 位置（节点）
</div>

当在设置中启用 Location 功能时，节点会提供 `location.get`。

CLI 辅助命令：

```bash
openclaw nodes location get --node <idOrNameOrIp>
openclaw nodes location get --node <idOrNameOrIp> --accuracy precise --max-age 15000 --location-timeout 10000
```

注意：

* 位置 **默认关闭**。
* “始终允许”（Always）需要系统权限；后台获取是尽力而为的。
* 响应包括经纬度、精度（米）和时间戳。

<div id="sms-android-nodes">
  ## SMS（Android 节点）
</div>

当用户授予 **SMS** 权限且设备支持电话/蜂窝通信功能时，Android 节点可以暴露 `sms.send`。

底层调用：

```bash
openclaw nodes invoke --node <idOrNameOrIp> --command sms.send --params '{"to":"+15555550123","message":"Hello from OpenClaw"}'
```

注意：

* 必须先在 Android 设备上接受权限提示，然后该能力才会被广播。
* 仅支持 Wi‑Fi 且不具备电话功能的设备不会广播 `sms.send`。

<div id="system-commands-node-host-mac-node">
  ## 系统命令（节点主机 / Mac 节点）
</div>

macOS 节点提供了 `system.run`、`system.notify` 和 `system.execApprovals.get/set`。
无头节点主机提供了 `system.run`、`system.which` 和 `system.execApprovals.get/set`。

示例：

```bash
openclaw nodes run --node <idOrNameOrIp> -- echo "Hello from mac node"
openclaw nodes notify --node <idOrNameOrIp> --title "Ping" --body "Gateway ready"
```

Notes:

* `system.run` 会在 payload 中返回 stdout/stderr/退出码。
* `system.notify` 会遵守 macOS 应用上的通知权限状态。
* `system.run` 支持 `--cwd`、`--env KEY=VAL`、`--command-timeout` 和 `--needs-screen-recording`。
* `system.notify` 支持 `--priority <passive|active|timeSensitive>` 和 `--delivery <system|overlay|auto>`。
* macOS 节点会丢弃对 `PATH` 的重写；无头节点主机仅在提供的 `PATH` 只是给节点主机现有 PATH 添加前缀时才接受该值。
* 在 macOS 节点模式下，`system.run` 受 macOS 应用中的执行审批（Settings → Exec approvals）约束。
  Ask/allowlist/full 的行为与无头节点主机相同；被拒绝的请求会返回 `SYSTEM_RUN_DENIED`。
* 在无头节点主机上，`system.run` 受执行审批（`~/.openclaw/exec-approvals.json`）约束。

<div id="exec-node-binding">
  ## Exec 节点绑定
</div>

当存在多个可用节点时，你可以将 exec 绑定到特定节点。
这会为 `exec host=node` 设置默认节点（也可以按智能体单独覆盖）。

全局默认值：

```bash
openclaw config set tools.exec.node "node-id-or-name"
```

按智能体的覆盖设置：

```bash
openclaw config get agents.list
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
```

留空以允许任意节点：

```bash
openclaw config unset tools.exec.node
openclaw config unset agents.list[0].tools.exec.node
```

<div id="permissions-map">
  ## 权限映射
</div>

节点可以在 `node.list` / `node.describe` 中包含一个 `permissions` 映射，其键为权限名称（例如 `screenRecording`、`accessibility`），值为布尔类型（`true` = 已授予）。

<div id="headless-node-host-cross-platform">
  ## 无头节点主机（跨平台）
</div>

OpenClaw 可以运行一个**无头节点主机**（无 UI），它会连接到 Gateway 的
WebSocket，并暴露 `system.run` / `system.which` 能力。这在 Linux/Windows 上，
或在服务器同机运行一个精简节点时很有用。

启动方法：

```bash
openclaw node run --host <gateway-host> --port 18789
```

注意：

* 仍然需要进行配对（Gateway 会显示节点审批提示）。
* 节点主机会将其节点 ID、token、显示名称和 Gateway 连接信息保存在 `~/.openclaw/node.json` 中。
* Exec 审批在本地通过 `~/.openclaw/exec-approvals.json` 强制执行
  （参见 [Exec approvals](/zh/tools/exec-approvals)）。
* 在 macOS 上，无头节点主机在可达时会优先使用配套应用作为执行主机，当应用不可用时回退到本地执行。设置 `OPENCLAW_NODE_EXEC_HOST=app` 以强制要求使用配套应用，或设置 `OPENCLAW_NODE_EXEC_FALLBACK=0` 以禁用回退。
* 当 Gateway 的 WS 使用 TLS 时，添加 `--tls` / `--tls-fingerprint`。

<div id="mac-node-mode">
  ## Mac 节点模式
</div>

* macOS 菜单栏应用会作为一个节点连接到 Gateway 的 WS 服务器（从而可以通过 `openclaw nodes …` 管理这台 Mac）。
* 在远程模式下，该应用会为 Gateway 端口建立 SSH 隧道，并连接到 `localhost`。