---
title: Ansible
summary: "使用 Ansible、Tailscale VPN 和防火墙隔离，实现自动化且安全加固的 OpenClaw 安装"
read_when:
  - 你希望以自动化方式部署服务器并进行安全加固
  - 你需要通过 VPN 访问、由防火墙隔离的环境
  - 你正在将其部署到远程 Debian/Ubuntu 服务器上
---

<div id="ansible-installation">
  # 使用 Ansible 安装
</div>

将 OpenClaw 部署到生产服务器的推荐方式是使用 **[openclaw-ansible](https://github.com/openclaw/openclaw-ansible)** —— 一款采用安全优先架构的自动化安装工具。

<div id="quick-start">
  ## 快速开始
</div>

一条命令安装：

```bash
curl -fsSL https://raw.githubusercontent.com/openclaw/openclaw-ansible/main/install.sh | bash
```

> **📦 完整指南：[github.com/openclaw/openclaw-ansible](https://github.com/openclaw/openclaw-ansible)**
>
> `openclaw-ansible` 仓库是 Ansible 部署的权威信息来源。本页只是一个快速概览。

<div id="what-you-get">
  ## 你将获得什么
</div>

* 🔒 **防火墙优先的安全性**：UFW + Docker 隔离（仅开放 SSH 和 Tailscale 访问）
* 🔐 **Tailscale VPN**：在不公开暴露服务的情况下实现安全远程访问
* 🐳 **Docker**：隔离的沙箱容器，仅绑定到 localhost
* 🛡️ **纵深防御**：四层安全架构
* 🚀 **一条命令完成部署**：几分钟内完成整体部署
* 🔧 **Systemd 集成**：随系统启动自动拉起并启用安全加固

<div id="requirements">
  ## 前置条件
</div>

* **OS**：Debian 11+ 或 Ubuntu 20.04+
* **访问权限**：root 或 sudo 权限
* **网络**：可访问互联网用于安装软件包
* **Ansible**：2.14+（由快速入门脚本自动安装）

<div id="what-gets-installed">
  ## 会安装哪些组件
</div>

Ansible playbook 将安装并配置：

1. **Tailscale**（用于安全远程访问的网状 VPN）
2. **UFW 防火墙**（仅允许 SSH 和 Tailscale 端口）
3. **Docker CE + Compose V2**（用于智能体沙箱）
4. **Node.js 22.x + pnpm**（运行时依赖）
5. **OpenClaw**（作为宿主机进程运行，而非容器化）
6. **systemd 服务**（随系统自动启动并进行安全加固）

注意：Gateway **直接在宿主机上运行**（不在 Docker 中），但智能体沙箱使用 Docker 进行隔离。详见 [Sandboxing](/zh/gateway/sandboxing)。

<div id="post-install-setup">
  ## 安装后的设置
</div>

安装完成后，切换为 openclaw 用户：

```bash
sudo -i -u openclaw
```

安装后脚本会引导你完成：

1. **入门向导**：配置 OpenClaw 设置
2. **提供方登录**：连接 WhatsApp/Telegram/Discord/Signal
3. **Gateway 测试**：验证安装
4. **Tailscale 设置**：连接到你的 VPN 网状网络

<div id="quick-commands">
  ### 快捷命令
</div>

```bash
# 检查服务状态
sudo systemctl status openclaw

# 查看实时日志
sudo journalctl -u openclaw -f

# 重启 Gateway
sudo systemctl restart openclaw

# 提供方登录(以 openclaw 用户运行)
sudo -i -u openclaw
openclaw channels login
```

<div id="security-architecture">
  ## 安全架构
</div>

<div id="4-layer-defense">
  ### 4 层防御
</div>

1. **防火墙（UFW）**：仅对公网开放 SSH（22）和 Tailscale（41641/UDP）端口
2. **VPN（Tailscale）**：Gateway 只能通过 VPN 网状网络访问
3. **Docker 隔离**：DOCKER-USER iptables 链阻止端口对外暴露
4. **Systemd 加固**：启用 NoNewPrivileges、PrivateTmp，并以非特权用户运行

<div id="verification">
  ### 验证
</div>

测试外部攻击面：

```bash
nmap -p- YOUR_SERVER_IP
```

应该**只显示 22 端口**（SSH）是开放的。所有其他服务（Gateway、Docker）都应被关闭并锁定。

<div id="docker-availability">
  ### Docker 可用性
</div>

Docker 是为 **Agent 代理沙箱**（隔离的工具执行环境）而安装的，而不是用于运行 Gateway 本身。Gateway 只绑定到 localhost，并通过 Tailscale VPN 访问。

有关沙箱配置，请参阅 [多智能体沙箱与工具](/zh/multi-agent-sandbox-tools)。

<div id="manual-installation">
  ## 手动安装
</div>

如果你更希望自己手动控制而不是使用自动化：

```bash
# 1. Install prerequisites
sudo apt update && sudo apt install -y ansible git

# 2. Clone repository
git clone https://github.com/openclaw/openclaw-ansible.git
cd openclaw-ansible

# 3. Install Ansible collections
ansible-galaxy collection install -r requirements.yml

# 4. Run playbook
./run-playbook.sh

# 或者直接运行(然后手动执行 /tmp/openclaw-setup.sh)
# ansible-playbook playbook.yml --ask-become-pass
```

<div id="updating-openclaw">
  ## 更新 OpenClaw
</div>

Ansible 安装器会将 OpenClaw 配置为手动更新模式。标准更新流程请参阅 [更新](/zh/install/updating)。

要重新执行 Ansible playbook（例如修改配置）：

```bash
cd openclaw-ansible
./run-playbook.sh
```

注意：该操作具有幂等性，可以安全地重复执行。

<div id="troubleshooting">
  ## 故障排查
</div>

<div id="firewall-blocks-my-connection">
  ### 防火墙阻止了我的连接
</div>

如果你被防火墙“挡在门外”：

* 先确认你可以通过 Tailscale VPN 访问
* SSH 访问（22 端口）始终被允许
* 按照设计，Gateway **只** 能通过 Tailscale 访问

<div id="service-wont-start">
  ### 服务无法启动
</div>

```bash
# Check logs
sudo journalctl -u openclaw -n 100

# Verify permissions
sudo ls -la /opt/openclaw

# 测试手动启动
sudo -i -u openclaw
cd ~/openclaw
pnpm start
```

<div id="docker-sandbox-issues">
  ### Docker 沙箱相关问题
</div>

```bash
# 验证 Docker 是否正在运行
sudo systemctl status docker

# 检查沙箱镜像
sudo docker images | grep openclaw-sandbox

# 如缺失则构建沙箱镜像
cd /opt/openclaw/openclaw
sudo -u openclaw ./scripts/sandbox-setup.sh
```

<div id="provider-login-fails">
  ### 提供方登录失败
</div>

请确保你正在以 `openclaw` 用户身份运行：

```bash
sudo -i -u openclaw
openclaw channels login
```

<div id="advanced-configuration">
  ## 高级配置
</div>

关于安全架构和故障排查的详细说明，请参阅：

* [安全架构](https://github.com/openclaw/openclaw-ansible/blob/main/docs/security.md)
* [技术细节](https://github.com/openclaw/openclaw-ansible/blob/main/docs/architecture.md)
* [故障排查指南](https://github.com/openclaw/openclaw-ansible/blob/main/docs/troubleshooting.md)

<div id="related">
  ## 相关内容
</div>

* [openclaw-ansible](https://github.com/openclaw/openclaw-ansible) — 完整部署指南
* [Docker](/zh/install/docker) — 基于容器的 Gateway 部署
* [Sandboxing](/zh/gateway/sandboxing) — 智能体沙箱配置
* [Multi-Agent Sandbox &amp; Tools](/zh/multi-agent-sandbox-tools) — 为每个智能体提供隔离