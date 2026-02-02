---
title: 菜单栏
summary: "菜单栏状态逻辑以及呈现给用户的内容"
read_when:
  - 调整 macOS 菜单栏 UI 或状态逻辑
---

<div id="menu-bar-status-logic">
  # 菜单栏状态显示逻辑
</div>

<div id="what-is-shown">
  ## 显示内容
</div>

- 我们会在菜单栏图标以及菜单的第一行状态中呈现当前智能体的工作状态。
- 在有工作进行时会隐藏健康状态指示；当所有会话空闲时健康状态会重新显示。
- 菜单中的 “Nodes” 模块只列出**设备**（通过 `node.list` 配对的节点），不包含客户端/在线状态条目。
- 当提供方用量快照可用时，会在 Context 下方显示 “Usage” 区域。

<div id="state-model">
  ## 状态模型
</div>

- 会话：事件到达时会在负载中携带 `runId`（每次运行）以及 `sessionKey`。`main` 会话的键为 `main`；如果不存在，则回退到最近更新的会话。
- 优先级：`main` 始终优先。如果 `main` 处于活跃状态，则立即显示其状态。如果 `main` 空闲，则显示最近一次活跃的非 `main` 会话。在活动中途不会来回切换；只会在当前会话变为空闲或 `main` 变为活跃时才切换。
- 活动类型：
  - `job`：高级别命令执行（`state: started|streaming|done|error`）。
  - `tool`：`phase: start|result`，并携带 `toolName` 和 `meta/args`。

<div id="iconstate-enum-swift">
  ## IconState 枚举类型（Swift）
</div>

- `idle`
- `workingMain(ActivityKind)`
- `workingOther(ActivityKind)`
- `overridden(ActivityKind)`（调试用 override）

<div id="activitykind-glyph">
  ### ActivityKind → 图标
</div>

- `exec` → 💻
- `read` → 📄
- `write` → ✍️
- `edit` → 📝
- `attach` → 📎
- default → 🛠️

<div id="visual-mapping">
  ### 视觉映射
</div>

- `idle`: 普通小生物。
- `workingMain`: 带图标的徽章、完整色调、腿部“工作中”动画。
- `workingOther`: 带图标的徽章、弱化色调、无奔跑动画。
- `overridden`: 无论活动状态如何，都使用所选的图标/色调。

<div id="status-row-text-menu">
  ## 状态行文本（菜单）
</div>

- 当有工作在进行时：`<Session role> · <activity label>`
  - 示例：`Main · exec: pnpm test`、`Other · read: apps/macos/Sources/OpenClaw/AppState.swift`。
- 空闲时：则显示健康状况摘要。

<div id="event-ingestion">
  ## 事件接收
</div>

- 来源：控制通道中的 `agent` 事件（`ControlChannel.handleAgentEvent`）。
- 解析出的字段：
  - `stream: "job"`，通过 `data.state` 标记开始/停止。
  - `stream: "tool"`，包含 `data.phase`、`name`，以及可选的 `meta`/`args`。
- 标签：
  - `exec`：取自 `args.command` 的第一行。
  - `read`/`write`：缩短后的路径。
  - `edit`：路径加上从 `meta`/diff 计数推断出的变更类型。
  - 回退：使用工具名称。

<div id="debug-override">
  ## 调试覆盖
</div>

- Settings ▸ Debug ▸ “Icon override” 选择器：
  - `System (auto)`（默认）
  - `Working: main`（按工具类型）
  - `Working: other`（按工具类型）
  - `Idle`
- 通过 `@AppStorage("iconOverride")` 进行存储，并映射到 `IconState.overridden`。

<div id="testing-checklist">
  ## 测试清单
</div>

- 触发主会话任务：验证图标立即切换，且状态行显示主会话标签。
- 在主会话空闲时触发非主会话任务：图标/状态显示为非主会话，并在任务结束前保持不变。
- 在其他会话处于活动状态时启动主会话：图标应立即切换为主会话。
- 工具高频连续触发：确保徽标不闪烁（工具结果有 TTL 宽限期）。
- 当所有会话空闲时，健康状态行应重新显示。