---
title: Exe Dev
summary: "在 exe.dev 上运行 OpenClaw Gateway（VM + HTTPS 代理）以实现远程访问"
read_when:
  - 你想要一个廉价的、常驻在线的 Linux 主机来运行 Gateway
  - 你希望远程访问 Control UI，但又不想自己搭建 VPS
---

<div id="exedev">
  # exe.dev
</div>

目标：让 OpenClaw Gateway 在一个 exe.dev 虚拟机上运行，并且可以从你的笔记本通过 `https://<vm-name>.exe.xyz` 访问。

本页假设你使用的是 exe.dev 提供的默认 **exeuntu** 镜像。如果你选择了其他发行版，请自行将相关软件包名称映射到该发行版中的等价包。

<div id="beginner-quick-path">
  ## 新手快速入门
</div>

1. [https://exe.new/openclaw](https://exe.new/openclaw)
2. 按需填写你的认证密钥/令牌
3. 点击虚拟机旁边的“Agent”，然后等待……
4. ???
5. 赚钱

<div id="what-you-need">
  ## 前提条件
</div>

* 一个 exe.dev 账号
* 使用 `ssh exe.dev` 访问 [exe.dev](https://exe.dev) 虚拟机的权限（可选）

<div id="automated-install-with-shelley">
  ## 使用 Shelley 自动安装
</div>

Shelley 是 [exe.dev](https://exe.dev) 的 Agent 代理，可以使用我们提供的提示词
立即安装 OpenClaw。所使用的提示词如下：

```
在此虚拟机上设置 OpenClaw (https://docs.openclaw.ai/install)。使用非交互式和接受风险标志进行 openclaw 入门配置。根据需要添加提供的身份验证或令牌。配置 nginx 将默认端口 18789 转发到默认启用站点配置的根位置,确保启用 WebSocket 支持。配对通过 "openclaw devices list" 和 "openclaw device approve <request id>" 完成。确保仪表板显示 OpenClaw 的健康状态为 OK。exe.dev 为我们处理从端口 8000 到端口 80/443 的转发以及 HTTPS,因此最终的 "可达" 地址应为 <vm-name>.exe.xyz,无需指定端口。
```

<div id="manual-installation">
  ## 手动安装
</div>

<div id="1-create-the-vm">
  ## 1) 创建虚拟机
</div>

从你的设备上：

```bash
ssh exe.dev new 
```

然后连接：

```bash
ssh <vm-name>.exe.xyz
```

提示：请让此 VM 保持为**有状态（stateful）**。OpenClaw 会在 `~/.openclaw/` 和 `~/.openclaw/workspace/` 下存储状态。

<div id="2-install-prerequisites-on-the-vm">
  ## 2) 在虚拟机上安装先决条件
</div>

```bash
sudo apt-get update
sudo apt-get install -y git curl jq ca-certificates openssl
```

<div id="3-install-openclaw">
  ## 3) 安装 OpenClaw
</div>

运行 OpenClaw 安装脚本：

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

<div id="4-setup-nginx-to-proxy-openclaw-to-port-8000">
  ## 4) 设置 nginx，将 OpenClaw 反向代理到 8000 端口
</div>

编辑 `/etc/nginx/sites-enabled/default`，修改为：

```
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    listen 8000;
    listen [::]:8000;

    server_name _;

    location / {
        proxy_pass http://127.0.0.1:18789;
        proxy_http_version 1.1;

        # WebSocket support
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        # Standard proxy headers
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # 长连接超时设置
        proxy_read_timeout 86400s;
        proxy_send_timeout 86400s;
    }
}
```

<div id="5-access-openclaw-and-grant-privileges">
  ## 5) 访问 OpenClaw 并授予权限
</div>

访问 `https://<vm-name>.exe.xyz/?token=YOUR-TOKEN-FROM-TERMINAL`。使用 `openclaw devices list` 和 `openclaw device approve` 来授权设备。如果不确定，可以直接在浏览器中使用 Shelley！

<div id="remote-access">
  ## 远程访问
</div>

远程访问由 [exe.dev](https://exe.dev) 的身份验证机制负责处理。默认情况下，来自 8000 端口的 HTTP 流量在通过电子邮件身份验证后会被转发到 `https://<vm-name>.exe.xyz`。

<div id="updating">
  ## 更新
</div>

```bash
npm i -g openclaw@latest
openclaw doctor
openclaw gateway restart
openclaw health
```

指南：[更新](/zh/install/updating)
