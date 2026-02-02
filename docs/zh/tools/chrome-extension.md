---
title: Chrome 扩展程序
summary: "Chrome 扩展程序：让 OpenClaw 驱动你当前的 Chrome 标签页"
read_when:
  - 你希望让智能体接管一个已打开的 Chrome 标签页（通过工具栏按钮）
  - 你需要通过 Tailscale 将远程 Gateway 与本地浏览器自动化联通
  - 你想了解让浏览器被接管的安全影响
---

<div id="chrome-extension-browser-relay">
  # Chrome 扩展程序（浏览器中继）
</div>

OpenClaw Chrome 扩展程序允许智能体控制你**现有的 Chrome 标签页**（你平时使用的 Chrome 窗口），而不是启动一个由 openclaw 管理的单独 Chrome 配置文件。

连接/断开是通过一个**Chrome 工具栏上的按钮**完成的。

<div id="what-it-is-concept">
  ## 它是什么（概念）
</div>

它由三个部分组成：

* **浏览器控制服务**（Gateway 或节点）：智能体/工具 通过 Gateway 调用的 API
* **本地中继服务器**（loopback CDP）：在控制服务器与扩展之间建立桥接（默认地址为 `http://127.0.0.1:18792`）
* **Chrome MV3 扩展**：使用 `chrome.debugger` 附加到活动标签页，并将 CDP 消息转发到中继

然后，OpenClaw 通过常规的 `browser` 工具接口（选择正确的配置文件）来控制已附加的标签页。

<div id="install-load-unpacked">
  ## 安装 / 加载（未打包）
</div>

1. 将扩展安装到一个固定的本地路径：

```bash
openclaw browser extension install
```

2. 输出已安装扩展的目录路径：

```bash
openclaw browser extension path
```

3. Chrome → `chrome://extensions`

* 启用“开发者模式”
* 点击“加载已解压的扩展程序” → 选择上面输出的目录

4. 将扩展程序固定在工具栏上。

<div id="updates-no-build-step">
  ## 更新（无需构建步骤）
</div>

该扩展作为静态文件，随 OpenClaw 的发布版本（npm 包）一起提供，无需单独的“构建”步骤。

升级 OpenClaw 之后：

* 重新运行 `openclaw browser extension install`，以刷新 OpenClaw 状态目录下已安装的文件。
* Chrome → `chrome://extensions` → 在该扩展上点击 “Reload”（重新加载）。

<div id="use-it-no-extra-config">
  ## 使用方式（无需额外配置）
</div>

OpenClaw 默认内置了一个名为 `chrome` 的浏览器配置文件，它通过默认端口连接到扩展中继。

使用方式：

* CLI：`openclaw browser --browser-profile chrome tabs`
* Agent 代理工具：`browser`，并设置 `profile="chrome"`

如果你想使用不同的名称或不同的中继端口，可以创建你自己的配置文件：

```bash
openclaw browser create-profile \
  --name my-chrome \
  --driver extension \
  --cdp-url http://127.0.0.1:18792 \
  --color "#00AA00"
```

<div id="attach-detach-toolbar-button">
  ## 绑定 / 解绑（工具栏按钮）
</div>

* 打开你希望由 OpenClaw 控制的标签页。
* 点击扩展图标。
  * 绑定后，徽章会显示 `ON`。
* 再次点击以解绑。

<div id="which-tab-does-it-control">
  ## 它会控制哪个标签页？
</div>

* 它**不会**自动控制“你当前正在查看的任何标签页”。
* 它**只会控制你通过点击工具栏按钮显式附加的标签页**。
* 若要切换：打开另一个标签页，并在该标签页中点击扩展图标。

<div id="badge-common-errors">
  ## 徽标状态 + 常见错误
</div>

* `ON`：已接入；OpenClaw 可以控制该标签页。
* `…`：正在连接到本地中继。
* `!`：无法访问中继（最常见原因：此机器上未运行浏览器中继服务器）。

如果你看到 `!`：

* 确保 Gateway 在本机运行（默认设置）；如果 Gateway 运行在其他地方，则在本机上运行一个节点主机。
* 打开扩展的 Options 页面；它会显示中继是否可访问。

