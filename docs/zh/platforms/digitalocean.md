---
title: Digitalocean
summary: "在 DigitalOcean 上部署 OpenClaw（简单的付费 VPS 选项）"
read_when:
  - 在 DigitalOcean 上设置 OpenClaw
  - 寻找用于部署 OpenClaw 的低成本 VPS 托管服务
---

<div id="openclaw-on-digitalocean">
  # 在 DigitalOcean 上运行 OpenClaw
</div>

<div id="goal">
  ## 目标
</div>

在 DigitalOcean 上以 **6 美元/月**（或预留定价为 4 美元/月）的成本持久运行一个 OpenClaw Gateway。

如果你想要免费（0 美元/月）的方案，并且不介意使用 ARM 架构和特定提供方的配置流程，请参阅 [Oracle Cloud 指南](/zh/platforms/oracle)。

<div id="cost-comparison-2026">
  ## 成本对比（2026）
</div>

| Provider | Plan | Specs | Price/mo | Notes |
|----------|------|-------|----------|-------|
| Oracle Cloud | Always Free ARM | up to 4 OCPU, 24GB RAM | $0 | ARM，容量有限 / 注册流程比较折腾 |
| Hetzner | CX22 | 2 vCPU, 4GB RAM | €3.79 (~$4) | 最便宜的付费方案 |
| DigitalOcean | Basic | 1 vCPU, 1GB RAM | $6 | UI 简单、文档完善 |
| Vultr | Cloud Compute | 1 vCPU, 1GB RAM | $6 | 可选地区很多 |
| Linode | Nanode | 1 vCPU, 1GB RAM | $5 | 现已并入 Akamai |

**如何选择提供方：**

* DigitalOcean：用户体验最简单，部署流程可预期（本指南使用）
* Hetzner：价格 / 性能比好（参见 [Hetzner 指南](/zh/platforms/hetzner)）
* Oracle Cloud：可以做到 $0/月，但更折腾且仅支持 ARM（参见 [Oracle 指南](/zh/platforms/oracle)）

***

<div id="prerequisites">
  ## 前提条件
</div>

