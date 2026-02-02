---
title: macOS 虚拟机
summary: "当你需要隔离或使用 iMessage 时，在受沙箱保护的 macOS 虚拟机（本地或远程托管）中运行 OpenClaw"
read_when:
  - 你想将 OpenClaw 与主 macOS 环境隔离开来
  - 你想在沙箱中使用 iMessage 集成（BlueBubbles）
  - 你想要一个可重置、可克隆的 macOS 环境
  - 你想对比本地和远程托管的 macOS 虚拟机方案
---

<div id="openclaw-on-macos-vms-sandboxing">
  # 在 macOS 虚拟机中运行 OpenClaw（沙箱）
</div>

<div id="recommended-default-most-users">
  ## 推荐默认方案（适合大多数用户）
</div>

* **小型 Linux VPS**：用于始终在线的 Gateway，成本较低。参见 [VPS hosting](/zh/vps)。
* **独立硬件**（Mac mini 或 Linux 机器）：适合你需要完全控制权，并且需要用于浏览器自动化的**家庭宽带 IP（residential IP）**时使用。很多网站会屏蔽机房 IP，因此本地浏览通常效果更好。
* **混合方案：**将 Gateway 部署在廉价 VPS 上，需要浏览器/UI 自动化时，再把你的 Mac 作为**节点**连接进来。参见 [Nodes](/zh/nodes) 和 [Gateway remote](/zh/gateway/remote)。

仅当你确实需要 macOS 独有能力（如 iMessage/BlueBubbles），或者需要与你日常使用的 Mac 严格隔离时，才使用 macOS 虚拟机（VM）。

<div id="macos-vm-options">
  ## macOS 虚拟机方案
</div>

<div id="local-vm-on-your-apple-silicon-mac-lume">
  ### 在你的 Apple Silicon Mac 上本地运行 VM（Lume）
</div>

