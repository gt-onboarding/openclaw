---
title: 在线状态（Presence）
summary: "OpenClaw 在线状态条目是如何产生、合并和显示的"
read_when:
  - 调试 Instances 选项卡
  - 排查重复或陈旧的实例行
  - 更改 Gateway 的 WS 连接或系统事件信标设置
---

<div id="presence">
  # 在线状态
</div>

OpenClaw 的 “presence” 是一个轻量级的、尽力而为的状态视图，用于展示：

- **Gateway** 本身，及
- **连接到 Gateway 的客户端**（macOS 应用、WebChat、CLI 等）

Presence 主要用于渲染 macOS 应用的 **Instances** 选项卡，并帮助运维人员快速了解当前状态。

<div id="presence-fields-what-shows-up">
  ## Presence 字段（会显示的内容）
</div>

Presence 条目是带有如下字段的结构化对象：

- `instanceId`（可选但强烈推荐）：稳定的客户端标识（通常为 `connect.client.instanceId`）
- `host`：便于识别的主机名
- `ip`：尽可能准确的 IP 地址
- `version`：客户端版本字符串
- `deviceFamily` / `modelIdentifier`：硬件相关信息提示
- `mode`：`ui`、`webchat`、`cli`、`backend`、`probe`、`test`、`node` 等
- `lastInputSeconds`：“自上次用户输入以来经过的秒数”（如果已知）
- `reason`：`self`、`connect`、`node-connected`、`periodic` 等
- `ts`：最后一次更新的时间戳（自 Unix 纪元（epoch）起经过的毫秒数）

<div id="producers-where-presence-comes-from">
  ## 生产者（Presence 的来源）
</div>

Presence 条目来自多个来源，并会被**合并**。

<div id="1-gateway-self-entry">
  ### 1) Gateway 自身条目
</div>

Gateway 在启动时始终预置一个“self”条目，这样即便尚无任何客户端连接，各个 UI 也会显示该 Gateway 主机。

<div id="2-websocket-connect">
  ### 2) WebSocket 连接
</div>

每个 WS 客户端都会先发送一次 `connect` 请求。握手成功后，Gateway 会为该连接插入或更新一条在线状态记录。

<div id="why-oneoff-cli-commands-dont-show-up">
  #### 为什么一次性 CLI 命令不会显示出来
</div>

CLI 通常只会为短暂的一次性命令建立连接。为了避免刷屏 Instances 列表，`client.mode === "cli"` **不会**被记录为 presence 条目。

<div id="3-system-event-beacons">
  ### 3) `system-event` 信标
</div>

客户端可以通过 `system-event` 方法发送更丰富的周期性信标消息。macOS 应用会使用它来上报主机名、IP 地址，以及 `lastInputSeconds`。

<div id="4-node-connects-role-node">
  ### 4) 节点连接（角色：node）
</div>

当一个节点以 `role: node` 通过 Gateway 的 WebSocket 连接时，Gateway
会为该节点插入或更新一条在线状态记录（与其他 WS 客户端的流程相同）。

<div id="merge-dedupe-rules-why-instanceid-matters">
  ## 合并与去重规则（为什么 `instanceId` 很重要）
</div>

Presence 条目存储在单个内存映射表中：

- 条目的键是一个 **presence 键**。
- 首选键是一个稳定的 `instanceId`（来自 `connect.client.instanceId`），它在重启后仍然保持不变。
- 键不区分大小写。

如果客户端在没有稳定 `instanceId` 的情况下重新连接，它可能会显示为一条**重复**记录。

<div id="ttl-and-bounded-size">
  ## TTL 和有界大小
</div>

Presence 被刻意设计为短暂存在：

- **TTL：** 超过 5 分钟的条目会被清理
- **最大条目数：** 200（优先丢弃最早的）

这样可以保持列表新鲜，并避免内存无界增长。

<div id="remotetunnel-caveat-loopback-ips">
  ## 远程/隧道注意事项（回环 IP）
</div>

当客户端通过 SSH 隧道 / 本地端口转发进行连接时，Gateway 看到的远程地址可能是 `127.0.0.1`。为避免覆盖客户端正确上报的真实 IP，会忽略回环远程地址。

<div id="consumers">
  ## 使用方
</div>

<div id="macos-instances-tab">
  ### macOS Instances 标签页
</div>

macOS 应用会展示 `system-presence` 的输出，并根据最近一次更新的时间间隔，显示一个小状态指示器（Active/Idle/Stale）。

<div id="debugging-tips">
  ## 调试建议
</div>

- 若要查看原始列表，请在 Gateway 上调用 `system-presence`。
- 如果你看到重复项：
  - 确认客户端在握手过程中发送了稳定的 `client.instanceId`
  - 确认周期性 beacon 信号使用的是同一个 `instanceId`
  - 检查基于连接派生的条目是否缺少 `instanceId`（这种情况下出现重复是正常的）