<div id="remote-gateway-use-a-node-host">
  ## 远程 Gateway（使用节点所在主机）
</div>

<div id="local-gateway-same-machine-as-chrome-usually-no-extra-steps">
  ### 本地 Gateway（与 Chrome 在同一台机器上）——通常**无需额外步骤**
</div>

如果 Gateway 与 Chrome 运行在同一台机器上，它会在回环接口上启动浏览器控制服务，
并自动启动中继服务器。扩展程序会与本地中继通信；CLI/工具调用则会发往 Gateway。

<div id="remote-gateway-gateway-runs-elsewhere-run-a-node-host">
  ### 远程 Gateway（Gateway 运行在其他地方）— **运行一个节点主机**
</div>

如果你的 Gateway 在另一台机器上运行，请在运行 Chrome 的那台机器上启动一个节点主机。
Gateway 会将浏览器操作代理到该节点；扩展程序和中继会保留在本地这台浏览器机器上。

如果连接了多个节点，可通过 `gateway.nodes.browser.node` 固定其中一个，或者设置 `gateway.nodes.browser.mode`。

<div id="sandboxing-tool-containers">
  ## 沙箱（工具容器）
</div>

如果你的智能体会话运行在沙箱中（`agents.defaults.sandbox.mode != "off"`），`browser` 工具可能会受到限制：

* 默认情况下，沙箱化会话通常会连接到**沙箱浏览器**（`target="sandbox"`），而不是你的宿主机 Chrome。
* 通过 Chrome extension relay 进行接管，需要能够控制**宿主机**的浏览器控制服务器。

选项：

* 最简单的方式：在**非沙箱**会话/智能体中使用该扩展。
* 或者，为沙箱化会话开放对宿主机浏览器的控制权限：

```json5
{
  agents: {
    defaults: {
      sandbox: {
        browser: {
          allowHostControl: true
        }
      }
    }
  }
}
```

然后确保该工具未被工具策略禁止，并在需要时调用 `browser`，同时使用 `target="host"`。

调试：`openclaw sandbox explain`

<div id="remote-access-tips">
  ## 远程访问注意事项
</div>

* 让 Gateway 和节点主机保持在同一个 tailnet 中；避免将中继端口暴露给局域网或公网。
* 按需配对节点；如果你不希望启用远程控制，请禁用浏览器代理路由（`gateway.nodes.browser.mode="off"`）。

<div id="how-extension-path-works">
  ## “extension path” 的工作原理
</div>

`openclaw browser extension path` 会输出包含扩展文件的 **已安装** 磁盘目录路径。

CLI 刻意 **不会** 输出 `node_modules` 路径。请务必先运行 `openclaw browser extension install`，将扩展复制到 OpenClaw 状态目录下的稳定位置。

如果你移动或删除该安装目录，Chrome 会将此扩展标记为已损坏，直到你从一个有效路径重新加载它为止。

<div id="security-implications-read-this">
  ## 安全影响（必读）
</div>

这既强大又有风险。要把它当成是“把你的浏览器交到模型手里”来对待。

* 该扩展使用 Chrome 的调试器 API（`chrome.debugger`）。一旦附加到标签页，模型可以：
  * 在该标签页中点击 / 输入 / 导航
  * 读取页面内容
  * 访问该标签页中已登录会话所能访问的一切
* **这不像 openclaw 管理的专用配置文件那样被隔离**。
  * 如果你附加到日常使用的配置文件/标签页，你实际上是在授予对该账号状态的访问权限。

建议：

* 优先使用一个专用的 Chrome 配置文件（与个人日常浏览分离）来进行扩展中继使用。
* 让 Gateway 和所有节点主机仅限 tailnet 内访问；依赖 Gateway 身份验证 + 节点配对。
* 避免通过 LAN 暴露中继端口（`0.0.0.0`），并避免使用 Funnel（公网暴露）。

相关内容：

* 浏览器工具概览：[Browser](/zh/tools/browser)
* 安全审计：[Security](/zh/gateway/security)
* Tailscale 设置：[Tailscale](/zh/gateway/tailscale)