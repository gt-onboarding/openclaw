---
title: 网络
summary: "网络枢纽：Gateway 对外界面、配对、发现与安全"
read_when:
  - 你需要了解网络架构和安全总览
  - 你在调试本地访问与 tailnet 访问或配对问题
  - 你想要查看网络文档的权威列表
---

<div id="network-hub">
  # 网络枢纽
</div>

此枢纽汇总了核心文档，介绍 OpenClaw 如何在 localhost、局域网（LAN）和 tailnet 上连接、配对并保护设备安全。

<div id="core-model">
  ## 核心模型
</div>

* [Gateway 架构](/zh/concepts/architecture)
* [Gateway 协议](/zh/gateway/protocol)
* [Gateway 运行手册](/zh/gateway)
* [Web 界面与绑定模式](/zh/web)

<div id="pairing-identity">
  ## 配对与身份
</div>

* [配对概览（DM 与节点）](/zh/start/pairing)
* [Gateway 所属节点配对](/zh/gateway/pairing)
* [Devices CLI（配对与令牌轮换）](/zh/cli/devices)
* [Pairing CLI（DM 审批）](/zh/cli/pairing)

本地信任：

* 本地连接（回环地址或 Gateway 主机自身的 tailnet 地址）可以自动通过配对审批，
  以保证同一主机上的用户体验（UX）顺畅。
* 非本地 tailnet/LAN 客户端仍然需要显式配对审批。

<div id="discovery-transports">
  ## 发现与传输
</div>

* [发现与传输](/zh/gateway/discovery)
* [Bonjour / mDNS](/zh/gateway/bonjour)
* [远程访问（SSH）](/zh/gateway/remote)
* [Tailscale](/zh/gateway/tailscale)

<div id="nodes-transports">
  ## 节点与传输通道
</div>

* [节点概览](/zh/nodes)
* [Bridge 协议（旧版节点）](/zh/gateway/bridge-protocol)
* [节点运维手册：iOS](/zh/platforms/ios)
* [节点运维手册：Android](/zh/platforms/android)

<div id="security">
  ## 安全
</div>

* [安全概览](/zh/gateway/security)
* [Gateway 配置参考](/zh/gateway/configuration)
* [故障排查](/zh/gateway/troubleshooting)
* [Doctor 命令](/zh/gateway/doctor)