---
title: 浏览器
summary: "`openclaw browser` 的 CLI 参考（配置文件、标签页、操作、扩展中继）"
read_when:
  - 你在使用 `openclaw browser`，并且希望查看常见任务的示例
  - 你想通过节点主机远程控制运行在另一台机器上的浏览器
  - 你想使用 Chrome 扩展中继功能（通过工具栏按钮进行连接/断开）
---

<div id="openclaw-browser">
  # `openclaw browser`
</div>

管理 OpenClaw 的浏览器控制服务，并执行浏览器操作（标签页、快照、截图、导航、点击、输入）。

相关内容：

* 浏览器工具与 API：[Browser tool](/zh/tools/browser)
* Chrome 扩展中继服务：[Chrome extension](/zh/tools/chrome-extension)

<div id="common-flags">
  ## 通用参数
</div>

* `--url <gatewayWsUrl>`: Gateway WebSocket URL（默认使用配置中的值）。
* `--token <token>`: Gateway 令牌（如有需要）。
* `--timeout <ms>`: 请求超时时间（毫秒）。
* `--browser-profile <name>`: 选择浏览器配置文件（默认从配置中读取）。
* `--json`: 机器可读输出（在支持该格式的情况下）。

<div id="quick-start-local">
  ## 快速开始（本地）
</div>

```bash
openclaw browser --browser-profile chrome tabs
openclaw browser --browser-profile openclaw start
openclaw browser --browser-profile openclaw open https://example.com
openclaw browser --browser-profile openclaw snapshot
```

<div id="profiles">
  ## 配置文件
</div>

配置文件是具名的浏览器路由配置。在实际使用中：

* `openclaw`：启动或连接到由 OpenClaw 管理的专用 Chrome 实例（独立的用户数据目录）。
* `chrome`：通过 Chrome 扩展中继控制你现有的 Chrome 标签页。

```bash
openclaw browser profiles
openclaw browser create-profile --name work --color "#FF5A36"
openclaw browser delete-profile --name work
```

使用特定配置文件：

```bash
openclaw browser --browser-profile work tabs
```

<div id="tabs">
  ## 标签页
</div>

```bash
openclaw browser tabs
openclaw browser open https://docs.openclaw.ai
openclaw browser focus <targetId>
openclaw browser close <targetId>
```

<div id="snapshot-screenshot-actions">
  ## 快照 / 截图 / 操作
</div>

快照：

```bash
openclaw browser snapshot
```

截图：

```bash
openclaw browser screenshot
```

导航/点击/输入（基于 ref 的 UI 自动化）：

```bash
openclaw browser navigate https://example.com
openclaw browser click <ref>
openclaw browser type <ref> "hello"
```

<div id="chrome-extension-relay-attach-via-toolbar-button">
  ## Chrome 扩展中继（通过工具栏按钮附加）
</div>

此模式允许智能体控制你手动附加的现有 Chrome 标签页（不会自动附加）。

将未打包的扩展程序安装到一个固定路径：

```bash
openclaw browser extension install
openclaw browser extension path
```

然后在 Chrome 中打开 `chrome://extensions` → 启用「开发者模式」→ 点击「加载已解压的扩展程序」→ 选择刚才生成的文件夹。

完整指南：[Chrome 扩展](/zh/tools/chrome-extension)

<div id="remote-browser-control-node-host-proxy">
  ## 远程浏览器控制（节点主机代理）
</div>

如果 Gateway 运行在和浏览器不同的机器上，请在安装了 Chrome/Brave/Edge/Chromium 的那台机器上运行一个 **节点主机（node host）**。Gateway 会将浏览器操作代理到该节点（不需要单独的浏览器控制服务器）。

使用 `gateway.nodes.browser.mode` 控制自动路由，并使用 `gateway.nodes.browser.node` 在连接了多个节点时固定到某个特定节点上。

安全与远程配置：[Browser 工具](/zh/tools/browser)、[远程访问](/zh/gateway/remote)、[Tailscale](/zh/gateway/tailscale)、[安全](/zh/gateway/security)