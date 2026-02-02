---
title: Linux
summary: "Linux 支持与配套应用状态"
read_when:
  - 了解 Linux 配套应用当前状态
  - 规划平台支持范围或贡献方向
---

<div id="linux-app">
  # Linux 应用
</div>

Gateway 在 Linux 上提供完整支持。**推荐使用 Node 作为运行时**。
不推荐在 Gateway 上使用 Bun（会导致 WhatsApp/Telegram 相关问题）。

计划提供原生 Linux 配套应用。如果你想参与开发，欢迎贡献。

<div id="beginner-quick-path-vps">
  ## 初学者快速上手路径（VPS）
</div>

1. 安装 Node 22+
2. `npm i -g openclaw@latest`
3. `openclaw onboard --install-daemon`
4. 在你的笔记本电脑上执行：`ssh -N -L 18789:127.0.0.1:18789 <user>@<host>`
5. 打开 `http://127.0.0.1:18789/`，然后粘贴你的令牌（token）

VPS 分步详细指南：[exe.dev](/zh/platforms/exe-dev)

<div id="install">
  ## 安装
</div>

* [快速开始](/zh/start/getting-started)
* [安装与更新](/zh/install/updating)
* 可选安装方案：[Bun（实验性）](/zh/install/bun)、[Nix](/zh/install/nix)、[Docker](/zh/install/docker)

<div id="gateway">
  ## Gateway
</div>

* [Gateway 运行手册](/zh/gateway)
* [配置](/zh/gateway/configuration)

<div id="gateway-service-install-cli">
  ## Gateway 服务安装（CLI）
</div>

通过以下任一方式：

```
openclaw onboard --install-daemon
```

或者：

```
openclaw gateway install
```

或者：

```
openclaw configure
```

在出现提示时选择 **Gateway service**。

修复/迁移：

```
openclaw doctor
```

<div id="system-control-systemd-user-unit">
  ## 系统控制（systemd 用户单元）
</div>

OpenClaw 默认会安装一个 systemd **用户**服务。对于共享或始终在线的服务器，请使用 **系统**服务。完整的单元示例和指导见 [Gateway 运行手册](/zh/gateway)。

最小配置：

创建 `~/.config/systemd/user/openclaw-gateway[-<profile>].service`：

```
[Unit]
Description=OpenClaw Gateway (profile: <profile>, v<version>)
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/usr/local/bin/openclaw gateway --port 18789
Restart=always
RestartSec=5

[Install]
WantedBy=default.target
```

启用该单元：

```
systemctl --user enable --now openclaw-gateway[-<profile>].service
```
