---
title: Raspberry Pi（树莓派）
summary: "在 Raspberry Pi 上运行 OpenClaw（低成本自托管部署）"
read_when:
  - 在 Raspberry Pi 上安装和配置 OpenClaw
  - 在 ARM 设备上运行 OpenClaw
  - 构建低成本、始终在线的个人 AI
---

<div id="openclaw-on-raspberry-pi">
  # 在 Raspberry Pi 上部署 OpenClaw
</div>

<div id="goal">
  ## 目标
</div>

在 Raspberry Pi 上部署一个常驻、始终在线的 OpenClaw Gateway，一次性成本约 **35 至 80 美元**（无月费）。

非常适合：

* 24/7 个人 AI 助手
* 家庭自动化中枢
* 低功耗、始终可用的 Telegram/WhatsApp 机器人

<div id="hardware-requirements">
  ## 硬件要求
</div>

| Pi 型号 | 内存 | 是否可用 | 备注 |
|----------|-----|--------|-------|
| **Pi 5** | 4GB/8GB | ✅ 最佳 | 速度最快，推荐使用 |
| **Pi 4** | 4GB | ✅ 良好 | 对大多数用户来说是理想选择 |
| **Pi 4** | 2GB | ✅ 一般 | 可用，建议添加交换分区 |
| **Pi 4** | 1GB | ⚠️ 吃紧 | 配合交换分区可行，需尽量精简配置 |
| **Pi 3B+** | 1GB | ⚠️ 很慢 | 可以运行但较为卡顿 |
| **Pi Zero 2 W** | 512MB | ❌ | 不推荐 |

**最低配置要求：** 1GB 内存、1 个 CPU 核心、500MB 磁盘空间\
**推荐配置：** 2GB+ 内存、64 位操作系统、16GB+ SD 卡（或 USB SSD）

<div id="what-youll-need">
  ## 准备工作
</div>

* Raspberry Pi 4 或 5（推荐至少 2GB 内存）
* MicroSD 卡（16GB 及以上）或 USB SSD（性能更佳）
* 电源适配器（建议使用官方 Pi 电源）
* 网络连接（以太网或 WiFi）
* 约 30 分钟时间

<div id="1-flash-the-os">
  ## 1) 刷写操作系统
</div>

使用 **Raspberry Pi OS Lite (64-bit)** —— 对于无显示器（headless）的服务器部署，不需要桌面环境。

