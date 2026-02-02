---
title: Clawnet
summary: "Clawnet 重构：统一网络协议、角色、认证、审批和身份"
read_when:
  - 在为节点和运维/操作员客户端规划统一网络协议时
  - 在重新设计跨设备的审批、配对、TLS 和在线状态机制时
---

<div id="clawnet-refactor-protocol-auth-unification">
  # Clawnet 重构（协议与认证统一）
</div>

<div id="hi">
  ## 嗨
</div>

嗨 Peter——这个方向很好；既简化了用户体验，又增强了安全性。

<div id="purpose">
  ## 目的
</div>

一份统一且严谨的文档，用于说明：

- 当前状态：协议、流程、信任边界。
- 痛点：审批、多跳路由、UI 重复。
- 拟议的新状态：单一协议、限 scope 的角色、统一认证/配对、TLS 证书固定（pinning）。
- 身份模型：稳定 ID + 可爱的短 slug 标识。
- 迁移方案、风险、未决问题。

<div id="goals-from-discussion">
  ## 目标（来源于讨论）
</div>

- 为所有客户端（macOS 应用、CLI、iOS、Android、无头节点）提供统一协议。
- 所有网络参与方都必须通过认证并完成配对。
- 角色边界清晰：节点 vs 操作员。
- 集中审批应路由到用户当前所在的端。
- 所有远程流量使用 TLS 加密，并可选启用证书固定（pinning）。
- 尽量减少代码重复。
- 单台机器只应出现一次（不应在 UI/节点中重复列出）。

<div id="nongoals-explicit">
  ## 非目标（明确说明）
</div>

- 移除能力隔离（仍然需要最小特权原则）。
- 在没有 scope 检查的情况下暴露完整 Gateway 控制平面。
- 让认证依赖人工标签（slug 仍不具备安全意义）。

---

<div id="current-state-asis">
  # 当前状态（现有情况）
</div>

<div id="two-protocols">
  ## 两种协议
</div>

<div id="1-gateway-websocket-control-plane">
  ### 1) Gateway WebSocket（控制平面）
</div>

- 完整 API 接口范围：config、channels、models、sessions、agent runs、logs、nodes 等。
- 默认绑定：本地回环（loopback）。通过 SSH/Tailscale 进行远程访问。
- 鉴权：通过 `connect` 使用令牌/密码。
- 不进行 TLS 绑定（pinning）（依赖本地回环/隧道）。
- 代码：
  - `src/gateway/server/ws-connection/message-handler.ts`
  - `src/gateway/client.ts`
  - `docs/gateway/protocol.md`

<div id="2-bridge-node-transport">
  ### 2) Bridge（节点传输）
</div>

- 收窄允许列表暴露面，节点身份 + 配对。
- JSONL over TCP；可选 TLS + 证书指纹绑定。
- TLS 在发现用的 TXT 记录中公布指纹。
- 代码：
  - `src/infra/bridge/server/connection.ts`
  - `src/gateway/server-bridge.ts`
  - `src/node-host/bridge-client.ts`
  - `docs/gateway/bridge-protocol.md`

<div id="control-plane-clients-today">
  ## 当前控制平面客户端
</div>

- CLI → 通过 `callGateway` (`src/gateway/call.ts`) 连接 Gateway 的 WS。
- macOS app UI → 通过 `GatewayConnection` 连接 Gateway 的 WS。
- Web Control UI → 连接 Gateway 的 WS。
- ACP → 连接 Gateway 的 WS。
- 浏览器控制端使用其自身的 HTTP 控制服务器。

<div id="nodes-today">
  ## 当前的节点情况
</div>

- 以节点模式运行的 macOS 应用会连接到 Gateway bridge（`MacNodeBridgeSession`）。
- iOS/Android 应用会连接到 Gateway bridge。
- 配对信息 + 每个节点的 token 存储在 Gateway 上。

<div id="current-approval-flow-exec">
  ## 当前审批流程（执行）
</div>

- Agent 代理通过 Gateway 调用 `system.run`。
- Gateway 通过 bridge 调用节点。
- 节点运行时决定是否批准。
- 当节点 == macOS app 时，由 macOS app 显示 UI 提示。
- 节点将 `invoke-res` 返回给 Gateway。
- 多跳流程，UI 绑定到节点所在主机。

<div id="presence-identity-today">
  ## 当前的在线状态与身份
