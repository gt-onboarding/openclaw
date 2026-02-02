---
title: VPS
summary: "OpenClaw 的 VPS 托管导航（Oracle/Fly/Hetzner/GCP/exe.dev）"
read_when:
  - 你想在云端运行 Gateway
  - 你需要一份 VPS/托管指南的快速索引
---

<div id="vps-hosting">
  # VPS 托管
</div>

本页汇总受支持的 VPS/托管指南，并从整体上说明云端部署的大致工作方式。

<div id="pick-a-provider">
  ## 选择提供方
</div>

* **Railway**（一键部署 + 浏览器配置）：[Railway](/zh/railway)
* **Northflank**（一键部署 + 浏览器配置）：[Northflank](/zh/northflank)
* **Oracle Cloud（Always Free）**：[Oracle](/zh/platforms/oracle) — $0/月（Always Free，ARM；容量/注册情况可能不太稳定）
* **Fly.io**：[Fly.io](/zh/platforms/fly)
* **Hetzner（Docker）**：[Hetzner](/zh/platforms/hetzner)
* **GCP（Compute Engine）**：[GCP](/zh/platforms/gcp)
* **exe.dev**（VM + HTTPS 代理）：[exe.dev](/zh/platforms/exe-dev)
* **AWS（EC2/Lightsail/free tier）**：也很好用。视频教程：
  https://x.com/techfrenAJ/status/2014934471095812547

<div id="how-cloud-setups-work">
  ## 云端部署的工作方式
</div>

* **Gateway 运行在 VPS 上**，并管理状态和工作区。
* 你从笔记本电脑/手机通过 **Control UI** 或 **Tailscale/SSH** 进行连接。
* 将 VPS 视为权威数据源，并对状态和工作区进行**备份**。
* 默认安全做法：将 Gateway 绑定在回环地址上，通过 SSH 隧道或 Tailscale Serve 访问。
  如果绑定到 `lan`/`tailnet`，则要求使用 `gateway.auth.token` 或 `gateway.auth.password`。

远程访问： [Gateway 远程访问](/zh/gateway/remote)\
平台概览： [平台](/zh/platforms)

<div id="using-nodes-with-a-vps">
  ## 在 VPS 上使用节点
</div>

你可以把 Gateway 部署在云端，并在本地设备上配对 **节点**
（Mac/iOS/Android/无头模式）。节点在本地提供屏幕/相机/画布和 `system.run`
功能，而 Gateway 则持续运行在云端。

文档：[Nodes](/zh/nodes)、[Nodes CLI](/zh/cli/nodes)