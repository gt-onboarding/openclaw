---
title: 形式化验证（安全模型）
summary: 覆盖 OpenClaw 最高风险路径的机器验证安全模型。
permalink: /security/formal-verification/
---

<div id="formal-verification-security-models">
  # 形式化验证（安全模型）
</div>

本页用于跟踪 OpenClaw 的**形式化安全模型**（目前为 TLA+/TLC，后续按需增加）。

> 注意：部分旧链接可能仍然指向之前的项目名称。

**目标（长远方向）：** 在明确给出前提假设的情况下，提供一份经机器检查的论证，证明 OpenClaw 会强制执行其预期的安全策略（授权、会话隔离、工具访问控制，以及错误配置安全性）。

**这在当前是什么：** 一个可执行的、以攻击者视角驱动的**安全回归测试套件**：

- 每条安全声明都有一个在有限状态空间上的可运行模型检查。
- 许多声明都有一个对应的**反例模型**，可以针对一个现实的缺陷类别生成反例执行轨迹。

**这目前还不是什么：** 还称不上是一个“OpenClaw 在各方面都是安全的”的证明，也不是对完整 TypeScript 实现正确性的证明。

<div id="where-the-models-live">
  ## 模型存放位置
</div>

模型维护在一个单独的仓库中：[vignesh07/openclaw-formal-models](https://github.com/vignesh07/openclaw-formal-models)。

<div id="important-caveats">
  ## 重要注意事项
</div>

- 这些是**模型**，而不是完整的 TypeScript 实现。模型与代码之间可能会存在偏差。
- 结果受限于 TLC 所探索的状态空间；出现“绿色”并不意味着在模型假设和边界之外同样安全。
- 部分断言依赖于显式的环境假设（例如，正确的部署、正确的配置输入）。

<div id="reproducing-results">
  ## 复现结果
</div>

目前，要复现结果，你需要在本地克隆模型仓库并运行 TLC（见下文）。未来的迭代可以提供：

* 由 CI 运行模型并公开产物（反例轨迹、运行日志）
* 面向小规模、有界检查的托管式“运行此模型”工作流

开始使用：

```bash
git clone https://github.com/vignesh07/openclaw-formal-models
cd openclaw-formal-models

# 需要 Java 11+（TLC 在 JVM 上运行）。
# 该仓库内置固定版本的 `tla2tools.jar`（TLA+ 工具）并提供 `bin/tlc` + Make 目标。

make <target>
```


<div id="gateway-exposure-and-open-gateway-misconfiguration">
  ### Gateway 暴露与开放 Gateway 的错误配置
</div>

**结论：** 在没有认证的情况下将绑定地址扩展到本地回环之外，会使远程入侵成为可能 / 增加暴露面；根据模型假设，引入 token/password 认证可以阻止未授权攻击者。

- 绿色运行：
  - `make gateway-exposure-v2`
  - `make gateway-exposure-v2-protected`
- 红色（预期）：
  - `make gateway-exposure-v2-negative`

另见 models 仓库中的 `docs/gateway-exposure-matrix.md`。

<div id="nodesrun-pipeline-highest-risk-capability">
  ### Nodes.run 流水线（最高风险能力）
</div>

**说明：** `nodes.run` 需要：(a) 节点命令允许列表加上已声明的命令，以及 (b) 在启用相关配置时进行实时审批；审批会在模型中被令牌化处理以防止重放。

- 绿色用例：
  - `make nodes-pipeline`
  - `make approvals-token`
- 红色用例（预期）：
  - `make nodes-pipeline-negative`
  - `make approvals-token-negative`

<div id="pairing-store-dm-gating">
  ### 配对存储（DM 门控）
</div>

**断言：** 配对请求遵循 TTL 和待处理请求上限。

- 绿色（预期通过）：
  - `make pairing`
  - `make pairing-cap`
- 红色（预期失败）：
  - `make pairing-negative`
  - `make pairing-cap-negative`

<div id="ingress-gating-mentions-control-command-bypass">
  ### 入口门控（提及 + 控制命令绕过）
</div>

**声明：** 在需要提及的群组场景中，未授权的“控制命令”无法绕过提及门控。

- 绿色：
  - `make ingress-gating`
- 红色（预期）：
  - `make ingress-gating-negative`

<div id="routingsession-key-isolation">
  ### 路由/会话键隔离
</div>

**断言：** 来自不同对端的私信不会被归并到同一个会话中，除非经过显式关联或配置。

- 绿色：
  - `make routing-isolation`
- 红色（预期）：
  - `make routing-isolation-negative`

<div id="v1-additional-bounded-models-concurrency-retries-trace-correctness">
  ## v1++：附加有界模型（并发、重试、执行轨迹正确性）
</div>

这些是后续模型，用于更精确地刻画真实世界中的故障模式（非原子更新、重试和消息扇出）。

<div id="pairing-store-concurrency-idempotency">
  ### 配对存储的并发性 / 幂等性
</div>

**主张：** 即使在交错执行的情况下，配对存储也必须强制执行 `MaxPending` 限制并保持幂等性（即“检查然后写入”的操作必须是原子/加锁的；刷新操作不应创建重复的待处理记录）。

含义：

- 在并发请求下，某个 channel 的 `MaxPending` 不得被超过。
- 对同一 `(channel, sender)` 的重复请求/刷新不应产生重复的活动待处理记录行。

- 绿色（预期通过）的运行：
  - `make pairing-race`（原子/加锁的上限检查）
  - `make pairing-idempotency`
  - `make pairing-refresh`
  - `make pairing-refresh-race`
- 红色（预期失败）：
  - `make pairing-race-negative`（非原子的 begin/commit 上限竞争条件）
  - `make pairing-idempotency-negative`
  - `make pairing-refresh-negative`
  - `make pairing-refresh-race-negative`

<div id="ingress-trace-correlation-idempotency">
  ### 入口追踪关联 / 幂等性
</div>

**声明：** 入口摄取在扇出场景下应保持追踪关联，并在提供方重试时保持幂等性。

含义如下：

- 当一个外部事件变成多个内部消息时，每一部分都保持相同的追踪/事件标识。
- 重试不会导致重复处理。
- 如果缺少提供方事件 ID，去重会回退到一个安全的键值（例如 trace ID），以避免丢弃本应视为不同的事件。

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

**论断：** 路由在默认情况下必须保持 DM 会话相互隔离，且只有在显式配置时（通道优先级 + identityLinks）才允许合并会话。

这意味着：

- 通道级的 dmScope 覆盖设置必须优先于全局默认配置。
- identityLinks 只应在显式链接的分组内合并，而不能在彼此无关的对等方之间合并。

- 绿色：
  - `make routing-precedence`
  - `make routing-identitylinks`
- 红色（预期）：
  - `make routing-precedence-negative`
  - `make routing-identitylinks-negative`