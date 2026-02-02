---
title: Exec 主机
summary: "重构方案：Exec 主机路由、节点授权和无头运行器"
read_when:
  - 设计 Exec 主机路由或 Exec 授权
  - 实现节点运行器 + UI IPC
  - 添加 Exec 主机安全模式和斜杠命令
---

<div id="exec-host-refactor-plan">
  # Exec Host 重构方案
</div>

<div id="goals">
  ## 目标
</div>

* 添加 `exec.host` + `exec.security`，在 **沙箱**、**Gateway** 和 **节点** 之间路由执行。
* 保持默认配置**安全**：除非显式启用，否则禁止跨主机执行。
* 将执行拆分为通过本地 IPC 与可选 UI（macOS 应用）协作的**无头运行服务**。
* 提供**按智能体**粒度的策略、允许列表、询问模式以及节点绑定。
* 支持在*有*或*没有*允许列表时都能工作的**询问模式**。
* 跨平台：Unix 套接字 + token 认证（在 macOS/Linux/Windows 上保持一致）。

<div id="non-goals">
  ## 非目标
</div>

* 不提供旧版允许列表迁移，也不支持旧版 schema。
* 不为 node exec 提供 PTY/流式输出（仅提供聚合输出）。
* 不引入现有 Bridge + Gateway 之外的任何额外网络层。

<div id="decisions-locked">
  ## 决策（已锁定）
</div>

* **配置键：** `exec.host` + `exec.security`（允许按智能体覆写）。
* **提权：** 保留 `/elevated` 作为 Gateway 完全访问权限的别名。
* **默认询问策略：** `on-miss`。
* **审批存储位置：** `~/.openclaw/exec-approvals.json`（JSON，无旧格式迁移）。
* **Runner：** 无头系统服务；UI 应用通过 Unix 套接字暴露审批接口。
* **节点标识：** 使用现有的 `nodeId`。
* **套接字认证：** Unix 套接字 + token（跨平台）；如有需要后续再拆分。
* **节点主机状态：** `~/.openclaw/node.json`（节点 id + 配对 token）。
* **macOS 执行主机：** 在 macOS 应用内运行 `system.run`；节点主机服务通过本地 IPC 转发请求。
* **无需 XPC helper：** 坚持使用 Unix 套接字 + token + 对端校验。

<div id="key-concepts">
  ## 核心概念
</div>

<div id="host">
  ### 主机
</div>

* `sandbox`: 在 Docker 容器中执行（当前行为）。
* `gateway`: 在 Gateway 主机上执行。
* `node`: 通过 Bridge 在节点运行器上执行（`system.run`）。

<div id="security-mode">
  ### 安全模式
</div>

* `deny`: 始终阻止。
* `allowlist`: 仅允许匹配项通过。
* `full`: 全部放行（等同于提升权限模式）。

<div id="ask-mode">
  ### 询问模式
</div>

* `off`: 永不询问。
* `on-miss`: 仅在允许列表不匹配时才询问。
* `always`: 每次都询问。

询问与允许列表是**相互独立**的；允许列表可以与 `always` 或 `on-miss` 一起使用。

<div id="policy-resolution-per-exec">
  ### 策略解析（按次 exec）
</div>

1. 解析 `exec.host`（工具参数 → 智能体级覆盖 → 全局默认值）。
2. 解析 `exec.security` 和 `exec.ask`（相同优先级顺序）。
3. 如果 host 为 `sandbox`，则在本地沙箱中执行。
4. 如果 host 为 `gateway` 或 `node`，则在该 host 上应用 security + ask 策略。

<div id="default-safety">
  ## 默认安全设置
</div>

* 默认 `exec.host = sandbox`。
* 默认 `exec.security = deny`，适用于 `Gateway` 和 `node`。
* 默认 `exec.ask = on-miss`（仅在安全策略允许时才会生效）。
* 如果未设置节点绑定，**智能体可以选择任意节点作为目标**，但前提是策略允许。

<div id="config-surface">
  ## 配置项范围
</div>

<div id="tool-parameters">
  ### 工具参数
</div>

* `exec.host`（可选）：`sandbox | gateway | node`。
* `exec.security`（可选）：`deny | allowlist | full`。
* `exec.ask`（可选）：`off | on-miss | always`。
* `exec.node`（可选）：当 `host=node` 时要使用的节点 ID/名称。

<div id="config-keys-global">
  ### 配置键（全局）
</div>

* `tools.exec.host`
* `tools.exec.security`
* `tools.exec.ask`
* `tools.exec.node`（默认节点绑定）

<div id="config-keys-per-agent">
  ### 配置键（按智能体）