* DigitalOcean 账户（[注册可获 200 美元免费额度](https://m.do.co/c/signup)）
* SSH 密钥对（或使用密码认证）
* 大约 20 分钟

<div id="1-create-a-droplet">
  ## 1) 创建 Droplet
</div>

1. 登录 [DigitalOcean](https://cloud.digitalocean.com/)
2. 点击 **Create → Droplets**
3. 选择：
   * **Region:** 距离你（或你的用户）最近的区域
   * **Image:** Ubuntu 24.04 LTS
   * **Size:** Basic → Regular → **$6/mo**（1 vCPU，1GB RAM，25GB SSD）
   * **Authentication:** SSH key（推荐）或 password（密码）
4. 点击 **Create Droplet**
5. 记下该 Droplet 的 IP 地址

<div id="2-connect-via-ssh">
  ## 2) 通过 SSH 连接
</div>

```bash
ssh root@YOUR_DROPLET_IP
```

<div id="3-install-openclaw">
  ## 3）安装 OpenClaw
</div>

```bash
# 更新系统
apt update && apt upgrade -y

# 安装 Node.js 22
curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
apt install -y nodejs

# 安装 OpenClaw
curl -fsSL https://openclaw.bot/install.sh | bash

# 验证安装
openclaw --version
```

<div id="4-run-onboarding">
  ## 4) 运行初始化向导
</div>

```bash
openclaw onboard --install-daemon
```

向导将引导你逐步完成以下配置：

* 模型认证（API 密钥或 OAuth）
* 渠道设置（Telegram、WhatsApp、Discord 等）
* Gateway 令牌（自动生成）
* 守护进程安装（systemd）

<div id="5-verify-the-gateway">
  ## 5) 检查 Gateway
</div>

```bash
# 检查状态
openclaw status

# 检查服务
systemctl --user status openclaw-gateway.service

# 查看日志
journalctl --user -u openclaw-gateway.service -f
```

<div id="6-access-the-dashboard">
  ## 6) 访问控制面板
</div>

Gateway 默认只监听本地回环地址。要访问 Control UI：

**选项 A：SSH 隧道（推荐）**

```bash
# 从本地机器执行
ssh -L 18789:localhost:18789 root@YOUR_DROPLET_IP

# 然后打开: http://localhost:18789
```

**选项 B：Tailscale Serve（HTTPS，仅限本地回环）**

```bash
# On the droplet
curl -fsSL https://tailscale.com/install.sh | sh
tailscale up

# Configure Gateway to use Tailscale Serve
openclaw config set gateway.tailscale.mode serve
openclaw gateway restart
```

打开：`https://<magicdns>/`

说明：

* Serve 会让 Gateway 仅在本机回环接口上监听，并通过 Tailscale 身份头进行认证。
* 若改为要求令牌/密码，可将 `gateway.auth.allowTailscale` 设为 `false`，或使用 `gateway.auth.mode: "password"`。

**选项 C：Tailnet 绑定（不使用 Serve）**

```bash
openclaw config set gateway.bind tailnet
openclaw gateway restart
```

访问：`http://<tailscale-ip>:18789`（需要令牌）。

<div id="7-connect-your-channels">
  ## 7）接入你的渠道
</div>

<div id="telegram">
  ### Telegram
</div>

```bash
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

<div id="whatsapp">
  ### WhatsApp
</div>

```bash
openclaw channels login whatsapp
# 扫描二维码
```

请参阅 [Channels](/zh/channels) 以了解其他提供方。

***

<div id="optimizations-for-1gb-ram">
  ## 针对 1GB 内存的优化
</div>

价格为 6 美元的 Droplet 只有 1GB 内存。为了让系统运行更顺畅：

<div id="add-swap-recommended">
  ### 添加交换空间（推荐）
</div>

```bash
fallocate -l 2G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap sw 0 0' >> /etc/fstab
```

<div id="use-a-lighter-model">
  ### 使用更轻量的模型
</div>

如果你遇到内存不足（OOM）问题，可以考虑：

* 改用基于 API 的模型（Claude、GPT），而不是本地模型
* 将 `agents.defaults.model.primary` 设置为更小的模型

<div id="monitor-memory">
  ### 内存监控
</div>

```bash
free -h
htop
```

***

<div id="persistence">
  ## 持久化
</div>

所有状态都存放在：

* `~/.openclaw/` — 配置、凭据、会话数据
* `~/.openclaw/workspace/` — 工作区（SOUL.md、memory 等）

这些目录在重启后仍会保留。请定期备份它们：

```bash
tar -czvf openclaw-backup.tar.gz ~/.openclaw ~/.openclaw/workspace
```

***

<div id="oracle-cloud-free-alternative">
  ## Oracle Cloud 免费替代方案
</div>

Oracle Cloud 提供 **Always Free** ARM 实例，性能远超本节中的任何付费方案 —— 每月 $0。

| 你能获得什么 | 规格 |
|--------------|-------|
| **4 个 OCPU** | ARM Ampere A1 |
| **24GB RAM** | 绰绰有余 |
| **200GB 存储** | 块存储 |
| **永久免费** | 无信用卡扣费 |

**注意事项：**

* 注册过程可能比较折腾（失败就多试几次）
* ARM 架构 —— 大多数软件可以正常运行，但部分二进制程序需要 ARM 构建

完整的安装配置指南参见 [Oracle Cloud](/zh/platforms/oracle)。关于注册技巧和开户流程排障，请参考这个[社区指南](https://gist.github.com/rssnyder/51e3cfedd730e7dd5f4a816143b25dbd)。

***

<div id="troubleshooting">
  ## 故障排查
</div>

<div id="gateway-wont-start">
  ### Gateway 无法启动
</div>

```bash
openclaw gateway status
openclaw doctor --non-interactive
journalctl -u openclaw --no-pager -n 50
```

<div id="port-already-in-use">
  ### 端口已被占用
</div>

```bash
lsof -i :18789
kill <PID>
```

<div id="out-of-memory">
  ### 内存耗尽
</div>

```bash
# 检查内存
free -h

# 添加更多交换空间
# 或升级到 $12/月 的 droplet (2GB RAM)
```

***

<div id="see-also">
  ## 另请参阅
</div>

* [Hetzner 指南](/zh/platforms/hetzner) — 更实惠、性能更强
* [Docker 安装](/zh/install/docker) — 容器化安装
* [Tailscale](/zh/gateway/tailscale) — 安全的远程访问
* [配置](/zh/gateway/configuration) — 完整的配置参考