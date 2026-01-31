---
title: Exec 执行审批
summary: "Exec 执行审批、允许列表和沙箱逃逸提示"
read_when:
  - 配置 Exec 执行审批或允许列表
  - 在 macOS 应用中实现 Exec 执行审批 UX
  - 审查沙箱逃逸提示及其影响
---

<div id="exec-approvals">
  # 执行审批（Exec approvals）
</div>

执行审批是允许沙箱中的智能体在真实主机（运行 `gateway` 或 `node` 的那台机器）上运行命令时的**配套应用 / 节点宿主侧安全护栏**。可以把它理解为安全联锁：只有在策略 + 允许列表 +（可选）用户审批**全部**同意时，命令才会被允许执行。
执行审批是**在**工具策略和提权门控**之外额外叠加**的一层（除非将提权级别设置为 `full`，这会跳过审批）。
生效的策略是 `tools.exec.*` 和审批默认值中**更严格**的那个；如果省略了某个审批字段，就会使用对应的 `tools.exec` 配置值。

如果配套应用 UI **不可用**，任何需要交互确认的请求都会由**询问回退机制**处理（默认：拒绝）。

<div id="where-it-applies">
  ## 适用范围
</div>

Exec 审批会在执行所在主机本地强制执行：

* **Gateway 主机** → Gateway 机器上的 `openclaw` 进程
* **节点主机** → 节点运行器（macOS 配套应用或无头节点主机）

macOS 拆分：

* **节点主机服务** 通过本地 IPC 将 `system.run` 转发给 **macOS 应用**。
* **macOS 应用** 在 UI 上下文中执行审批并运行命令。

<div id="settings-and-storage">
  ## 设置和存储
</div>

审批记录保存在执行主机上的本地 JSON 文件中：

`~/.openclaw/exec-approvals.json`

示例结构：

```json
{
  "version": 1,
  "socket": {
    "path": "~/.openclaw/exec-approvals.sock",
    "token": "base64url-token"
  },
  "defaults": {
    "security": "deny",
    "ask": "on-miss",
    "askFallback": "deny",
    "autoAllowSkills": false
  },
  "agents": {
    "main": {
      "security": "allowlist",
      "ask": "on-miss",
      "askFallback": "deny",
      "autoAllowSkills": true,
      "allowlist": [
        {
          "id": "B0C8C0B3-2C2D-4F8A-9A3C-5A4B3C2D1E0F",
          "pattern": "~/Projects/**/bin/rg",
          "lastUsedAt": 1737150000000,
          "lastUsedCommand": "rg -n TODO",
          "lastResolvedPath": "/Users/user/Projects/.../bin/rg"
        }
      ]
    }
  }
}
```

<div id="policy-knobs">
  ## 策略参数
</div>

<div id="security-execsecurity">
  ### 安全（`exec.security`）
</div>

* **deny**：阻止所有主机 exec 请求。
* **allowlist**：只允许允许列表中的命令。
* **full**：允许所有操作（等同于 elevated）。

<div id="ask-execask">
  ### Ask (`exec.ask`)
</div>

* **off**: 从不询问。
* **on-miss**: 仅在允许列表未命中时询问。
* **always**: 每条命令都询问。

<div id="ask-fallback-askfallback">
  ### 询问回退（`askFallback`）
</div>

当需要提示但无法访问 UI 时，由回退策略决定：

* **deny**：拒绝。
* **allowlist**：仅在允许列表匹配时允许。
* **full**：直接允许。

<div id="allowlist-per-agent">
  ## 允许列表（按 Agent 代理划分）
</div>

允许列表是**按 Agent 代理划分**的。如果存在多个 Agent 代理，请在 macOS app 中切换当前正在编辑的 Agent 代理。模式是**不区分大小写的 glob 匹配**。
模式应解析为**二进制可执行文件路径**（仅包含基名的条目会被忽略）。
旧版 `agents.default` 条目在加载时会迁移到 `agents.main`。

示例：

* `~/Projects/**/bin/bird`
* `~/.local/bin/*`
* `/opt/homebrew/bin/rg`

每个允许列表条目会记录：

* **id** 用于 UI 标识的稳定 UUID（可选）
* **last used** 时间戳
* **last used command**
* **last resolved path**

<div id="auto-allow-skill-clis">
  ## 自动允许技能 CLI
</div>

当启用 **自动允许技能 CLI** 时，由已知技能引用的可执行文件
会在节点（macOS 节点或无头节点主机）上被视为已加入允许列表。它会通过
Gateway 的 RPC 使用 `skills.bins` 获取技能可执行文件列表。如果你需要严格的手动允许列表，请禁用此选项。

<div id="safe-bins-stdin-only">
  ## 安全二进制（仅 stdin）
</div>

