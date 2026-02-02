---
title: Oracle
summary: "在 Oracle Cloud Always Free ARM 实例上运行 OpenClaw"
read_when:
  - 在 Oracle Cloud 上设置 OpenClaw
  - 为 OpenClaw 寻找低成本 VPS 托管方案
  - 希望在小型服务器上 24/7 持续运行 OpenClaw
---

<div id="openclaw-on-oracle-cloud-oci">
  # Oracle Cloud（OCI）上的 OpenClaw
</div>

<div id="goal">
  ## 目标
</div>

在 Oracle Cloud 的 **Always Free** ARM 免费层上运行一个长期运行的 OpenClaw Gateway。

Oracle 的免费层对 OpenClaw 来说非常适合（尤其是如果你已经有 OCI 账号），但也存在一些取舍：

* ARM 架构（大多数组件都能正常工作，但有些二进制可能仅提供 x86 版本）
* 容量和注册过程可能比较挑人 / 不太稳定

<div id="cost-comparison-2026">
  ## 成本对比（2026）
</div>

| 提供方 | 方案 | 规格 | 价格/月 | 备注 |
|----------|------|-------|----------|-------|
| Oracle Cloud | Always Free ARM | 最多 4 个 OCPU，24GB 内存 | $0 | ARM，容量有限 |
| Hetzner | CX22 | 2 vCPU，4GB 内存 | 约 $4 | 最便宜的付费选项 |
| DigitalOcean | Basic | 1 vCPU，1GB 内存 | $6 | UI 简洁，文档完善 |
| Vultr | Cloud Compute | 1 vCPU，1GB 内存 | $6 | 可选机房/区域多 |
| Linode | Nanode | 1 vCPU，1GB 内存 | $5 | 现已并入 Akamai |

***

<div id="prerequisites">
  ## 前置条件
</div>

