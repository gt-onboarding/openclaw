---
title: 浏览器
summary: "集成浏览器控制服务 + 操作指令"
read_when:
  - 添加由智能体控制的浏览器自动化功能
  - 调试为什么 openclaw 会干扰你自己的 Chrome 浏览器
  - 在 macOS 应用中实现浏览器设置和生命周期管理逻辑
---

<div id="browser-openclaw-managed">
  # 浏览器（由 openclaw 管理）
</div>

OpenClaw 可以运行一个由智能体控制的**专用 Chrome/Brave/Edge/Chromium 用户配置**。
它与您的个人浏览器完全隔离，并通过 Gateway 内部的一个本地轻量级
控制服务进行管理（仅通过本机回环接口访问）。

入门视角：

* 可以把它看作一个**独立的、仅供智能体使用的浏览器**。
* `openclaw` 配置**不会**影响你的个人浏览器配置文件。
* 智能体可以在一个安全隔离的通道中**打开标签页、浏览页面、点击和输入**。
* 默认的 `chrome` 配置会通过扩展中继使用**系统默认的 Chromium 浏览器**；
  切换到 `openclaw` 则会使用隔离托管的浏览器。

<div id="what-you-get">
  ## 你将获得：
</div>

* 一个名为 **openclaw** 的独立浏览器配置文件（默认橙色强调色）。
* 确定性的标签页控制（列出/打开/聚焦/关闭）。
* Agent 代理操作（点击/输入/拖拽/选择）、快照、截图、PDF。
* 可选的多配置文件支持（`openclaw`、`work`、`remote` 等）。

这个浏览器**不是**你日常使用的浏览器。它是一个用于智能体自动化与验证的安全、隔离的环境。

<div id="quick-start">
  ## 快速开始
</div>

```bash
openclaw browser --browser-profile openclaw status
openclaw browser --browser-profile openclaw start
openclaw browser --browser-profile openclaw open https://example.com
openclaw browser --browser-profile openclaw snapshot
```

如果出现 “Browser disabled” 提示，请在配置文件中启用浏览器功能（见下文），然后重启 Gateway。

<div id="profiles-openclaw-vs-chrome">
  ## 配置集：`openclaw` 与 `chrome`
</div>

* `openclaw`：由 OpenClaw 托管的隔离浏览器（不需要扩展程序）。
* `chrome`：通过扩展程序将请求中继到你的**系统浏览器**（需要在某个标签页中附加 OpenClaw 扩展程序）。

如果你希望默认使用托管模式，请将 `browser.defaultProfile: "openclaw"` 设为该值。

<div id="configuration">
  ## 配置
</div>

浏览器配置保存在 `~/.openclaw/openclaw.json` 中。

```json5
{
  browser: {
    enabled: true,                    // default: true
    // cdpUrl: "http://127.0.0.1:18792", // legacy single-profile override
    remoteCdpTimeoutMs: 1500,         // remote CDP HTTP timeout (ms)
    remoteCdpHandshakeTimeoutMs: 3000, // 远程 CDP WebSocket 握手超时(毫秒)
    defaultProfile: "chrome",
    color: "#FF4500",
    headless: false,
    noSandbox: false,
    attachOnly: false,
    executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    profiles: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work: { cdpPort: 18801, color: "#0066CC" },
      remote: { cdpUrl: "http://10.0.0.42:9222", color: "#00AA00" }
    }
  }
}
```

注意：

* 浏览器控制服务会绑定到基于 `gateway.port` 推导出的回环地址端口
  （默认：`18791`，即 gateway 端口 + 2）。中继使用下一个端口（`18792`）。
* 如果你覆盖了 Gateway 端口（通过 `gateway.port` 或 `OPENCLAW_GATEWAY_PORT`），
  推导出的浏览器端口也会一并偏移，以保持在同一“端口族”中。
* `cdpUrl` 在未设置时默认指向中继端口。
* `remoteCdpTimeoutMs` 适用于远程（非回环）CDP 可达性检查。
* `remoteCdpHandshakeTimeoutMs` 适用于远程 CDP WebSocket 可达性检查。
* `attachOnly: true` 表示“从不启动本地浏览器；仅在浏览器已运行时附加到它。”
* 顶层 `color` 配合各个 profile 各自的 `color` 会给浏览器 UI 着色，以便你分辨当前激活的是哪个 profile。
* 默认 profile 是 `chrome`（扩展中继）。如需使用受管浏览器，请设置 `defaultProfile: "openclaw"`。
* 自动检测顺序：如果系统默认浏览器是基于 Chromium，则优先使用；否则依次是 Chrome → Brave → Edge → Chromium → Chrome Canary。
* 本地 `openclaw` profile 会自动分配 `cdpPort`/`cdpUrl`——只在远程 CDP 场景下才需要显式设置这些值。

