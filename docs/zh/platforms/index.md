---
title: 平台
summary: "平台支持概览（Gateway + 配套应用程序）"
read_when:
  - 查找操作系统支持情况或安装路径
  - 决定在哪个平台上运行 Gateway
---

<div id="platforms">
  # 平台
</div>

OpenClaw 核心使用 TypeScript 编写。**推荐使用 Node 作为运行时环境**。
不推荐在 Gateway 上使用 Bun（会导致 WhatsApp/Telegram 相关问题）。

目前已提供适用于 macOS（菜单栏应用）和移动节点（iOS/Android）的配套应用。
Windows 和 Linux 的配套应用正在规划中，但这些平台已经完全支持运行 Gateway。
也计划提供 Windows 原生配套应用；目前推荐通过 WSL2 运行 Gateway。

<div id="choose-your-os">
  ## 选择操作系统
</div>

* macOS: [macOS](/zh/platforms/macos)
* iOS: [iOS](/zh/platforms/ios)
* Android: [Android](/zh/platforms/android)
* Windows: [Windows](/zh/platforms/windows)
* Linux: [Linux](/zh/platforms/linux)

<div id="vps-hosting">
  ## VPS 与托管
</div>

* VPS 汇总页：[VPS hosting](/zh/vps)
* Fly.io：[Fly.io](/zh/platforms/fly)
* Hetzner（Docker）：[Hetzner](/zh/platforms/hetzner)
* GCP（Compute Engine）：[GCP](/zh/platforms/gcp)
* exe.dev（VM + HTTPS 代理）：[exe.dev](/zh/platforms/exe-dev)

<div id="common-links">
  ## 常用链接
</div>

* 安装指南：[快速开始](/zh/start/getting-started)
* Gateway 运行手册：[Gateway](/zh/gateway)
* Gateway 配置文档：[Configuration](/zh/gateway/configuration)
* 服务状态：`openclaw gateway status`

<div id="gateway-service-install-cli">
  ## Gateway 服务安装（CLI）
</div>

使用以下任一方式（均受支持）：

* 向导（推荐）：`openclaw onboard --install-daemon`
* 直接安装：`openclaw gateway install`
* 配置流程：`openclaw configure` → 选择 **Gateway service**
* 修复/迁移：`openclaw doctor`（可选择安装或修复该服务）

具体使用的服务机制取决于操作系统：

* macOS：LaunchAgent（`bot.molt.gateway` 或 `bot.molt.<profile>`；旧版为 `com.openclaw.*`）
* Linux/WSL2：systemd 用户服务（`openclaw-gateway[-<profile>].service`）