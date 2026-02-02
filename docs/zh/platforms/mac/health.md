---
title: 运行状况
summary: "macOS 应用如何上报 Gateway/Baileys 运行状况"
read_when:
  - 调试 macOS 应用的运行状况指示器
---

<div id="health-checks-on-macos">
  # macOS 上的健康检查
</div>

如何在菜单栏应用中查看已连接通道的健康状况。

<div id="menu-bar">
  ## 菜单栏
</div>

* 状态指示点现在反映 Baileys 的健康状态：
  * 绿色：已链接，且最近成功打开 socket 连接。
  * 橙色：正在连接或重试中。
  * 红色：已登出或探测失败。
* 第二行会显示 &quot;linked · auth 12m&quot;，或显示失败原因。
* &quot;Run Health Check&quot; 菜单项会触发一次按需健康检查探测。

<div id="settings">
  ## 设置
</div>

* “General” 选项卡新增一个 “Health” 卡片，显示：关联认证的创建时间、会话存储路径/数量、上次检查时间、上次错误/状态码，以及 “Run Health Check”/“Reveal Logs” 按钮。
* 使用缓存快照，使 UI 即刻加载，并在离线时平滑降级。
* **Channels 选项卡** 展示并控制 WhatsApp/Telegram 通道状态（登录二维码、登出、探测、上次断开连接/错误）。

<div id="how-the-probe-works">
  ## 探针的工作方式
</div>

* 应用通过 `ShellExecutor` 大约每 60 秒以及在按需检测时运行一次 `openclaw health --json`。该探针会加载凭据并报告状态，但不会发送消息。
* 分别缓存最近一次正常的快照和最近一次错误，以避免界面闪烁；并显示各自的时间戳。

<div id="when-in-doubt">
  ## 不确定时
</div>

* 你仍然可以使用 [Gateway health](/zh/gateway/health) 中的 CLI 诊断流程（`openclaw status`、`openclaw status --deep`、`openclaw health --json`），并通过 `tail` 查看 `/tmp/openclaw/openclaw-*.log` 日志，留意其中的 `web-heartbeat` / `web-reconnect`。