---
title: iOS 节点
summary: "iOS 节点应用：连接 Gateway、配对、画布以及故障排查"
read_when:
  - 为 iOS 节点配对或重新连接
  - 从源代码运行 iOS 应用
  - 调试 Gateway 发现流程或画布命令
---

<div id="ios-app-node">
  # iOS 应用（节点）
</div>

可用性：内部预览版本。iOS 应用尚未公开发布。

<div id="what-it-does">
  ## 它的功能
</div>

* 通过 WebSocket（局域网或 tailnet）连接到 Gateway。
* 暴露节点能力：Canvas、屏幕快照、摄像头采集、定位、对话模式、语音唤醒。
* 接收 `node.invoke` 命令并上报节点状态事件。

<div id="requirements">
  ## 要求
</div>

* 在另一台设备上运行的 Gateway（运行于 macOS、Linux，或通过 WSL2 运行于 Windows）。
* 网络路径：
  * 通过 Bonjour 在同一局域网（LAN），**或**
  * 在 Tailnet 中通过单播 DNS-SD（示例域名：`openclaw.internal.`），**或**
  * 手动指定主机/端口（备用方案）。

<div id="quick-start-pair-connect">
  ## 快速开始（配对与连接）
</div>

1. 启动 Gateway：

```bash
openclaw gateway --port 18789
```

2. 在 iOS 应用中，打开“Settings”，并选择一个已发现的 Gateway（或启用“Manual Host”并输入主机/端口）。

3. 在 Gateway 所在主机上批准配对请求：

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
```

4. 验证连接：

```bash
openclaw nodes status
openclaw gateway call node.list --params "{}"
```

<div id="discovery-paths">
  ## 发现方式
</div>

<div id="bonjour-lan">
  ### Bonjour（局域网）
</div>

Gateway 会在 `local.` 上广播 `_openclaw-gw._tcp`。iOS 应用会自动显示这些网关。

<div id="tailnet-cross-network">
  ### Tailnet（跨网络）
</div>

如果 mDNS 被屏蔽，则使用单播 DNS-SD 区域（选择一个域名；示例：`openclaw.internal.`），并启用 Tailscale 的 split DNS 功能。
有关 CoreDNS 示例，请参见 [Bonjour](/zh/gateway/bonjour)。

<div id="manual-hostport">
  ### 手动主机/端口
</div>

在“Settings”中启用 **Manual Host**，然后输入 Gateway 的主机和端口（默认端口为 `18789`）。

<div id="canvas-a2ui">
  ## Canvas + A2UI
</div>

iOS 节点会渲染一个 WKWebView 画布。使用 `node.invoke` 对其进行控制：

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.navigate --params '{"url":"http://<gateway-host>:18793/__openclaw__/canvas/"}'
```

注意事项：

* Gateway 画布主机会提供 `/__openclaw__/canvas/` 和 `/__openclaw__/a2ui/` 服务。
* 当画布主机 URL 被广播时，iOS 节点在连接后会自动跳转到 A2UI。
* 使用 `canvas.navigate` 并传入 `{"url":""}` 返回内置脚手架。

<div id="canvas-eval-snapshot">
  ### 画布评估/快照
</div>

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.eval --params '{"javaScript":"(() => { const {ctx} = window.__openclaw; ctx.clearRect(0,0,innerWidth,innerHeight); ctx.lineWidth=6; ctx.strokeStyle=\"#ff2d55\"; ctx.beginPath(); ctx.moveTo(40,40); ctx.lineTo(innerWidth-40, innerHeight-40); ctx.stroke(); return \"ok\"; })()"}'
```

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.snapshot --params '{"maxWidth":900,"format":"jpeg"}'
```

<div id="voice-wake-talk-mode">
  ## 语音唤醒 + 对话模式
</div>

* 语音唤醒和对话模式可在「设置」（Settings）中进行配置。
* iOS 可能会暂停后台音频；当应用不在前台时，语音功能只能视为尽力提供，可靠性无法保证。

<div id="common-errors">
  ## 常见错误
</div>

* `NODE_BACKGROUND_UNAVAILABLE`：将 iOS 应用切换到前台（canvas/camera/screen 命令需要在前台运行）。
* `A2UI_HOST_NOT_CONFIGURED`：Gateway 未提供 canvas 主机 URL；检查 [Gateway 配置](/zh/gateway/configuration) 中的 `canvasHost`。
* 从未出现配对提示：运行 `openclaw nodes pending` 并手动批准。
* 重新安装后重新连接失败：钥匙串中的配对令牌已被清除；请重新为该节点进行配对。

<div id="related-docs">
  ## 相关文档
</div>

* [配对](/zh/gateway/pairing)
* [发现](/zh/gateway/discovery)
* [Bonjour](/zh/gateway/bonjour)