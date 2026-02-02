---
title: Windows
summary: "Windows（WSL2）支持情况与配套应用状态"
read_when:
  - 在 Windows 上安装 OpenClaw
  - 查看 Windows 配套应用状态
---

<div id="windows-wsl2">
  # Windows（WSL2）
</div>

在 Windows 上使用 OpenClaw 时，推荐通过 **WSL2**（推荐使用 Ubuntu 发行版）。CLI 和 Gateway 在 Linux 中运行，可以保持运行时环境的一致性，并显著提升工具链兼容性（Node/Bun/pnpm、Linux 可执行文件、技能等）。原生 Windows 安装尚未经过测试，且更容易出现问题。

原生 Windows 配套应用已在规划中。

<div id="install-wsl2">
  ## 安装（WSL2）
</div>

* [快速开始](/zh/start/getting-started)（在 WSL2 中使用）
* [安装与更新](/zh/install/updating)
* 官方 WSL2 指南（Microsoft）：https://learn.microsoft.com/windows/wsl/install

<div id="gateway">
  ## Gateway
</div>

* [Gateway 运维手册](/zh/gateway)
* [配置](/zh/gateway/configuration)

<div id="gateway-service-install-cli">
  ## Gateway 服务安装（CLI）
</div>

在 WSL2 环境中：

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

<div id="advanced-expose-wsl-services-over-lan-portproxy">
  ## 进阶：通过局域网暴露 WSL 服务（portproxy）
</div>

WSL 有自己独立的虚拟网络。如果另一台机器需要访问运行在 **WSL 内部** 的服务
（SSH、本地 TTS 服务器或 Gateway），你必须将某个 Windows 端口转发到当前的
WSL IP 地址。WSL 的 IP 在重启后会变化，因此你可能需要更新该转发规则。

示例（以 **管理员身份** 运行 PowerShell）：

```powershell
$Distro = "Ubuntu-24.04"
$ListenPort = 2222
$TargetPort = 22

$WslIp = (wsl -d $Distro -- hostname -I).Trim().Split(" ")[0]
if (-not $WslIp) { throw "WSL IP not found." }

netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=$ListenPort `
  connectaddress=$WslIp connectport=$TargetPort
```

在 Windows 防火墙中放行该端口（仅需执行一次）：

```powershell
New-NetFirewallRule -DisplayName "WSL SSH $ListenPort" -Direction Inbound `
  -Protocol TCP -LocalPort $ListenPort -Action Allow
```

WSL 重启后需要刷新 portproxy：

```powershell
netsh interface portproxy delete v4tov4 listenport=$ListenPort listenaddress=0.0.0.0 | Out-Null
netsh interface portproxy add v4tov4 listenport=$ListenPort listenaddress=0.0.0.0 `
  connectaddress=$WslIp connectport=$TargetPort | Out-Null
```

注意事项：

* 从另一台机器通过 SSH 连接时，目标是 **Windows 主机 IP**（示例：`ssh user@windows-host -p 2222`）。
* 远程节点必须指向一个**可访问的** Gateway URL（不能是 `127.0.0.1`）；使用
  `openclaw status --all` 来确认。
* 需要局域网访问时使用 `listenaddress=0.0.0.0`；`127.0.0.1` 只在本机可用。
* 如果你希望这一过程自动执行，可注册一个计划任务（Scheduled Task），在登录时运行刷新步骤。

<div id="step-by-step-wsl2-install">
  ## WSL2 分步安装
</div>

<div id="1-install-wsl2-ubuntu">
  ### 1) 安装 WSL2 + Ubuntu
</div>

以管理员身份打开 PowerShell：

```powershell
wsl --install
# 或明确选择一个发行版：
wsl --list --online
wsl --install -d Ubuntu-24.04
```

如果 Windows 要求重启，请重新启动电脑。

<div id="2-enable-systemd-required-for-gateway-install">
  ### 2) 启用 systemd（Gateway 安装所必需）
</div>

在你的 WSL 终端中：

```bash
sudo tee /etc/wsl.conf >/dev/null <<'EOF'
[boot]
systemd=true
EOF
```

然后在 PowerShell 中执行：

```powershell
wsl --shutdown
```

重新打开 Ubuntu，然后检查：

```bash
systemctl --user status
```

<div id="3-install-openclaw-inside-wsl">
  ### 3) 在 WSL 中安装 OpenClaw（WSL 环境内）
</div>

在 WSL 中按照 Linux 快速开始流程进行操作：

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # 首次运行时自动安装 UI 依赖项
pnpm build
openclaw onboard
```

完整指南：[快速开始](/zh/start/getting-started)

<div id="windows-companion-app">
  ## Windows 配套应用
</div>

我们目前还没有 Windows 配套应用。如果你希望它早日实现，非常欢迎你来贡献代码，帮我们一起把它做出来。