`tools.exec.safeBins` 定义了一小组**仅通过 stdin** 的二进制程序（例如 `jq`），
它们可以在允许列表模式下运行，而**不需要**显式的允许列表条目。安全二进制会拒绝
位置式文件参数和类似路径的标记，因此它们只能对传入的数据流进行操作。
在允许列表模式下不会自动允许 Shell 链式调用和重定向。

当每一个顶层命令段都满足允许列表（包括安全二进制或技能自动允许）时，
允许使用 Shell 链式调用（`&&`、`||`、`;`）。在允许列表模式下仍然不支持重定向。

默认安全二进制：`jq`、`grep`、`cut`、`sort`、`uniq`、`head`、`tail`、`tr`、`wc`。

<div id="control-ui-editing">
  ## Control UI 编辑
</div>

使用 **Control UI → Nodes → Exec approvals** 卡片来编辑默认值、按智能体的覆盖配置以及允许列表。选择一个 scope（Defaults 或某个智能体），调整策略，添加或移除允许列表模式，然后点击 **Save**。UI 会为每个模式显示 **last used** 元数据，方便你保持列表整洁。

目标选择器用于在 **Gateway**（本地审批）或某个 **Node** 之间切换。节点必须对外暴露 `system.execApprovals.get/set` 能力（macOS 应用或无界面节点宿主机）。如果某个节点还没有暴露 exec approvals，就直接编辑它本地的
`~/.openclaw/exec-approvals.json`。

CLI：`openclaw approvals` 支持编辑 Gateway 或节点（参见 [Approvals CLI](/zh/cli/approvals)）。

<div id="approval-flow">
  ## 审批流程
</div>

当需要审批确认时，Gateway 会向运维侧客户端广播 `exec.approval.requested`。
Control UI 和 macOS 应用通过 `exec.approval.resolve` 进行处理，然后 Gateway 将
已批准的请求转发到节点主机。

在启用审批的情况下，exec 工具会立刻返回一个审批 ID。使用该 ID 将后续系统事件（`Exec finished` / `Exec denied`）关联起来。如果在超时时间前没有决策到达，该请求会被视为审批超时，并显示为拒绝原因。

确认对话框包括：

* 命令 + 参数
* cwd（当前工作目录）
* 智能体 ID
* 解析出的可执行文件路径
* 主机 + 策略元数据

可执行操作：

* **Allow once** → 立即运行
* **Always allow** → 添加到允许列表并运行
* **Deny** → 阻止

<div id="approval-forwarding-to-chat-channels">
  ## 将审批转发到聊天通道
</div>

你可以将 exec 执行审批请求转发到任意聊天通道（包括插件通道），并通过 `/approve`
进行批准。系统会使用标准的出站投递流水线。

配置：

```json5
{
  approvals: {
    exec: {
      enabled: true,
      mode: "session", // "session" | "targets" | "both"
      agentFilter: ["main"],
      sessionFilter: ["discord"], // substring or regex
      targets: [
        { channel: "slack", to: "U12345678" },
        { channel: "telegram", to: "123456789" }
      ]
    }
  }
}
```

在聊天中回复：

```
/approve <id> allow-once
/approve <id> allow-always
/approve <id> deny
```

<div id="macos-ipc-flow">
  ### macOS 进程间通信（IPC）流程
</div>

```
Gateway -> Node Service (WS)
                 |  IPC (UDS + token + HMAC + TTL)
                 v
             Mac App (UI + approvals + system.run)
```

安全注意事项：

* Unix 套接字权限模式为 `0600`，令牌存储在 `exec-approvals.json` 中。
* 校验对端是否为同一 UID。
* 质询/响应机制（随机数 nonce + HMAC 令牌 + 请求哈希）+ 短 TTL。

<div id="system-events">
  ## 系统事件
</div>

Exec 的生命周期会以系统消息的形式呈现：

* `Exec running`（仅当命令运行时间超过通知阈值时）
* `Exec finished`
* `Exec denied`

这些消息会在节点报告事件后发送到智能体的会话中。
由 Gateway 托管的 exec 审批在命令结束时会发出相同的生命周期事件（也可以在运行时间超过阈值时发出）。
受审批控制的 exec 会在这些消息中将审批 ID 复用为 `runId`，以便于关联。

<div id="implications">
  ## 影响
</div>

* **full** 权限很大；在可行时应优先使用允许列表。
* **ask** 让你始终掌握情况，同时仍然支持快速批准。
* 针对每个智能体分别配置的允许列表，可以防止一个智能体的批准“泄漏”到其他智能体。
* 批准仅适用于来自**已授权发送方**的主机 exec 请求。未授权发送方无法发出 `/exec`。
* `/exec security=full` 是为已授权的运维人员提供的会话级便捷选项，按设计会跳过审批。
  要彻底阻止主机 exec，将 approvals 的 security 设为 `deny`，或通过工具策略禁用 `exec` 工具。

相关内容：

* [Exec 工具](/zh/tools/exec)
* [Elevated 模式](/zh/tools/elevated)
* [技能](/zh/tools/skills)