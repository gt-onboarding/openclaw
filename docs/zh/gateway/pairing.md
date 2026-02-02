---
title: 配对
summary: "由 Gateway 主导的节点配对（选项 B），适用于 iOS 和其他远程节点"
read_when:
  - 在没有 macOS UI 的情况下实现节点配对审批功能
  - 添加用于审批远程节点的 CLI 流程
  - 扩展 Gateway 协议以支持节点管理
---

<div id="gateway-owned-pairing-option-b">
  # Gateway 主导的配对（选项 B）
</div>

在 Gateway 主导的配对模式下，**Gateway** 是判定哪些节点
被允许加入的唯一权威来源。UI（macOS 应用、后续的客户端）只是用来
批准或拒绝待处理请求的前端。

**重要说明：**WS 节点在 `connect` 期间使用 **设备配对**（角色 `node`）。
`node.pair.*` 是一个单独的配对存储，并且**不会**对 WS 握手起到准入控制作用。
只有显式调用 `node.pair.*` 的客户端才会使用这一流程。

<div id="concepts">
  ## 概念
</div>

- **待处理请求**：节点发起加入请求，等待批准。
- **已配对节点**：已获批准并已签发认证令牌的节点。
- **传输**：Gateway 的 WS 端点只负责转发请求，不决定节点成员资格。（旧版 TCP bridge 支持已弃用/移除。）

<div id="how-pairing-works">
  ## 配对的工作原理
</div>

1. 某个节点通过 Gateway 的 WS 连接并发起配对请求。
2. Gateway 保留一个**待处理请求**并发出 `node.pair.requested` 事件。
3. 你通过 CLI 或 UI 批准或拒绝该请求。
4. 批准后，Gateway 签发一个**新令牌**（重新配对时会轮换令牌）。
5. 节点使用该令牌重新连接，此时即视为已“配对”。

待处理请求会在 **5 分钟**后自动过期。

<div id="cli-workflow-headless-friendly">
  ## CLI 工作流程（适合无头环境）
</div>

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
openclaw nodes reject <requestId>
openclaw nodes status
openclaw nodes rename --node <id|name|ip> --name "Living Room iPad"
```

`nodes status` 显示已配对/已连接节点及其可用能力。


<div id="api-surface-gateway-protocol">
  ## API 接口（Gateway 协议）
</div>

事件：

- `node.pair.requested` —— 当创建新的待处理请求时触发。
- `node.pair.resolved` —— 当请求被批准/拒绝/过期时触发。

方法：

- `node.pair.request` —— 创建或复用一个待处理请求。
- `node.pair.list` —— 列出待处理和已配对的节点。
- `node.pair.approve` —— 批准一个待处理请求（签发 token）。
- `node.pair.reject` —— 拒绝一个待处理请求。
- `node.pair.verify` —— 验证 `{ nodeId, token }`。

注意事项：

- `node.pair.request` 对每个节点都是幂等的：重复调用会返回相同的待处理请求。
- 批准操作**始终**生成新的 token；`node.pair.request` 永远不会返回 token。
- 请求可以包含 `silent: true`，作为自动批准流程的提示信息。

<div id="auto-approval-macos-app">
  ## 自动批准（macOS 应用）
</div>

macOS 应用可以在以下情况下尝试执行**静默批准**：

- 请求被标记为 `silent`，并且
- 应用可以使用同一用户验证到 Gateway 主机的 SSH 连接。

如果静默批准失败，则会退回到正常的 “Approve/Reject” 提示。

<div id="storage-local-private">
  ## 存储（本地、私有）
</div>

配对状态存储在 Gateway 状态目录下（默认 `~/.openclaw`）：

- `~/.openclaw/nodes/paired.json`
- `~/.openclaw/nodes/pending.json`

如果你重写了 `OPENCLAW_STATE_DIR`，`nodes/` 文件夹也会随之一起移动。

安全注意事项：

- Token 属于机密信息；将 `paired.json` 视为敏感文件。
- 轮换（更换）一个 token 需要重新审批（或删除该节点条目）。

<div id="transport-behavior">
  ## 传输行为
</div>

- 该传输是**无状态**的；不会存储成员关系。
- 如果 Gateway 离线或配对被禁用，节点无法进行配对。
- 如果 Gateway 处于远程模式，配对仍然会使用远程 Gateway 的存储。