</div>

* `agents.list[].tools.exec.host`
* `agents.list[].tools.exec.security`
* `agents.list[].tools.exec.ask`
* `agents.list[].tools.exec.node`

<div id="alias">
  ### 别名
</div>

* `/elevated on` = 将 `tools.exec.host=gateway`、`tools.exec.security=full` 应用于该智能体会话。
* `/elevated off` = 恢复该智能体会话之前的执行设置。

<div id="approvals-store-json">
  ## 审批存储（JSON）
</div>

路径：`~/.openclaw/exec-approvals.json`

用途：

* **执行主机**（Gateway 或节点运行服务）的本地策略和允许列表。
* 在没有可用 UI 时用于回退交互式确认。
* 面向 UI 客户端的 IPC 凭证。

拟议的 schema（v1）：

```json
{
  "version": 1,
  "socket": {
    "path": "~/.openclaw/exec-approvals.sock",
    "token": "base64-opaque-token"
  },
  "defaults": {
    "security": "deny",
    "ask": "on-miss",
    "askFallback": "deny"
  },
  "agents": {
    "agent-id-1": {
      "security": "allowlist",
      "ask": "on-miss",
      "allowlist": [
        {
          "pattern": "~/Projects/**/bin/rg",
          "lastUsedAt": 0,
          "lastUsedCommand": "rg -n TODO",
          "lastResolvedPath": "/Users/user/Projects/.../bin/rg"
        }
      ]
    }
  }
}
```

Notes:

* 不支持旧版的允许列表格式。
* 仅当需要 `ask` 且无法访问 UI 时，`askFallback` 才会生效。
* 文件权限：`0600`。

<div id="runner-service-headless">
  ## Runner 服务（无界面）
</div>

<div id="role">
  ### 角色
</div>

* 在本地强制执行 `exec.security` 和 `exec.ask`。
* 执行系统命令并返回输出。
* 在 exec 生命周期中触发 Bridge 事件（可选但推荐启用）。

<div id="service-lifecycle">
  ### 服务生命周期
</div>

* 在 macOS 上通过 Launchd/守护进程运行；在 Linux/Windows 上作为系统服务运行。
* Approvals JSON 仅保存在执行主机本地。
* UI 监听本地 Unix 套接字；runners 按需进行连接。

<div id="ui-integration-macos-app">
  ## UI 集成（macOS 应用）
</div>

<div id="ipc">
  ### IPC
</div>

* 位于 `~/.openclaw/exec-approvals.sock`（0600）的 Unix 套接字。
* 令牌存储在 `exec-approvals.json`（0600）中。
* 对端校验：仅允许同一 UID。
* 挑战/响应：使用 nonce + HMAC(token, request-hash) 防止重放攻击。
* 短 TTL（例如 10 秒）+ 最大载荷大小限制 + 速率限制。

<div id="ask-flow-macos-app-exec-host">
  ### Ask 流程（macOS 应用 exec host）
</div>

1. 节点服务从 Gateway 收到 `system.run`。
2. 节点服务连接到本地 socket 并发送 prompt/exec 请求。
3. 应用校验对端 + token + HMAC + TTL，如有需要则显示对话框。
4. 应用在 UI 上下文中执行命令并返回输出。
5. 节点服务将输出返回给 Gateway。

如果没有 UI：

* 采用 `askFallback`（`deny|allowlist|full`）。

<div id="diagram-sci">
  ### 示意图（SCI）
</div>

```
Agent -> Gateway -> Bridge -> Node Service (TS)
                         |  IPC (UDS + token + HMAC + TTL)
                         v
                     Mac App (UI + TCC + system.run)
```

<div id="node-identity-binding">
  ## 节点身份与绑定
</div>

* 使用 Bridge 配对生成的现有 `nodeId`。
* 绑定方式：
  * `tools.exec.node` 将智能体限制到特定节点。
  * 如果未设置，智能体可以选择任意节点（策略仍会强制执行默认行为）。
* 节点选择解析规则：
  * `nodeId` 精确匹配
  * `displayName`（规范化）
  * `remoteIp`
  * `nodeId` 前缀（≥ 6 个字符）

<div id="eventing">
  ## 事件系统
</div>

<div id="who-sees-events">
  ### 谁会看到事件
</div>

* 系统事件是**按会话隔离**的，并会在下一条提示时展示给智能体。
* 存储在 Gateway 的内存队列（`enqueueSystemEvent`）中。

<div id="event-text">
  ### 事件文本
</div>