</div>

- 来自 WS 客户端的 Gateway 在线状态记录。
- 来自 bridge 的节点在线状态记录。
- mac 应用在同一台机器上可能显示两个记录（UI + 节点）。
- 节点身份存储在配对存储中；UI 身份单独存储。

---

<div id="problems-pain-points">
  # 问题 / 痛点
</div>

- 需要维护两套协议栈（WS + Bridge）。
- 远程节点上的授权：确认提示出现在节点主机上，而不是用户所在的位置。
- 仅 Bridge 支持 TLS 证书固定；WS 依赖 SSH/Tailscale。
- 身份重复：同一台机器显示为多个实例。
- 角色模糊：UI、节点和 CLI 的能力边界不清晰。

---

<div id="proposed-new-state-clawnet">
  # 提议的新状态（Clawnet）
</div>

<div id="one-protocol-two-roles">
  ## 单一协议，两个角色
</div>

单一 WS 协议，包含角色 + scope。

- **角色：节点（node）**（功能宿主）
- **角色：operator**（控制平面）
- 可选的 **scope**（用于 operator）：
  - `operator.read`（状态 + 查看）
  - `operator.write`（智能体运行、发送）
  - `operator.admin`（配置、通道、模型）

<div id="role-behaviors">
  ### 角色行为
</div>

**节点（Node）**

- 可以注册能力（`caps`、`commands`、权限）。
- 可以接收 `invoke` 命令（`system.run`、`camera.*`、`canvas.*`、`screen.record` 等）。
- 可以发送事件：`voice.transcript`、`agent.request`、`chat.subscribe`。
- 不能调用配置/模型/通道/会话/智能体的控制平面 api。

**Operator**

- 拥有完整的控制平面 api，受 scope 约束。
- 接收所有审批。
- 不直接执行操作系统层面的操作，而是将请求路由到节点。

<div id="key-rule">
  ### 关键规则
</div>

角色是按连接划分的，而不是按设备划分的。同一设备可以分别以两种角色建立各自的连接。

<div id="unified-authentication-pairing">
  # 统一认证与配对
</div>

<div id="client-identity">
  ## 客户端身份
</div>

每个客户端会提供：

- `deviceId`（稳定值，由设备密钥派生而来）。
- `displayName`（供人阅读的名称）。
- `role` + `scope` + `caps` + `commands`。

<div id="pairing-flow-unified">
  ## 配对流程（统一）
</div>

- 客户端在未认证状态下发起连接。
- Gateway 为该 `deviceId` 创建一个**配对请求**。
- 操作员收到提示；选择批准或拒绝。
- Gateway 签发绑定到以下内容的凭证：
  - 设备公钥
  - 角色
  - scope
  - 能力/命令
- 客户端持久化保存令牌，并以已认证状态重新连接。

<div id="devicebound-auth-avoid-bearer-token-replay">
  ## 与设备绑定的认证（避免 bearer token 重放）
</div>

首选：设备密钥对。

- 设备只生成一次密钥对。
- `deviceId = fingerprint(publicKey)`。
- Gateway 发送随机数（nonce）；设备对其签名；Gateway 验证签名。
- 令牌是针对公钥签发的（基于持有证明），而不是针对某个字符串。

替代方案：

- mTLS（客户端证书）：最强，但运维复杂度更高。
- 仅在临时过渡阶段使用短生命周期的 bearer token（频繁轮换并尽早吊销）。

<div id="silent-approval-ssh-heuristic">
  ## 静默批准（SSH 启发式）
</div>

需要对其进行精确定义，以避免成为薄弱环节。优先选择以下其一：

- **仅限本地**：当客户端通过 loopback/Unix 套接字连接时自动配对。
- **通过 SSH 质询**：Gateway 下发一次性随机数（nonce）；客户端通过 SSH 获取该随机数来证明身份。
- **物理在场时间窗**：在 Gateway 主机 UI 上进行一次本地批准后，在短时间窗口内（例如 10 分钟）允许自动配对。

始终记录并保存所有自动批准操作。

---

<div id="tls-everywhere-dev-prod">
  # 各环境全面启用 TLS（开发 + 生产）
</div>

<div id="reuse-existing-bridge-tls">
  ## 复用现有 bridge TLS
</div>

使用当前 TLS 运行时 + 指纹锁定（pinning）：