使用 [Lume](https://cua.ai/docs/lume)，在你现有的 Apple Silicon Mac 上的沙箱化 macOS VM 中运行 OpenClaw。

这样你可以获得：

* 与宿主机完全隔离的完整 macOS 环境（宿主系统始终保持干净）
* 通过 BlueBubbles 实现 iMessage 支持（在 Linux/Windows 上无法实现）
* 通过克隆 VM 实现快速重置
* 无需额外硬件或云端开销

<div id="hosted-mac-providers-cloud">
  ### 托管 Mac 提供方（云端）
</div>

如果你想在云端使用 macOS，也可以使用托管 Mac 提供方：

* [MacStadium](https://www.macstadium.com/)（托管 Mac）
* 其他托管 Mac 供应商同样可行；按其 VM 和 SSH 文档操作即可

一旦你获得对 macOS VM 的 SSH 访问权限，请从下面的第 6 步继续。

***

<div id="quick-path-lume-experienced-users">
  ## 快速流程（Lume，适合有经验的用户）
</div>

1. 安装 Lume
2. `lume create openclaw --os macos --ipsw latest`
3. 完成 Setup Assistant，启用 Remote Login（远程登录/SSH）
4. `lume run openclaw --no-display`
5. 使用 SSH 登录，安装 OpenClaw 并配置渠道
6. 完成

***

<div id="what-you-need-lume">
  ## 所需条件（Lume）
</div>

* Apple Silicon Mac（M1/M2/M3/M4）
* 宿主机需运行 macOS Sequoia 或更高版本
* 每个 VM 约需 60 GB 可用磁盘空间
* 耗时约 20 分钟

***

<div id="1-install-lume">
  ## 1) 安装 Lume
</div>

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/trycua/cua/main/libs/lume/scripts/install.sh)"
```

如果 `~/.local/bin` 没有包含在你的 PATH 环境变量中：

```bash
echo 'export PATH="$PATH:$HOME/.local/bin"' >> ~/.zshrc && source ~/.zshrc
```

验证：

```bash
lume --version
```

文档：[Lume 安装](https://cua.ai/docs/lume/guide/getting-started/installation)

***

<div id="2-create-the-macos-vm">
  ## 2) 创建 macOS 虚拟机
</div>

```bash
lume create openclaw --os macos --ipsw latest
```

这会下载 macOS 并创建虚拟机。VNC 窗口会自动弹出。

注意：下载过程可能需要一段时间，具体取决于你的网络状况。

***

<div id="3-complete-setup-assistant">
  ## 3) 完成设置助理
</div>

在 VNC 窗口中：

1. 选择语言和地区
2. 跳过 Apple ID（如果你之后想用 iMessage，也可以在这里登录）
3. 创建用户帐户（记住用户名和密码）
4. 跳过所有可选功能

完成初始设置后，启用 SSH：

1. 打开“系统设置” → “通用” → “共享”
2. 启用“远程登录”

***

<div id="4-get-the-vms-ip-address">
  ## 4) 获取 VM 的 IP 地址
</div>

```bash
lume get openclaw
```

查找 IP 地址（通常为 `192.168.64.x`）。

***

<div id="5-ssh-into-the-vm">
  ## 5) 使用 SSH 登录虚拟机
</div>

```bash
ssh youruser@192.168.64.X
```

将 `youruser` 替换为你创建的用户，将 IP 替换为该虚拟机的 IP 地址。

***

<div id="6-install-openclaw">
  ## 6) 安装 OpenClaw
</div>

在虚拟机中：

```bash
npm install -g openclaw@latest
openclaw onboard --install-daemon
```

按照引导完成模型提供方（Anthropic、OpenAI 等）的配置。

***

<div id="7-configure-channels">
  ## 7) 配置渠道
</div>

编辑配置文件：

```bash
nano ~/.openclaw/openclaw.json
```

添加你的消息渠道：

```json
{
  "channels": {
    "whatsapp": {
      "dmPolicy": "allowlist", // 直接消息策略:允许列表
      "allowFrom": ["+15551234567"]
    },
    "telegram": {
      "botToken": "YOUR_BOT_TOKEN"
    }
  }
}
```

然后在 WhatsApp 中登录（扫码二维码）：

```bash
openclaw channels login
```

***

<div id="8-run-the-vm-headlessly">
  ## 8) 以无头模式运行虚拟机
</div>

先停止虚拟机，然后在不启用显示界面的情况下重新启动：

```bash
lume stop openclaw
lume run openclaw --no-display
```

虚拟机在后台运行。OpenClaw 的守护进程会确保 Gateway 持续运行。

要检查状态：

```bash
ssh youruser@192.168.64.X "openclaw status"
```

***

<div id="bonus-imessage-integration">
  ## 附加功能：iMessage 集成
</div>

这是在 macOS 上运行的杀手级功能。使用 [BlueBubbles](https://bluebubbles.app) 将 iMessage 集成到 OpenClaw 中。

在虚拟机中执行以下操作：

1. 从 bluebubbles.app 下载 BlueBubbles
2. 使用你的 Apple ID 登录
3. 启用 Web API 并设置密码
4. 将 BlueBubbles 的 webhook 指向你的 Gateway（示例：`https://your-gateway-host:3000/bluebubbles-webhook?password=<password>`）

在你的 OpenClaw 配置中添加：

```json
{
  "channels": {
    "bluebubbles": {
      "serverUrl": "http://localhost:1234",
      "password": "your-api-password",
      "webhookPath": "/bluebubbles-webhook"
    }
  }
}
```

重启 Gateway。现在你的智能体可以发送和接收 iMessage 了。

完整设置说明：[BlueBubbles 通道](/zh/channels/bluebubbles)

***

<div id="save-a-golden-image">
  ## 保存黄金镜像
</div>

在进一步自定义之前，先为当前的干净状态创建一个快照：

```bash
lume stop openclaw
lume clone openclaw openclaw-golden
```

随时重置：

```bash
lume stop openclaw && lume delete openclaw
lume clone openclaw-golden openclaw
lume run openclaw --no-display
```

***

<div id="running-247">
  ## 24/7 运行
</div>

通过以下方式让 VM 持续运行：

* 保持 Mac 接通电源
* 在“系统设置” → “节能”中禁用睡眠
* 需要时使用 `caffeinate`

如果你需要真正的“始终在线”，可以考虑一台专用的 Mac mini 或一个小型 VPS。参见 [VPS 托管](/zh/vps)。

***

<div id="troubleshooting">
  ## 故障排查
</div>

| 问题 | 解决方案 |
|---------|----------|
| 无法通过 SSH 连接到 VM | 检查 VM 的系统设置中是否启用了“远程登录（Remote Login）” |
| 未显示 VM 的 IP 地址 | 等待 VM 完全启动后，再次运行 `lume get openclaw` |
| 找不到 lume 命令 | 将 `~/.local/bin` 添加到你的 PATH 环境变量中 |
| 无法扫描 WhatsApp 二维码 | 确保是在 VM 中（而不是宿主机）运行 `openclaw channels login` |
---------------------------------------------------------------------

## 相关文档

* [VPS 托管](/zh/vps)
* [节点](/zh/nodes)
* [Gateway 远程访问](/zh/gateway/remote)
* [BlueBubbles 通道](/zh/channels/bluebubbles)
* [Lume 快速入门](https://cua.ai/docs/lume/guide/getting-started/quickstart)
* [Lume CLI 参考](https://cua.ai/docs/lume/reference/cli-reference)
* [无人值守 VM 配置](https://cua.ai/docs/lume/guide/fundamentals/unattended-setup)（高级）
* [Docker 沙箱](/zh/install/docker)（另一种隔离方式）