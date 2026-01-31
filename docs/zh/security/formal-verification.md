---
title: 形式化验证（安全模型）
summary: 针对 OpenClaw 最高风险路径的经机器验证安全模型。
permalink: /security/formal-verification/
---

<div id="formal-verification-security-models">
  # 形式化验证（安全模型）
</div>

本页汇总并维护 OpenClaw 的**形式化安全模型**（当前为 TLA+/TLC，后续视需要扩展）。

> 注意：部分较早的链接可能仍指向旧的项目名称。

**目标（“北极星”）：** 在明确前提下，给出一个经机器检查的论证，证明 OpenClaw 会强制执行其预期的安全策略（授权、会话隔离、工具访问门控，以及错误配置防护能力）。

**目前的状态：** 一个可执行、以攻击者为驱动的**安全回归套件**：

- 每条安全断言都对应一个在有限状态空间上的可运行模型检查。
- 许多断言都有配对的**反面模型**，会针对现实中的某类缺陷生成反例执行轨迹。

**目前还不是：** “OpenClaw 在所有方面都是安全的”的证明，或对完整 TypeScript 实现正确性的证明。

<div id="where-the-models-live">
  ## 模型所在位置
</div>

模型保存在一个单独的代码仓库中：[vignesh07/openclaw-formal-models](https://github.com/vignesh07/openclaw-formal-models)。

<div id="important-caveats">
  ## 重要注意事项
</div>

- 这些是**模型**，而不是完整的 TypeScript 实现。模型与代码之间可能存在偏差。
- 结果受限于 TLC 探索到的状态空间；“绿色”并不意味着在建模假设和边界之外仍然安全。
- 一些论断依赖于显式的环境假设（例如，正确的部署、正确的配置输入）。

<div id="reproducing-results">
  ## 复现结果
</div>

目前，复现结果的方式是本地克隆 models 仓库并运行 TLC（见下文）。未来的迭代可以提供：

* 在 CI 中运行模型并生成公开工件（反例跟踪、运行日志）
* 一个托管的“运行此模型”工作流，用于小规模、有界检查

开始使用：

```bash
git clone https://github.com/vignesh07/openclaw-formal-models
cd openclaw-formal-models

# 需要 Java 11+（TLC 在 JVM 上运行）。
# 该仓库内置固定版本的 `tla2tools.jar`（TLA+ 工具）并提供 `bin/tlc` + Make 目标。

make <target>
```


<div id="gateway-exposure-and-open-gateway-misconfiguration">
  ### Gateway 暴露与对外开放 Gateway 的错误配置
</div>

**断言：** 在没有认证的情况下绑定到非本地回环地址，可能使远程入侵成为可能 / 提高暴露面；根据模型假设，令牌/密码可以阻止未授权攻击者。

- 绿色运行：
  - `make gateway-exposure-v2`
  - `make gateway-exposure-v2-protected`
- 红色（预期）：
  - `make gateway-exposure-v2-negative`

另见：models 仓库中的 `docs/gateway-exposure-matrix.md`。

<div id="nodesrun-pipeline-highest-risk-capability">
  ### Nodes.run 流水线（最高风险能力）
</div>

**声明：**`nodes.run` 需要 (a) 节点命令允许列表加上已声明的命令，以及 (b) 在相应机制已配置时需要实时审批；审批会被令牌化处理，以防止重放（在模型中）。

- 绿色运行：
  - `make nodes-pipeline`
  - `make approvals-token`
- 红色（预期）：
  - `make nodes-pipeline-negative`
  - `make approvals-token-negative`

<div id="pairing-store-dm-gating">
  ### 配对存储（DM 门控）
</div>

**命题：** 配对请求遵守 TTL 和待处理请求上限的限制。

- 绿色运行（应通过）：
  - `make pairing`
  - `make pairing-cap`
- 红色运行（预期失败）：
  - `make pairing-negative`
  - `make pairing-cap-negative`

<div id="ingress-gating-mentions-control-command-bypass">
  ### 入口 gating（@提及 + 控制命令绕过）
</div>

**断言：** 在要求使用 @提及 的群组场景中，未授权的「控制命令」无法绕过提及 gating。

- 绿色：
  - `make ingress-gating`
- 红色（预期）：
  - `make ingress-gating-negative`

<div id="routingsession-key-isolation">
  ### 路由/会话键隔离
</div>

**断言：** 来自不同对端的私信不会被合并到同一个会话中，除非显式进行关联或配置。

- 绿色：
  - `make routing-isolation`
- 红色（预期失败）：
  - `make routing-isolation-negative`

<div id="v1-additional-bounded-models-concurrency-retries-trace-correctness">
  ## v1++：附加有界模型（并发、重试、执行轨迹正确性）
</div>

这些是后续模型，用于围绕真实世界中的故障模式（非原子更新、重试和消息扇出）进行更严格的精确建模。

<div id="pairing-store-concurrency-idempotency">
  ### 配对存储的并发性 / 幂等性
</div>

**论断：** 即使在交错执行的情况下，配对存储也应该强制执行 `MaxPending` 和幂等性（即“检查再写入”必须是原子/加锁的；刷新操作不应产生重复记录）。

含义：

- 在并发请求下，某个 channel 的挂起数量绝不能超过其 `MaxPending`。
- 对同一 `(channel, sender)` 的重复请求/刷新不应产生重复的活动挂起记录行。

- 预期为绿色（通过）的运行：
  - `make pairing-race`（原子/加锁的上限检查）
  - `make pairing-idempotency`
  - `make pairing-refresh`
  - `make pairing-refresh-race`
- 预期为红色（失败）的运行：
  - `make pairing-race-negative`（非原子的 begin/commit 上限竞态）
  - `make pairing-idempotency-negative`
  - `make pairing-refresh-negative`
  - `make pairing-refresh-race-negative`

<div id="ingress-trace-correlation-idempotency">
  ### Ingress 追踪关联 / 幂等性
</div>

**主张：** 入口摄取在扇出时应保持追踪关联，并且在提供方重试场景下保持幂等性。

含义：

- 当一个外部事件被扇出为多个内部消息时，每一部分都保留相同的 trace/event 标识。
- 重试不会导致重复处理。
- 如果缺少提供方事件 ID，去重逻辑会回退到一个安全的键值（例如 trace ID），以避免误丢弃本应视为不同的事件。

- 绿色：
  - `make ingress-trace`
  - `make ingress-trace2`
  - `make ingress-idempotency`
  - `make ingress-dedupe-fallback`
- 红色（预期）：
  - `make ingress-trace-negative`
  - `make ingress-trace2-negative`
  - `make ingress-idempotency-negative`
  - `make ingress-dedupe-fallback-negative`

<div id="routing-dmscope-precedence-identitylinks">
  ### Routing dmScope precedence + identityLinks
</div>

**主张：** 路由在默认情况下必须保持 DM 会话相互隔离，只有在显式配置时（通道优先级 + identityLinks）才可以合并会话。

含义：

- 通道级的 dmScope 覆盖配置必须优先于全局默认值。
- identityLinks 只能在显式链接的分组内合并，而不能跨不相关的对端进行合并。

- 绿色：
  - `make routing-precedence`
  - `make routing-identitylinks`
- 红色（预期失败）：
  - `make routing-precedence-negative`
  - `make routing-identitylinks-negative`