* `Exec started (node=<id>, id=<runId>)`
* `Exec finished (node=<id>, id=<runId>, code=<code>)` + 可选的输出尾部内容
* `Exec denied (node=<id>, id=<runId>, <reason>)`

<div id="transport">
  ### 传输
</div>

方案 A（推荐）：

* Runner 向 Bridge 发送 `event` 事件帧 `exec.started` / `exec.finished`。
* Gateway 的 `handleBridgeEvent` 将其映射为 `enqueueSystemEvent`。

方案 B：

* Gateway 的 `exec` 工具直接处理生命周期（仅同步模式）。

<div id="exec-flows">
  ## 执行流程
</div>

<div id="sandbox-host">
  ### 沙箱主机
</div>

* 当前的 `exec` 行为（在未使用沙箱时，在 Docker 或宿主机上执行）。
* PTY 仅在非沙箱模式下受支持。

<div id="gateway-host">
  ### Gateway 主机
</div>

* Gateway 进程在独立的机器上运行。
* 使用本地 `exec-approvals.json` 进行约束（security/ask/允许列表）。

<div id="node-host">
  ### 节点主机
</div>

* Gateway 使用 `system.run` 调用 `node.invoke`。
* Runner 强制执行本地审批。
* Runner 返回聚合后的 stdout/stderr。
* 可选的 Bridge 事件用于开始/结束/拒绝。

<div id="output-caps">
  ## 输出大小上限
</div>

* 将合并后的 stdout+stderr 输出限制为 **200k**；为事件仅保留末尾 **20k**。
* 截断时添加清晰的后缀（例如，`"… (truncated)"`）。

<div id="slash-commands">
  ## 斜杠命令
</div>

* `/exec host=<sandbox|gateway|node> security=<deny|allowlist|full> ask=<off|on-miss|always> node=<id>`
* 可按智能体和会话级别进行覆盖；除非通过配置保存，否则不会持久化。
* `/elevated on|off|ask|full` 仍是 `host=gateway security=full` 的快捷方式（其中 `full` 会跳过审批）。

<div id="cross-platform-story">
  ## 跨平台方案
</div>

* runner service 是可移植的执行目标。
* UI 是可选的；如果缺失，则使用 `askFallback`。
* Windows/Linux 共享相同的 approvals JSON 和 socket 协议。

<div id="implementation-phases">
  ## 实施阶段
</div>

<div id="phase-1-config-exec-routing">
  ### 阶段 1：配置与执行路由
</div>

* 为 `exec.host`、`exec.security`、`exec.ask`、`exec.node` 添加配置 schema 定义。
* 更新工具底层逻辑以遵循 `exec.host`。
* 添加 `/exec` 斜杠命令，并保留 `/elevated` 别名。

<div id="phase-2-approvals-store-gateway-enforcement">
  ### 阶段 2：审批存储 + Gateway 强制执行
</div>

* 实现 `exec-approvals.json` 读写功能。
* 在 `gateway` 主机上强制执行允许列表和 ask 模式。
* 添加输出上限。

<div id="phase-3-node-runner-enforcement">
  ### 阶段 3：节点运行器强制执行
</div>

* 更新节点运行器，使其强制执行允许列表 + ask 模式。
* 为 macOS 应用 UI 添加 Unix 套接字提示桥接器。
* 接入 `askFallback`。

<div id="phase-4-events">
  ### 阶段 4：事件
</div>

* 为 exec 生命周期添加 节点 → Gateway Bridge 事件。
* 将其映射到 `enqueueSystemEvent`，用于智能体提示。

<div id="phase-5-ui-polish">
  ### 第 5 阶段：UI 打磨
</div>

* Mac 应用：允许列表编辑器、按智能体划分的切换器、提问策略 UI。
* 节点绑定控件（可选）。

<div id="testing-plan">
  ## 测试计划
</div>

* 单元测试：允许列表匹配（glob 通配符 + 不区分大小写）。
* 单元测试：策略解析优先级（tool 参数 → 智能体级覆盖 → 全局）。
* 集成测试：节点运行器 deny/allow/ask 流程。
* Bridge 事件测试：节点事件 → 系统事件路由。

<div id="open-risks">
  ## 未解决风险
</div>

* UI 不可用：确保遵循 `askFallback` 配置。
* 长时间运行的命令：依赖超时机制和输出上限。
* 多节点歧义：除非已有节点绑定或提供显式节点参数，否则报错。

<div id="related-docs">
  ## 相关文档
</div>

* [Exec 工具](/zh/tools/exec)
* [Exec 审批](/zh/tools/exec-approvals)
* [节点](/zh/nodes)
* [提权模式](/zh/tools/elevated)