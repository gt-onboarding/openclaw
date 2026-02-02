---
title: 远程 Gateway 使用说明
summary: "为 OpenClaw.app 配置 SSH 隧道以连接远程 Gateway"
read_when: "当通过 SSH 将 macOS 应用连接到远程 Gateway 时"
---

<div id="running-openclawapp-with-a-remote-gateway">
  # 使用远程 Gateway 运行 OpenClaw.app
</div>

OpenClaw.app 通过 SSH 隧道连接到远程 Gateway。本指南将指导你完成相关配置。

<div id="overview">
  ## 概览
</div>

```
┌─────────────────────────────────────────────────────────────┐
│                        客户端机器                          │
│                                                              │
│  OpenClaw.app ──► ws://127.0.0.1:18789 (本地端口)           │
│                     │                                        │
│                     ▼                                        │
│  SSH Tunnel ────────────────────────────────────────────────│
│                     │                                        │
└─────────────────────┼──────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│                         远程机器                        │
│                                                              │
│  Gateway WebSocket ──► ws://127.0.0.1:18789 ──►              │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```


<div id="quick-setup">
  ## 快速开始
</div>

<div id="step-1-add-ssh-config">
  ### 步骤 1：添加 SSH 配置
</div>

编辑 `~/.ssh/config` 文件并添加：

```ssh
Host remote-gateway
    HostName <REMOTE_IP>          # 例如:172.27.187.184
    User <REMOTE_USER>            # 例如:jefferson
    LocalForward 18789 127.0.0.1:18789
    IdentityFile ~/.ssh/id_rsa
```

将 `<REMOTE_IP>` 和 `<REMOTE_USER>` 替换为你自己的实际值。


<div id="step-2-copy-ssh-key">
  ### 步骤 2：复制 SSH 密钥
</div>

将你的公钥复制到远程主机（需要输入一次密码）：

```bash
ssh-copy-id -i ~/.ssh/id_rsa <REMOTE_USER>@<REMOTE_IP>
```


<div id="step-3-set-gateway-token">
  ### 步骤 3：设置 Gateway Token
</div>

```bash
launchctl setenv OPENCLAW_GATEWAY_TOKEN "<your-token>"
```


<div id="step-4-start-ssh-tunnel">
  ### 步骤 4：启动 SSH 隧道
</div>

```bash
ssh -N remote-gateway &
```


<div id="step-5-restart-openclawapp">
  ### 第 5 步：重启 OpenClaw.app
</div>

```bash
# 退出 OpenClaw.app (⌘Q),然后重新打开:
open /path/to/OpenClaw.app
```

应用现在将通过 SSH 隧道连接到远程 Gateway。

***


<div id="auto-start-tunnel-on-login">
  ## 登录时自动启动隧道
</div>

要在你登录时自动启动 SSH 隧道，请创建一个 Launch Agent。

<div id="create-the-plist-file">
  ### 创建 PLIST 文件
</div>

将其保存为 `~/Library/LaunchAgents/bot.molt.ssh-tunnel.plist`：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>bot.molt.ssh-tunnel</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/bin/ssh</string>
        <string>-N</string>
        <string>remote-gateway</string>
    </array>
    <key>KeepAlive</key>
    <true/>
    <key>RunAtLoad</key>
    <true/>
</dict>
</plist>
```


<div id="load-the-launch-agent">
  ### 加载 Launch Agent
</div>

```bash
launchctl bootstrap gui/$UID ~/Library/LaunchAgents/bot.molt.ssh-tunnel.plist
```

该隧道现在将会：

* 在你登录时自动启动
* 崩溃时自动重启
* 持续在后台运行

旧版本说明：如果系统中仍存在残留的 `com.openclaw.ssh-tunnel` LaunchAgent，请将其删除。

***


<div id="troubleshooting">
  ## 故障排查
</div>

**检查隧道是否在运行：**

```bash
ps aux | grep "ssh -N remote-gateway" | grep -v grep
lsof -i :18789
```

**重启隧道：**

```bash
launchctl kickstart -k gui/$UID/bot.molt.ssh-tunnel
```

**停止隧道连接：**

```bash
launchctl bootout gui/$UID/bot.molt.ssh-tunnel
```

***


<div id="how-it-works">
  ## 工作原理
</div>

| Component | 功能说明 |
|-----------|---------|
| `LocalForward 18789 127.0.0.1:18789` | 将本地 18789 端口的流量转发到远程 18789 端口 |
| `ssh -N` | 以不执行远程命令的方式运行 SSH（仅进行端口转发） |
| `KeepAlive` | 如果隧道崩溃，会自动重启隧道 |
| `RunAtLoad` | 在 Agent 代理加载时自动启动隧道 |

OpenClaw.app 会在你的客户端机器上连接到 `ws://127.0.0.1:18789`。SSH 隧道会将该连接转发到运行 Gateway 的远程机器上的 18789 端口。