<div id="use-brave-or-another-chromium-based-browser">
  ## 使用 Brave（或其他基于 Chromium 的浏览器）
</div>

如果你的**系统默认**浏览器是基于 Chromium 的（如 Chrome/Brave/Edge 等），
OpenClaw 会自动使用它。你可以通过设置 `browser.executablePath` 来覆盖
自动检测：

CLI 示例：

```bash
openclaw config set browser.executablePath "/usr/bin/google-chrome"
```

```json5
// macOS
{
  browser: {
    executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser"
  }
}

// Windows
{
  browser: {
    executablePath: "C:\\Program Files\\BraveSoftware\\Brave-Browser\\Application\\brave.exe"
  }
}

// Linux
{
  browser: {
    executablePath: "/usr/bin/brave-browser"
  }
}
```

<div id="local-vs-remote-control">
  ## 本地控制 vs 远程控制
</div>

* **本地控制（默认）：** Gateway 启动本地回环控制服务，并且可以启动本地浏览器。
* **远程控制（节点主机）：** 在有浏览器的机器上运行节点主机；Gateway 会将浏览器操作代理到该主机。
* **远程 CDP：** 设置 `browser.profiles.<name>.cdpUrl`（或 `browser.cdpUrl`）以
  附加到远程的 Chromium 内核浏览器。在这种情况下，OpenClaw 不会启动本地浏览器。

远程 CDP URL 可以包含认证信息：

* 查询参数中的令牌（例如：`https://provider.example?token=<token>`）
* HTTP Basic 认证（例如：`https://user:pass@provider.example`）

OpenClaw 在调用 `/json/*` 端点以及连接到 CDP WebSocket 时会保留这些认证信息。相比于把令牌写入配置文件，推荐使用环境变量或机密管理器来存放令牌。

<div id="node-browser-proxy-zero-config-default">
  ## 节点浏览器代理（零配置默认）
</div>

如果你在装有浏览器的机器上运行一个**节点主机**，OpenClaw 可以
自动将浏览器工具调用路由到该节点，而无需任何额外的浏览器配置。
这是远程 Gateway 的默认方式。

注意：

* 节点主机通过一个**代理命令**对外暴露其本地浏览器控制服务器。
* Profile 来源于节点自身的 `browser.profiles` 配置（与本地相同）。
* 如果你不想使用，可以禁用：
  * 在节点上：`nodeHost.browserProxy.enabled=false`
  * 在 Gateway 上：`gateway.nodes.browser.mode="off"`

<div id="browserless-hosted-remote-cdp">
  ## Browserless（托管远程 CDP）
</div>