* Oracle Cloud 账户（[注册](https://www.oracle.com/cloud/free/)）——如果遇到问题，请参考[社区注册指南](https://gist.github.com/rssnyder/51e3cfedd730e7dd5f4a816143b25dbd)
* Tailscale 账户（可在 [tailscale.com](https://tailscale.com) 免费注册）
* 约 30 分钟

<div id="1-create-an-oci-instance">
  ## 1) 创建 OCI 实例
</div>

1. 登录 [Oracle Cloud Console](https://cloud.oracle.com/)
2. 导航到 **Compute → Instances → Create Instance**
3. 配置：
   * **Name：** `openclaw`
   * **Image：** Ubuntu 24.04 (aarch64)
   * **Shape：** `VM.Standard.A1.Flex`（Ampere ARM）
   * **OCPUs：** 2（或最多 4）
   * **Memory：** 12 GB（或最多 24 GB）
   * **Boot volume：** 50 GB（最多可用 200 GB）
   * **SSH key：** 添加你的公钥
4. 点击 **Create**
5. 记录公网 IP 地址

**提示：** 如果实例创建失败并显示 “Out of capacity”，尝试选择不同的可用性域或稍后重试。免费层容量有限。

<div id="2-connect-and-update">
  ## 2）连接并更新
</div>

```bash
# 通过公网 IP 连接
ssh ubuntu@YOUR_PUBLIC_IP

# Update system
sudo apt update && sudo apt upgrade -y
sudo apt install -y build-essential
```

**注意：** 在 ARM 架构上编译某些依赖项时，需要安装 `build-essential`。

<div id="3-configure-user-and-hostname">
  ## 3) 配置用户和主机名
</div>

```bash
# 设置主机名
sudo hostnamectl set-hostname openclaw

# 为 ubuntu 用户设置密码
sudo passwd ubuntu

# 启用 lingering(使用户服务在注销后保持运行)
sudo loginctl enable-linger ubuntu
```

<div id="4-install-tailscale">
  ## 4) 安装 Tailscale
</div>

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up --ssh --hostname=openclaw
```

这将启用 Tailscale SSH，因此你可以在 tailnet 中的任何设备上通过 `ssh openclaw` 进行连接——无需公网 IP。

请验证：

```bash
tailscale status
```

**从现在起，请通过 Tailscale 连接：** `ssh ubuntu@openclaw`（或使用 Tailscale 的 IP）。

<div id="5-install-openclaw">
  ## 5) 安装 OpenClaw
</div>

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
source ~/.bashrc
```

当出现 “How do you want to hatch your bot?” 提示时，选择 **“Do this later”**。

> 注意：如果遇到 ARM 架构下的原生构建问题，请先安装系统软件包（例如 `sudo apt install -y build-essential`），再考虑使用 Homebrew。

<div id="6-configure-gateway-loopback-token-auth-and-enable-tailscale-serve">
  ## 6) 配置 Gateway（回环地址 + 令牌认证）并启用 Tailscale Serve
</div>

将令牌认证设为默认方式。它的行为更可预期，并且无需在 Control UI 中启用任何“不安全认证”开关。

```bash
# Keep the Gateway private on the VM
openclaw config set gateway.bind loopback

# Require auth for the Gateway + Control UI
openclaw config set gateway.auth.mode token
openclaw doctor --generate-gateway-token

# 通过 Tailscale Serve 暴露(HTTPS + tailnet 访问)
openclaw config set gateway.tailscale.mode serve
openclaw config set gateway.trustedProxies '["127.0.0.1"]'

systemctl --user restart openclaw-gateway
```

<div id="7-verify">
  ## 7) 验证
</div>

```bash
# 检查版本
openclaw --version

# 检查守护进程状态
systemctl --user status openclaw-gateway

# 检查 Tailscale Serve 状态
tailscale serve status

# 测试本地响应
curl http://localhost:18789
```

<div id="8-lock-down-vcn-security">
  ## 8) 锁定 VCN 安全
</div>

确认一切工作正常后，锁定 VCN，仅允许 Tailscale 流量，阻止所有其他流量。OCI 的 Virtual Cloud Network（VCN）在网络边缘充当防火墙——流量会在到达实例之前就被拦截。

1. 在 OCI Console 中进入 **Networking → Virtual Cloud Networks**
2. 点击你的 VCN → **Security Lists** → Default Security List
3. **移除**所有入站规则，仅保留：
   * `0.0.0.0/0 UDP 41641`（Tailscale）
4. 保留默认出站规则（允许所有出站流量）

这会在网络边缘阻断 22 端口上的 SSH、HTTP、HTTPS 以及其他所有流量。从现在开始，你只能通过 Tailscale 进行连接。

***

<div id="access-the-control-ui">
  ## 访问 Control UI
</div>

在 Tailscale 网络中的任意设备上：

```
https://openclaw.<tailnet-name>.ts.net/
```

将 `<tailnet-name>` 替换为你的 tailnet 名称（可在 `tailscale status` 中查看）。

不需要 SSH 隧道。Tailscale 提供：

* HTTPS 加密（自动签发证书）
* 基于 Tailscale 身份的认证
* 可从 tailnet 内的任意设备访问（笔记本电脑、手机等）

***

<div id="security-vcn-tailscale-recommended-baseline">
  ## 安全性：VCN + Tailscale（推荐基础配置）
</div>

在 VCN 已锁定（仅开放 UDP 41641 端口）且 Gateway 只绑定到本地回环地址的情况下，你可以获得很强的纵深防御：公网流量在网络边界就被阻断，管理访问通过你的 tailnet 完成。

这种配置通常可以免去*仅仅*为了阻止面向全网的 SSH 暴力破解而额外配置主机防火墙规则的需求——但你仍然应该保持操作系统及时更新，运行 `openclaw security audit`，并确认自己没有不小心在公网接口上监听。

<div id="whats-already-protected">
  ### 已经受到哪些保护
</div>

| 传统步骤 | 还需要吗？ | 原因 |
|------------------|---------|-----|
| UFW firewall | 不需要 | VCN 会在流量到达实例之前就进行阻断 |
| fail2ban | 不需要 | 如果在 VCN 层阻断了 22 端口，就不会出现暴力破解 |
| sshd 加固 | 不需要 | Tailscale SSH 不使用 sshd |
| 禁用 root 登录 | 不需要 | Tailscale 使用 Tailscale 身份，而不是系统用户 |
| 仅允许 SSH 密钥认证 | 不需要 | Tailscale 通过你的 tailnet 进行身份验证 |
| IPv6 加固 | 通常不需要 | 取决于你的 VCN/子网设置；请核实实际分配/暴露的内容 |

<div id="still-recommended">
  ### 仍建议执行
</div>

* **凭据目录权限：** `chmod 700 ~/.openclaw`
* **安全审计：** `openclaw security audit`
* **系统更新：** 定期执行 `sudo apt update && sudo apt upgrade`
* **监控 Tailscale：** 在 [Tailscale 管理控制台](https://login.tailscale.com/admin) 中查看设备

<div id="verify-security-posture">
  ### 核查安全防护状态
</div>

```bash
# 确认没有公共端口在监听
sudo ss -tlnp | grep -v '127.0.0.1\|::1'

# 验证 Tailscale SSH 是否激活
tailscale status | grep -q 'offers: ssh' && echo "Tailscale SSH active"

# 可选：完全禁用 sshd
sudo systemctl disable --now ssh
```

***

<div id="fallback-ssh-tunnel">
  ## 回退方案：SSH 隧道
</div>

如果 Tailscale Serve 不可用，请使用 SSH 隧道：

```bash
# 从本地机器（通过 Tailscale）
ssh -L 18789:127.0.0.1:18789 ubuntu@openclaw
```

然后打开 `http://localhost:18789`。

***

<div id="troubleshooting">
  ## 故障排查
</div>

<div id="instance-creation-fails-out-of-capacity">
  ### 实例创建失败（&quot;Out of capacity&quot;）
</div>

免费层 ARM 实例非常受欢迎。你可以尝试：

* 使用不同的可用性域
* 在非高峰时段（清晨）重试
* 在选择计算规格（shape）时使用 &quot;Always Free&quot; 筛选器

<div id="tailscale-wont-connect">
  ### Tailscale 无法连接
</div>

```bash
# 检查状态
sudo tailscale status

# 重新进行身份验证
sudo tailscale up --ssh --hostname=openclaw --reset
```

<div id="gateway-wont-start">
  ### Gateway 无法启动
</div>

```bash
openclaw gateway status
openclaw doctor --non-interactive
journalctl --user -u openclaw-gateway -n 50
```

<div id="cant-reach-control-ui">
  ### 无法访问 Control UI
</div>

```bash
# 验证 Tailscale Serve 是否正在运行
tailscale serve status

# Check gateway is listening
curl http://localhost:18789

# Restart if needed
systemctl --user restart openclaw-gateway
```

<div id="arm-binary-issues">
  ### ARM 二进制文件问题
</div>

某些工具可能没有适用于 ARM 的构建版本。请检查：

```bash
uname -m  # 应显示 aarch64
```

大多数 npm 包都能正常运行。对于二进制包，请选择 `linux-arm64` 或 `aarch64` 版本。

***

<div id="persistence">
  ## 持久化
</div>

所有状态位于：

* `~/.openclaw/` — 配置、凭据、会话数据
* `~/.openclaw/workspace/` — 工作区（SOUL.md、memory、artifacts）

请定期备份：

```bash
tar -czvf openclaw-backup.tar.gz ~/.openclaw ~/.openclaw/workspace
```

***

<div id="see-also">
  ## 另请参阅
</div>

* [Gateway 远程访问](/zh/gateway/remote) — 其他远程访问模式
* [Tailscale 集成](/zh/gateway/tailscale) — 完整的 Tailscale 文档
* [Gateway 配置](/zh/gateway/configuration) — 所有配置选项
* [DigitalOcean 指南](/zh/platforms/digitalocean) — 适合想要付费但注册流程更简单的方案
* [Hetzner 指南](/zh/platforms/hetzner) — 基于 Docker 的替代方案