- `src/infra/bridge/server/tls.ts`
- `src/node-host/bridge-client.ts` 中的指纹验证逻辑

<div id="apply-to-ws">
  ## 适用于 WS
</div>

- WS 服务器支持使用相同证书/密钥和指纹的 TLS。
- WS 客户端可以绑定指纹（可选）。
- Discovery 会为所有端点发布 TLS 和指纹信息。
  - Discovery 仅作为定位提示；绝不是信任锚点。

<div id="why">
  ## 为什么
</div>

- 降低在机密性上的对 SSH/Tailscale 的依赖。
- 使远程移动连接在默认情况下即是安全的。

---

<div id="approvals-redesign-centralized">
  # 审批系统重新设计（集中化）
</div>

<div id="current">
  ## 现状
</div>

审批在节点主机上完成（macOS 应用的节点运行时）。提示会出现在节点运行的设备上。

<div id="proposed">
  ## 提案
</div>

审批由 **Gateway** 统一托管，UI 下发到运维人员的客户端。

<div id="new-flow">
  ### 新流程
</div>

1) Gateway 接收到 `system.run` 意图（来自智能体）。
2) Gateway 创建审批记录：`approval.requested`。
3) 运维人员的 UI 显示提示。
4) 将审批决定发送给 Gateway：`approval.resolve`。
5) 如果获批，Gateway 调用节点命令。
6) 节点执行并返回 `invoke-res`。

<div id="approval-semantics-hardening">
  ### 审批语义（强化）
</div>

- 向所有 operator 广播；只有当前活动的 UI 显示模态窗口（其他只收到 toast 通知）。
- 以首次决议为准；Gateway 会将后续的决议视为已处理并予以拒绝。
- 默认超时：在 N 秒后（例如 60s）自动拒绝，并记录原因。
- 进行决议需要具备 `operator.approvals` scope。

<div id="benefits">
  ## 优势
</div>

- 交互入口出现在用户所在的设备上（Mac / 手机）。
- 远程节点使用统一的授权/审批流程。
- 节点运行时保持无界面（headless），不依赖 UI。

---

<div id="role-clarity-examples">
  # 角色界定示例
</div>

<div id="iphone-app">
  ## iPhone 应用
</div>

- **节点角色** 用于：麦克风、相机、语音聊天、位置、按键说话（对讲）。
- 可选的 **operator.read**，用于状态和聊天视图。
- 可选的 **operator.write/admin**，仅在显式启用时使用。

<div id="macos-app">
  ## macOS 应用
</div>

- 默认为 Operator 角色（Control UI）。
- 启用「Mac node」后为节点角色（system.run、screen、camera）。
- 两个连接使用相同的 deviceId → 在 UI 中合并为同一条目。

<div id="cli">
  ## CLI
</div>

- 始终使用 Operator 角色。
- scope 由子命令确定：
  - `status`, `logs` → read
  - `agent`, `message` → write
  - `config`, `channels` → admin
  - approvals + pairing → `operator.approvals` / `operator.pairing`

---

<div id="identity-slugs">
  # 标识与 slug
</div>

<div id="stable-id">
  ## 稳定 ID
</div>

认证所必需；永不变化。
优先使用：

- 密钥对指纹（公钥哈希）。

<div id="cute-slug-lobsterthemed">
  ## 可爱 slug（龙虾主题）
</div>

仅供人类阅读的标签。

- 示例：`scarlet-claw`、`saltwave`、`mantis-pinch`。
- 存储在 Gateway 注册表中，可编辑。
- 冲突处理：追加 `-2`、`-3`。

<div id="ui-grouping">
  ## UI 分组
</div>

相同的 `deviceId` 在不同角色下 → 单行“实例”记录：

- 徽标：`operator`、`node`。
- 显示能力与最后一次在线时间。

---

<div id="migration-strategy">
  # 迁移策略
</div>

<div id="phase-0-document-align">
  ## 阶段 0：文档化与对齐
</div>

- 发布此文档。
- 梳理所有协议调用和审批流程。

<div id="phase-1-add-rolesscopes-to-ws">
  ## 阶段 1：在 WS 中添加角色和 scope
</div>

- 扩展 `connect` 参数以包含 `role`、`scope`、`deviceId`。
- 为节点角色添加基于允许列表的访问控制。