1. 下载 [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
2. 选择系统：**Raspberry Pi OS Lite (64-bit)**
3. 点击齿轮图标 (⚙️) 进行预配置：
   * 设置主机名：`gateway-host`
   * 启用 SSH
   * 设置用户名/密码
   * 配置 WiFi（如果不使用以太网）
4. 将镜像刷写到 SD 卡 / USB 驱动器
5. 插入介质并启动树莓派

<div id="2-connect-via-ssh">
  ## 2) 通过 SSH 连接
</div>

```bash
ssh user@gateway-host
# 或使用 IP 地址
ssh user@192.168.x.x
```

<div id="3-system-setup">
  ## 3) 系统准备
</div>

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install essential packages
sudo apt install -y git curl build-essential

# 设置时区（对 cron/提醒功能很重要）
sudo timedatectl set-timezone America/Chicago  # 改为你的时区
```

<div id="4-install-nodejs-22-arm64">
  ## 4) 安装 Node.js 22（ARM64）
</div>

```bash
# Install Node.js via NodeSource
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs

# Verify
node --version  # 应显示 v22.x.x
npm --version
```

<div id="5-add-swap-important-for-2gb-or-less">
  ## 5) 添加 Swap（对内存为 2GB 及以下的设备尤为重要）
</div>

Swap 可以帮助避免因内存不足导致的系统崩溃：

```bash
# Create 2GB swap file
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make permanent
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# 针对低内存优化（降低交换积极性）
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

<div id="6-install-openclaw">
  ## 6）安装 OpenClaw
</div>

<div id="option-a-standard-install-recommended">
  ### 选项 A：标准安装（推荐）
</div>

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

<div id="option-b-hackable-install-for-tinkering">
  ### 选项 B：可 Hack 安装（适合折腾）
</div>

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
npm install
npm run build
npm link
```

可 Hack 的安装方式让你可以直接访问日志和代码，对于调试 ARM 特定问题非常有用。

<div id="7-run-onboarding">
  ## 7）执行初始化流程
</div>

```bash
openclaw onboard --install-daemon
```

按照向导进行操作：

1. **Gateway 模式：** 本地
2. **认证：** 建议使用 API 密钥（在无头的树莓派上 OAuth 可能不太稳定）
3. **通道：** Telegram 是最容易上手的
4. **守护进程：** 启用（systemd）

<div id="8-verify-installation">
  ## 8）验证安装
</div>

```bash
# 检查状态
openclaw status

# 检查服务
sudo systemctl status openclaw

# 查看日志
journalctl -u openclaw -f
```

<div id="9-access-the-dashboard">
  ## 9) 访问 Dashboard
</div>

由于树莓派是无头运行的，请使用 SSH 隧道：

```bash
# 从你的笔记本电脑/台式机
ssh -L 18789:localhost:18789 user@gateway-host

# 然后在浏览器中打开
open http://localhost:18789
```

或者使用 Tailscale 来实现始终可用的访问：

```bash
# On the Pi
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up

# 更新配置
openclaw config set gateway.bind tailnet
sudo systemctl restart openclaw
```

***

<div id="performance-optimizations">
  ## 性能优化
</div>

<div id="use-a-usb-ssd-huge-improvement">
  ### 使用 USB SSD（大幅提升）
</div>

SD 卡速度慢，而且寿命有限。改用 USB SSD 可以大幅提升性能：

```bash
# 检查是否从 USB 启动
lsblk
```

有关设置步骤，请参阅 [Pi USB 启动指南](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#usb-mass-storage-boot)。

<div id="reduce-memory-usage">
  ### 降低内存占用
</div>

```bash
# 禁用 GPU 内存分配（无头模式）
echo 'gpu_mem=16' | sudo tee -a /boot/config.txt

# Disable Bluetooth if not needed
sudo systemctl disable bluetooth
```

<div id="monitor-resources">
  ### 监控系统资源
</div>

```bash
# 检查内存
free -h

# 检查 CPU 温度
vcgencmd measure_temp

# 实时监控
htop
```

***

<div id="arm-specific-notes">
  ## ARM 特定注意事项
</div>

<div id="binary-compatibility">
  ### 二进制兼容性
</div>

大多数 OpenClaw 功能在 ARM64 上都能正常运行，但某些外部二进制工具可能需要提供 ARM 构建版本：

| Tool | ARM64 状态 | 说明 |
|------|--------------|-------|
| Node.js | ✅ | 运行良好 |
| WhatsApp (Baileys) | ✅ | 纯 JS，无问题 |
| Telegram | ✅ | 纯 JS，无问题 |
| gog (Gmail CLI) | ⚠️ | 检查是否有 ARM 版本发布 |
| Chromium (browser) | ✅ | `sudo apt install chromium-browser` |

如果某个 skill 运行失败，请检查其对应的二进制文件是否有 ARM 构建版本。许多 Go/Rust 工具都提供了 ARM 构建，但也有一些没有。

<div id="32-bit-vs-64-bit">
  ### 32 位 vs 64 位
</div>

**务必使用 64 位操作系统。** Node.js 和许多现代工具都依赖它。请使用以下命令进行检查：

```bash
uname -m
# 应显示：aarch64（64 位）而非 armv7l（32 位）
```

***

<div id="recommended-model-setup">
  ## 推荐的模型配置
</div>

由于树莓派只运行 Gateway（模型在云端运行），建议使用基于 api 的模型：

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-sonnet-4-20250514",
        "fallbacks": ["openai/gpt-4o-mini"]
      }
    }
  }
}
```

**不要尝试在树莓派上运行本地 LLM** —— 即便是小模型，速度也慢得不可用。让 Claude/GPT 来干重活。

***

<div id="auto-start-on-boot">
  ## 开机自动启动
</div>

入门向导已经帮你配置好了，但你可以按以下步骤进行确认：

```bash
# 检查服务是否已启用
sudo systemctl is-enabled openclaw

# Enable if not
sudo systemctl enable openclaw

# Start on boot
sudo systemctl start openclaw
```

***

<div id="troubleshooting">
  ## 故障排查
</div>

<div id="out-of-memory-oom">
  ### 内存不足（OOM）
</div>

```bash
# 检查内存
free -h

# 添加更多交换空间(参见步骤 5)
# 或减少在 Pi 上运行的服务
```

<div id="slow-performance">
  ### 性能变慢
</div>

* 使用 USB SSD，而不是 SD 卡
* 禁用未使用的服务：`sudo systemctl disable cups bluetooth avahi-daemon`
* 检查 CPU 降频状态：`vcgencmd get_throttled`（应返回 `0x0`）

<div id="service-wont-start">
  ### 服务无法启动
</div>

```bash
# 检查日志
journalctl -u openclaw --no-pager -n 100

# 常见修复：重新构建
cd ~/openclaw  # 如果使用可修改的安装方式
npm run build
sudo systemctl restart openclaw
```

<div id="arm-binary-issues">
  ### ARM 二进制问题
</div>

如果某个 skill 运行失败并报错 “exec format error”：

1. 检查该二进制文件是否提供 ARM64 构建版本
2. 尝试从源码进行构建
3. 或使用支持 ARM 的 Docker 容器

<div id="wifi-drops">
  ### WiFi 掉线
</div>

对于通过 WiFi 连接的无头（headless）树莓派：

```bash
# 禁用 WiFi 电源管理
sudo iwconfig wlan0 power off

# Make permanent
echo 'wireless-power off' | sudo tee -a /etc/network/interfaces
```

***

<div id="cost-comparison">
  ## 成本对比
</div>

| 部署方案 | 一次性成本 | 每月成本 | 备注 |
|-------|---------------|--------------|-------|
| **Pi 4 (2GB)** | 约 $45 | $0 | 另加电费（约 $5/年） |
| **Pi 4 (4GB)** | 约 $55 | $0 | 推荐配置 |
| **Pi 5 (4GB)** | 约 $60 | $0 | 性能最佳 |
| **Pi 5 (8GB)** | 约 $80 | $0 | 性能过剩，但更具前瞻性 |
| DigitalOcean | $0 | $6/月 | $72/年 |
| Hetzner | $0 | €3.79/月 | 约 $50/年 |

**回本点：** 与云端 VPS 相比，一块 Pi 大约在 6–12 个月内即可回本。

***

<div id="see-also">
  ## 另见
</div>

* [Linux 指南](/zh/platforms/linux) — 通用 Linux 安装配置
* [DigitalOcean 指南](/zh/platforms/digitalocean) — 云端托管替代方案
* [Hetzner 指南](/zh/platforms/hetzner) — Docker 部署
* [Tailscale](/zh/gateway/tailscale) — 远程访问
* [节点](/zh/nodes) — 将你的笔记本电脑/手机与 Pi Gateway 配对