[Browserless](https://browserless.io) 是一个托管的 Chromium 服务，通过 HTTPS
提供 CDP 端点。你可以将 OpenClaw 浏览器配置文件指向某个 Browserless
区域端点，并使用你的 API 密钥进行身份验证。

示例：

```json5
{
  browser: {
    enabled: true,
    defaultProfile: "browserless",
    remoteCdpTimeoutMs: 2000,
    remoteCdpHandshakeTimeoutMs: 4000,
    profiles: {
      browserless: {
        cdpUrl: "https://production-sfo.browserless.io?token=<BROWSERLESS_API_KEY>",
        color: "#00AA00"
      }
    }
  }
}
```

注意：

* 将 `<BROWSERLESS_API_KEY>` 替换为你实际的 Browserless 令牌。
* 选择与你的 Browserless 账户所在区域相匹配的端点（参见其文档）。

<div id="security">
  ## 安全性
</div>

关键要点：

* 浏览器控制仅监听本机回环地址（localhost）；访问必须通过 Gateway 的身份验证或节点配对。
* 将 Gateway 和所有节点所在主机放在私有网络（如 Tailscale）中；避免直接暴露到公网。
* 将远程 CDP URL/令牌视为机密；优先使用环境变量或机密管理服务。

远程 CDP 使用建议：

* 尽可能使用 HTTPS 端点和短期令牌。
* 避免将长效令牌直接写入配置文件。

<div id="profiles-multi-browser">
  ## 配置文件（多浏览器）
</div>

OpenClaw 支持多个具名配置文件（路由配置）。配置文件可以是：

* **openclaw-managed**：独立的基于 Chromium 的浏览器实例，拥有自己的用户数据目录和 CDP 端口
* **remote**：显式指定的 CDP URL（在其他位置运行的基于 Chromium 的浏览器）
* **extension relay**：通过本地中继 + Chrome 扩展程序复用你现有的 Chrome 标签页

默认行为：

* 若不存在，将自动创建 `openclaw` 配置文件。
* `chrome` 配置文件是为 Chrome 扩展中继预置的（默认指向 `http://127.0.0.1:18792`）。
* 本地 CDP 端口默认从 **18800–18899** 范围内分配。
* 删除配置文件会将其本地数据目录移至系统回收站/废纸篓。

所有控制端点都支持 `?profile=&lt;name&gt;`；CLI 使用 `--browser-profile`。

<div id="chrome-extension-relay-use-your-existing-chrome">
  ## Chrome 扩展中继（使用你现有的 Chrome）
</div>

OpenClaw 也可以通过本地 CDP 中继 + 一个 Chrome 扩展来驱动**你现有的 Chrome 标签页**（不需要单独的 “openclaw” Chrome 实例）。

完整指南：[Chrome extension](/zh/tools/chrome-extension)

流程：

* Gateway 本地运行（同一台机器），或者在浏览器所在机器上运行一个节点主机。
* 本地**中继服务器**在回环地址上的 `cdpUrl` 监听（默认：`http://127.0.0.1:18792`）。
* 你在要接入的标签页上点击 **OpenClaw Browser Relay** 扩展图标进行连接（不会自动连接）。
* 智能体通过常规的 `browser` 工具控制该标签页，并选择正确的 profile。

如果 Gateway 运行在其他地方，请在浏览器所在机器上运行一个节点主机，以便 Gateway 可以通过该节点代理浏览器操作。

<div id="sandboxed-sessions">
  ### 沙箱化会话
</div>

如果智能体会话在沙箱中运行，`browser` 工具可能会默认使用 `target="sandbox"`（沙箱浏览器）。
Chrome 扩展中继接管需要控制宿主浏览器，因此你需要：

* 以非沙箱模式运行会话，或
* 设置 `agents.defaults.sandbox.browser.allowHostControl: true`，并在调用该工具时使用 `target="host"`。

<div id="setup">
  ### 设置
</div>

1. 加载扩展程序（开发模式/未打包）：

```bash
openclaw browser extension install
```

* Chrome → `chrome://extensions` → 启用“Developer mode”（开发者模式）
* 点击“Load unpacked”（加载已解压的扩展）→ 选择由 `openclaw browser extension path` 命令输出的目录
* 将该扩展固定到工具栏，然后在你想控制的标签页上点击它（图标徽标会显示为 `ON`）。

2. 使用：

* CLI：`openclaw browser --browser-profile chrome tabs`
* Agent 工具：`browser`，并设置 `profile="chrome"`

可选：如果你想使用不同的名称或中继端口，可以创建你自己的 profile：

```bash
openclaw browser create-profile \
  --name my-chrome \
  --driver extension \
  --cdp-url http://127.0.0.1:18792 \
  --color "#00AA00"
```

Notes:

* 此模式的大多数操作（截图/快照/操作）都依赖 Playwright-on-CDP。
* 再次点击扩展图标即可断开。

<div id="isolation-guarantees">
  ## 隔离保障
</div>

* **专用用户数据目录**：完全不会使用或污染你的个人浏览器配置文件。
* **专用端口**：避免使用 `9222`，以防与开发工作流发生冲突。
* **确定性的标签页控制**：通过 `targetId` 精确定位目标标签页，而不是依赖“最后一个标签页”。

<div id="browser-selection">
  ## 浏览器选择
</div>

在本地启动时，OpenClaw 会按以下优先顺序选择第一个可用的浏览器：

1. Chrome
2. Brave
3. Edge
4. Chromium
5. Chrome Canary

你可以通过设置 `browser.executablePath` 来覆盖该选择。

各平台行为：

* macOS：检查 `/Applications` 和 `~/Applications`。
* Linux：查找 `google-chrome`、`brave`、`microsoft-edge`、`chromium` 等可执行文件。
* Windows：检查常见的安装位置。

<div id="control-api-optional">
  ## 控制 API（可选）
</div>

仅用于本地集成场景时，Gateway 会暴露一个小型本地回环 HTTP API：

* 状态/启动/停止：`GET /`, `POST /start`, `POST /stop`
* 标签页：`GET /tabs`, `POST /tabs/open`, `POST /tabs/focus`, `DELETE /tabs/:targetId`
* 快照/截图：`GET /snapshot`, `POST /screenshot`
* 操作：`POST /navigate`, `POST /act`
* 钩子：`POST /hooks/file-chooser`, `POST /hooks/dialog`
* 下载：`POST /download`, `POST /wait/download`
* 调试：`GET /console`, `POST /pdf`
* 调试：`GET /errors`, `GET /requests`, `POST /trace/start`, `POST /trace/stop`, `POST /highlight`
* 网络：`POST /response/body`
* 状态：`GET /cookies`, `POST /cookies/set`, `POST /cookies/clear`
* 状态：`GET /storage/:kind`, `POST /storage/:kind/set`, `POST /storage/:kind/clear`
* 设置：`POST /set/offline`, `POST /set/headers`, `POST /set/credentials`, `POST /set/geolocation`, `POST /set/media`, `POST /set/timezone`, `POST /set/locale`, `POST /set/device`

所有端点都接受 `?profile=&lt;name&gt;`。

<div id="playwright-requirement">
  ### Playwright 要求
</div>

某些功能（navigate/act/AI snapshot/role snapshot、元素截图、PDF）需要
Playwright。如果未安装 Playwright，这些端点会返回明确的 501
错误。对于由 openclaw 管理的 Chrome，ARIA 快照和基础截图仍然可用。
而对于 Chrome 扩展 relay 驱动，ARIA 快照和截图则需要 Playwright。

如果你看到 `Playwright is not available in this gateway build`，请安装完整的
Playwright 包（而不是 `playwright-core`），并重启 Gateway，或者重新安装带浏览器支持的
OpenClaw。

<div id="how-it-works-internal">
  ## 工作原理（内部）
</div>

高层流程：

* 一个小型的**控制服务器**接收 HTTP 请求。
* 它通过 **CDP** 连接到基于 Chromium 的浏览器（Chrome/Brave/Edge/Chromium）。
* 对于高级操作（点击/输入/页面快照/PDF），它在 CDP 之上使用 **Playwright**。
* 如果未安装 Playwright，则只能执行不依赖 Playwright 的操作。

这种设计让智能体始终通过一个稳定、确定性的接口工作，同时允许你灵活切换本地/远程浏览器和浏览器配置文件（profile）。

<div id="cli-quick-reference">
  ## CLI 速查表
</div>

所有命令都接受 `--browser-profile <name>`，用于指定目标浏览器配置文件（profile）。
所有命令也都接受 `--json`，用于生成机器可读的输出（稳定的负载格式）。

基础：

* `openclaw browser status`
* `openclaw browser start`
* `openclaw browser stop`
* `openclaw browser tabs`
* `openclaw browser tab`
* `openclaw browser tab new`
* `openclaw browser tab select 2`
* `openclaw browser tab close 2`
* `openclaw browser open https://example.com`
* `openclaw browser focus abcd1234`
* `openclaw browser close abcd1234`

检查：

* `openclaw browser screenshot`
* `openclaw browser screenshot --full-page`
* `openclaw browser screenshot --ref 12`
* `openclaw browser screenshot --ref e12`
* `openclaw browser snapshot`
* `openclaw browser snapshot --format aria --limit 200`
* `openclaw browser snapshot --interactive --compact --depth 6`
* `openclaw browser snapshot --efficient`
* `openclaw browser snapshot --labels`
* `openclaw browser snapshot --selector "#main" --interactive`
* `openclaw browser snapshot --frame "iframe#main" --interactive`
* `openclaw browser console --level error`
* `openclaw browser errors --clear`
* `openclaw browser requests --filter api --clear`
* `openclaw browser pdf`
* `openclaw browser responsebody "**/api" --max-chars 5000`

操作：

* `openclaw browser navigate https://example.com`
* `openclaw browser resize 1280 720`
* `openclaw browser click 12 --double`
* `openclaw browser click e12 --double`
* `openclaw browser type 23 "hello" --submit`
* `openclaw browser press Enter`
* `openclaw browser hover 44`
* `openclaw browser scrollintoview e12`
* `openclaw browser drag 10 11`
* `openclaw browser select 9 OptionA OptionB`
* `openclaw browser download e12 /tmp/report.pdf`
* `openclaw browser waitfordownload /tmp/report.pdf`
* `openclaw browser upload /tmp/file.pdf`
* `openclaw browser fill --fields '[{"ref":"1","type":"text","value":"Ada"}]'`
* `openclaw browser dialog --accept`
* `openclaw browser wait --text "Done"`
* `openclaw browser wait "#main" --url "**/dash" --load networkidle --fn "window.ready===true"`
* `openclaw browser evaluate --fn '(el) => el.textContent' --ref 7`
* `openclaw browser highlight e12`
* `openclaw browser trace start`
* `openclaw browser trace stop`

状态：

* `openclaw browser cookies`
* `openclaw browser cookies set session abc123 --url "https://example.com"`
* `openclaw browser cookies clear`
* `openclaw browser storage local get`
* `openclaw browser storage local set theme dark`
* `openclaw browser storage session clear`
* `openclaw browser set offline on`
* `openclaw browser set headers --json '{"X-Debug":"1"}'`
* `openclaw browser set credentials user pass`
* `openclaw browser set credentials --clear`
* `openclaw browser set geo 37.7749 -122.4194 --origin "https://example.com"`
* `openclaw browser set geo --clear`
* `openclaw browser set media dark`
* `openclaw browser set timezone America/New_York`
* `openclaw browser set locale en-US`
* `openclaw browser set device "iPhone 14"`

备注：

* `upload` 和 `dialog` 是**预备（arming）**调用；需要在触发选择器/对话框的点击/按键之前先运行它们。
* `upload` 也可以通过 `--input-ref` 或 `--element` 直接设置文件输入。
* `snapshot`：
  * `--format ai`（在安装 Playwright 时为默认值）：返回带有数字引用的 AI 快照（`aria-ref="<n>"`）。
  * `--format aria`：返回可访问性树（无引用；仅用于检查）。
  * `--efficient`（或 `--mode efficient`）：紧凑角色快照预设（interactive + compact + depth + 较低的 maxChars）。
  * 配置默认值（仅工具/CLI）：将 `browser.snapshotDefaults.mode: "efficient"` 设为 `"efficient"`，这样当调用方未传入 mode 时会使用高效快照（参见 [Gateway configuration](/zh/gateway/configuration#browser-openclaw-managed-browser)）。
  * 角色快照选项（`--interactive`、`--compact`、`--depth`、`--selector`）会强制生成带有类似 `ref=e12` 的基于角色的快照。
  * `--frame "<iframe selector>"` 将角色快照限制在某个 iframe 内（与 `e12` 之类的角色引用配对使用）。
  * `--interactive` 输出一个扁平、易于选取的交互元素列表（最适合用于驱动操作）。
  * `--labels` 会添加一个仅视口区域的屏幕截图，并叠加引用标签（输出 `MEDIA:<path>`）。
* `click`/`type`/等操作需要来自 `snapshot` 的 `ref`（可以是数字 `12` 或角色引用 `e12`）。
  出于刻意设计，动作不支持 CSS 选择器。

<div id="snapshots-and-refs">
  ## 快照与引用（refs）
</div>

OpenClaw 支持两种“快照”样式：

* **AI 快照（数字引用）**：`openclaw browser snapshot`（默认；`--format ai`）
  * 输出：包含数字引用的文本快照。
  * 操作：`openclaw browser click 12`、`openclaw browser type 23 "hello"`。
  * 内部行为：通过 Playwright 的 `aria-ref` 来解析引用。

* **角色快照（类似 `e12` 的角色引用）**：`openclaw browser snapshot --interactive`（或 `--compact`、`--depth`、`--selector`、`--frame`）
  * 输出：基于角色的列表/树，其中包含 `[ref=e12]`（以及可选的 `[nth=1]`）。
  * 操作：`openclaw browser click e12`、`openclaw browser highlight e12`。
  * 内部行为：通过 `getByRole(...)` 解析引用（对于重复元素再配合 `nth()`）。
  * 添加 `--labels` 可输出一张带有叠加 `e12` 标签的视口截图。

引用行为：

* 引用在**不同页面导航之间不稳定**；如果某个操作失败，请重新运行 `snapshot` 并使用新的引用。
* 如果角色快照是通过 `--frame` 获取的，则角色引用会限定在该 iframe 中，直到下一次角色快照为止。

<div id="wait-power-ups">
  ## 高级等待能力
</div>

你可以等待的不只是时间或文本：

* 等待 URL（Playwright 支持通配符）：
  * `openclaw browser wait --url "**/dash"`
* 等待加载状态：
  * `openclaw browser wait --load networkidle`
* 等待 JavaScript 断言条件为真：
  * `openclaw browser wait --fn "window.ready===true"`
* 等待某个选择器变为可见：
  * `openclaw browser wait "#main"`

这些可以组合使用：

```bash
openclaw browser wait "#main" \
  --url "**/dash" \
  --load networkidle \
  --fn "window.ready===true" \
  --timeout-ms 15000
```

<div id="debug-workflows">
  ## 调试流程
</div>

当某个操作失败时（例如出现 “not visible”、“strict mode violation”、“covered” 等错误）：

1. `openclaw browser snapshot --interactive`
2. 使用 `click <ref>` / `type <ref>`（在交互模式下优先使用 role 引用）
3. 如果仍然失败：运行 `openclaw browser highlight <ref>` 查看 Playwright 实际定位到的元素
4. 如果页面行为异常：
   * `openclaw browser errors --clear`
   * `openclaw browser requests --filter api --clear`
5. 需要深入调试时：录制 trace：
   * `openclaw browser trace start`
   * 复现问题
   * `openclaw browser trace stop`（会打印 `TRACE:<path>`）

<div id="json-output">
  ## JSON 输出
</div>

`--json` 选项用于脚本化和结构化工具。

示例：

```bash
openclaw browser status --json
openclaw browser snapshot --interactive --json
openclaw browser requests --filter api --json
openclaw browser cookies --json
```

以 JSON 表示的角色快照包含 `refs`，以及一个小型的 `stats` 统计块（lines/chars/refs/interactive），便于工具评估负载的大小和密度。

<div id="state-and-environment-knobs">
  ## 状态与环境控制项
</div>

这些命令适用于这类“让站点表现得像 X”的工作流：

* Cookies：`cookies`、`cookies set`、`cookies clear`
* 存储：`storage local|session get|set|clear`
* 离线：`set offline on|off`
* 请求头：`set headers --json '{"X-Debug":"1"}'`（或 `--clear`）
* HTTP 基本认证：`set credentials user pass`（或 `--clear`）
* 地理位置：`set geo <lat> <lon> --origin "https://example.com"`（或 `--clear`）
* 媒体偏好：`set media dark|light|no-preference|none`
* 时区 / 语言与区域设置：`set timezone ...`，`set locale ...`
* 设备 / 视口：
  * `set device "iPhone 14"`（Playwright 设备预设）
  * `set viewport 1280 720`

<div id="security-privacy">
  ## 安全与隐私
</div>

* openclaw 浏览器 profile（配置文件）可能包含已登录的会话；请将其视为敏感数据。
* `browser act kind=evaluate` / `openclaw browser evaluate` 和 `wait --fn`
  会在页面上下文中执行任意 JavaScript，可能被提示注入所操控。
  如果不需要此功能，请通过 `browser.evaluateEnabled=false` 将其禁用。
* 有关登录和反机器人说明（X/Twitter 等），请参阅 [Browser login + X/Twitter posting](/zh/tools/browser-login)。
* 将运行 Gateway/节点 的主机保持为私有（仅回环地址或仅限 tailnet）。
* 远程 CDP 端点能力很强；请通过隧道访问并做好防护。

<div id="troubleshooting">
  ## 故障排查
</div>

对于 Linux 相关的问题（尤其是使用 snap 安装的 Chromium 浏览器），请参阅
[浏览器故障排查](/zh/tools/browser-linux-troubleshooting)。

<div id="agent-tools-how-control-works">
  ## Agent 代理工具与控制机制
</div>

该智能体仅获得用于浏览器自动化的**一个工具**：

* `browser` — status/start/stop/tabs/open/focus/close/snapshot/screenshot/navigate/act

其作用方式如下：

* `browser snapshot` 返回稳定的 UI 树（AI 或 ARIA）。
* `browser act` 使用快照中的 `ref` ID 来执行点击/输入/拖拽/选择等操作。
* `browser screenshot` 捕获像素图（整页或单个元素）。
* `browser` 接受以下参数：
  * `profile` 用于选择命名浏览器配置文件（openclaw、chrome 或远程 CDP）。
  * `target`（`sandbox` | `host` | `node`）用于选择浏览器所在的位置。
  * 在沙箱会话中，将 `target` 设为 `"host"` 需要 `agents.defaults.sandbox.browser.allowHostControl=true`。
  * 如果省略 `target`：沙箱会话默认使用 `sandbox`，非沙箱会话默认使用 `host`。
  * 如果已连接具备浏览器能力的节点，该工具可能会自动路由到该节点，除非你显式固定设置 `target="host"` 或 `target="node"`。

这样可以保持智能体行为的确定性，并避免依赖脆弱的选择器。