<div id="phase-2-bridge-compatibility">
  ## 阶段 2：Bridge 兼容性
</div>

- 保持 Bridge 持续运行。
- 并行添加 WS 节点支持。
- 通过配置开关控制相关功能。

<div id="phase-3-central-approvals">
  ## 阶段 3：集中审批
</div>

- 在 WS 中添加审批请求和审批完成事件。
- 更新 macOS 应用 UI 以发起提示并处理响应。
- 节点运行时停止直接向 UI 发起提示。

<div id="phase-4-tls-unification">
  ## 第 4 阶段：TLS 统一
</div>

- 使用 bridge 的 TLS 运行时为 WS 添加 TLS 配置。
- 在客户端启用证书固定（pinning）。

<div id="phase-5-deprecate-bridge">
  ## 第 5 阶段：弃用 bridge
</div>

- 将 iOS/Android/macOS 节点迁移到 WS。
- 保留 bridge 作为回退方案；在稳定后移除。

<div id="phase-6-devicebound-auth">
  ## 阶段 6：设备绑定认证
</div>

- 要求所有非本地连接使用基于密钥的身份认证。
- 添加用于撤销和轮换的 UI。

---

<div id="security-notes">
  # 安全注意事项
</div>

- 在 Gateway 边界强制执行角色和允许列表策略。
- 未授予 operator scope 时，任何客户端都不会获得“完整”的 API 访问权限。
- *所有* 连接都需要配对。
- TLS + 证书锁定可降低移动端遭受中间人攻击的风险。
- SSH 静默批准只是为了方便；仍会被记录并且可撤销。
- 发现机制（discovery）绝不能作为信任锚点。
- 能力声明会按平台/类型对照服务器允许列表进行验证。

<div id="streaming-large-payloads-node-media">
  # 流式传输与大负载（节点媒体）
</div>

WS 控制平面对小体量消息没问题，但节点还需要处理：

- 摄像头短片
- 屏幕录制
- 音频流

选项：

1) WS 二进制帧 + 分块 + 背压规则。
2) 独立的流式传输端点（仍然使用 TLS + 认证）。
3) 对媒体数据较多的命令延长保留 bridge，最后再迁移。

在实现前先选定一种方案，以避免后续偏移。

<div id="capability-command-policy">
  # 能力与命令策略
</div>

- 节点上报的能力/命令被视为**声明**。
- Gateway 在各个平台上强制执行允许列表。
- 任何新命令都需要运维管理员批准或明确更新允许列表。
- 审计变更并记录时间戳。

<div id="audit-rate-limiting">
  # 审计与限流
</div>

- 记录：配对请求、批准/拒绝、令牌发放/轮换/撤销。
- 对配对垃圾请求和配对确认提示进行限流。

<div id="protocol-hygiene">
  # 协议规范
</div>

- 明确的协议版本和错误代码。
- 重连规则和心跳策略。
- 在线状态 TTL 和最近在线语义定义。

---

<div id="open-questions">
  # 未决问题
</div>

1) 单一设备同时运行两种角色：令牌模型
   - 建议按角色分别使用独立令牌（节点 vs operator）。
   - 共享同一 deviceId，使用不同 scope，撤销控制更清晰。

2) operator scope 的粒度
   - read/write/admin + approvals + 配对（最小可行集）。
   - 日后再考虑按功能细分的 scope。

3) 令牌轮换与撤销的 UX
   - 在角色变更时自动轮换。
   - 在 UI 中按 deviceId + 角色进行撤销。

4) 发现（Discovery）
   - 扩展当前 Bonjour TXT，包含 WS TLS 指纹 + 角色提示。
   - 仅将其作为定位提示使用。

5) 跨网络审批
   - 广播到所有 operator 客户端；活动中的 UI 显示模态对话框。
   - 先响应者生效；Gateway 强制保证原子性。

---

<div id="summary-tldr">
  # 摘要（TL;DR）
</div>

- 现状：WS 控制平面 + Bridge 节点传输。
- 痛点：审批 / 授权繁琐 + 重复配置多 + 维护两套技术栈。
- 提案：一个带有明确角色和 scope 的统一 WS 协议，统一的配对流程 + TLS 证书固定，由 Gateway 托管审批，稳定的设备 ID + 可爱的 slug 短标识。
- 结果：更简洁的用户体验、更强的安全性、更少的重复、更好的移动端路由。