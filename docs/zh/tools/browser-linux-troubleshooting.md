---
title: 浏览器 Linux 故障排查
summary: "修复 OpenClaw 在 Linux 上进行浏览器控制时遇到的 Chrome/Brave/Edge/Chromium CDP 启动问题"
read_when: "在 Linux 上浏览器控制失败，特别是使用 snap 版 Chromium 时"
---

<div id="browser-troubleshooting-linux">
  # 浏览器故障排查（Linux）
</div>

<div id="problem-failed-to-start-chrome-cdp-on-port-18800">
  ## 问题：&quot;Failed to start Chrome CDP on port 18800&quot;
</div>

OpenClaw 的浏览器控制服务器在尝试启动 Chrome/Brave/Edge/Chromium 时失败，并报出以下错误：

```
{"error":"Error: Failed to start Chrome CDP on port 18800 for profile \"openclaw\"."}
```


<div id="root-cause">
  ### 根本原因
</div>

在 Ubuntu（以及许多其他 Linux 发行版）上，默认的 Chromium 安装是一个 **snap 包**。Snap 的 AppArmor 沙箱/限制机制会干扰 OpenClaw 启动和监控浏览器进程的方式。

命令 `apt install chromium` 实际安装的是一个会重定向到 snap 的占位包（stub package）：

```
Note, selecting 'chromium-browser' instead of 'chromium'
chromium-browser is already the newest version (2:1snap1-0ubuntu2).
```

这不是真正的浏览器——它只是一个外壳而已。


<div id="solution-1-install-google-chrome-recommended">
  ### 解决方案 1：安装 Google Chrome（推荐）
</div>

安装官方的 Google Chrome `.deb` 软件包，该软件包不使用 snap 的沙箱机制：

```bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb
sudo apt --fix-broken install -y  # 如果有依赖错误
```

然后更新你的 OpenClaw 配置文件（`~/.openclaw/openclaw.json`）：

```json
{
  "browser": {
    "enabled": true,
    "executablePath": "/usr/bin/google-chrome-stable",
    "headless": true,
    "noSandbox": true
  }
}
```


<div id="solution-2-use-snap-chromium-with-attach-only-mode">
  ### 解决方案 2：在仅附加模式下使用 Snap 版 Chromium
</div>

如果你必须使用 Snap 版 Chromium，请将 OpenClaw 配置为附加到你手动启动的浏览器：

1. 更新配置：

```json
{
  "browser": {
    "enabled": true,
    "attachOnly": true,
    "headless": true,
    "noSandbox": true
  }
}
```

2. 手动启动 Chromium 浏览器：

```bash
chromium-browser --headless --no-sandbox --disable-gpu \
  --remote-debugging-port=18800 \
  --user-data-dir=$HOME/.openclaw/browser/openclaw/user-data \
  about:blank &
```

3. （可选）创建一个 systemd 用户服务，用于自动启动 Chrome：

```ini
# ~/.config/systemd/user/openclaw-browser.service
[Unit]
Description=OpenClaw 浏览器 (Chrome CDP)
After=network.target

[Service]
ExecStart=/snap/bin/chromium --headless --no-sandbox --disable-gpu --remote-debugging-port=18800 --user-data-dir=%h/.openclaw/browser/openclaw/user-data about:blank
Restart=on-failure
RestartSec=5

[Install]
WantedBy=default.target
```

通过以下命令启用：`systemctl --user enable --now openclaw-browser.service`


<div id="verifying-the-browser-works">
  ### 确认浏览器是否正常运行
</div>

检查状态：

```bash
curl -s http://127.0.0.1:18791/ | jq '{running, pid, chosenBrowser}'
```

测试浏览功能：

```bash
curl -s -X POST http://127.0.0.1:18791/start
curl -s http://127.0.0.1:18791/tabs
```


<div id="config-reference">
  ### 配置参考
</div>

| 选项 | 描述 | 默认值 |
|--------|-------------|---------|
| `browser.enabled` | 启用浏览器控制 | `true` |
| `browser.executablePath` | 基于 Chromium 的浏览器可执行文件路径（Chrome/Brave/Edge/Chromium） | 自动检测（在基于 Chromium 时优先使用默认浏览器） |
| `browser.headless` | 以无界面模式运行 | `false` |
| `browser.noSandbox` | 添加 `--no-sandbox` 标志（某些 Linux 环境需要） | `false` |
| `browser.attachOnly` | 不启动浏览器，仅附加到已有实例 | `false` |
| `browser.cdpPort` | Chrome DevTools Protocol 端口 | `18800` |

<div id="problem-chrome-extension-relay-is-running-but-no-tab-is-connected">
  ### 问题：“Chrome extension relay is running, but no tab is connected”
</div>

你正在使用 `chrome` 配置（extension relay）。它需要将 OpenClaw
浏览器扩展附加到一个正在使用的标签页上。

解决方式：

1. **使用托管浏览器：** `openclaw browser start --browser-profile openclaw`
   （或设置 `browser.defaultProfile: "openclaw"`）。
2. **使用扩展中继：** 安装扩展，打开一个标签页，然后点击
   OpenClaw 扩展图标将其附加到该标签页。

注意：

- `chrome` 配置在可能的情况下会使用你的**系统默认 Chromium 浏览器**。
- 本地 `openclaw` 配置会自动分配 `cdpPort`/`cdpUrl`；只有在远程 CDP 场景下才需要手动设置这些值。