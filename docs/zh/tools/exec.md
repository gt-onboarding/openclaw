---
title: Exec
summary: "Exec 工具的使用、stdin 模式和 TTY 支持"
read_when:
  - 使用或修改 Exec 工具时
  - 调试 stdin 或 TTY 行为时
---

<div id="exec-tool">
  # Exec 工具
</div>

在工作区中运行 shell 命令。通过 `process` 支持前台和后台执行。
如果禁用了 `process`，`exec` 将以同步方式运行，并忽略 `yieldMs`/`background`。
后台会话按智能体划分；`process` 只能看到来自同一智能体的会话。

<div id="parameters">
  ## 参数
</div>

* `command`（必填）
* `workdir`（默认为 cwd）
* `env`（键/值覆盖）
* `yieldMs`（默认 10000）：延迟后自动转入后台
* `background`（布尔值）：立即在后台运行
* `timeout`（秒，默认 1800）：超时后终止
* `pty`（布尔值）：在可用时运行于伪终端中（仅支持 TTY 的 CLI、编码智能体、终端 UI）
* `host`（`sandbox | gateway | node`）：执行所在主机/环境
* `security`（`deny | allowlist | full`）：`gateway`/`node` 的执行管控模式
* `ask`（`off | on-miss | always`）：`gateway`/`node` 的审批提示策略
* `node`（字符串）：用于 `host=node` 的节点 ID/名称
* `elevated`（布尔值）：请求在 Gateway 宿主机上以提权模式运行；仅当提权结果解析为 `full` 时才会强制使用 `security=full`

说明：

* `host` 默认为 `sandbox`。
* 当沙箱关闭时会忽略 `elevated`（exec 已经在宿主机上运行）。
* `gateway`/`node` 的审批由 `~/.openclaw/exec-approvals.json` 控制。
* `node` 需要已配对的节点（配套应用或无头节点宿主）。
* 如果有多个节点可用，设置 `exec.node` 或 `tools.exec.node` 来选择一个。
* 在非 Windows 宿主机上，如果设置了 `SHELL`，exec 会使用它；如果 `SHELL` 是 `fish`，会优先从 `PATH` 中选择 `bash`（或 `sh`）以避免与 fish 不兼容的脚本，如果两者都不存在，则回退到 `SHELL`。
* 重要：沙箱默认是**关闭的**。如果沙箱关闭，`host=sandbox` 会直接在 Gateway 宿主机上运行（不使用容器），并且**不需要审批**。如需强制审批，请使用 `host=gateway` 并配置 exec 审批（或启用沙箱）。

<div id="config">
  ## 配置
</div>

* `tools.exec.notifyOnExit`（默认：true）：为 true 时，后台执行的 exec 会话在退出时会入队一个系统事件并请求一次心跳。
* `tools.exec.approvalRunningNoticeMs`（默认：10000）：当一个受审批控制的 exec 运行时间超过该值时，发出一次“running”通知（0 表示禁用）。
* `tools.exec.host`（默认：`sandbox`）
* `tools.exec.security`（默认：对于 sandbox 为 `deny`，未设置时对于 Gateway 和 节点 为 `allowlist`）
* `tools.exec.ask`（默认：`on-miss`）
* `tools.exec.node`（默认：未设置）
* `tools.exec.pathPrepend`：在 exec 运行时要预先添加到 `PATH` 的目录列表。
* `tools.exec.safeBins`：仅通过 stdin 接收输入的安全二进制程序，可在没有显式允许列表条目的情况下运行。

示例：

```json5
{
  tools: {
    exec: {
      pathPrepend: ["~/bin", "/opt/oss/bin"]
    }
  }
}
```

<div id="path-handling">
  ### PATH 处理
</div>

* `host=gateway`：将你的登录 shell 的 `PATH` 合并进 exec 环境（除非 exec 调用已经设置了 `env.PATH`）。守护进程本身仍然只使用一个最小化的 `PATH`：
  * macOS: `/opt/homebrew/bin`, `/usr/local/bin`, `/usr/bin`, `/bin`
  * Linux: `/usr/local/bin`, `/usr/bin`, `/bin`
* `host=sandbox`：在容器内运行 `sh -lc`（登录 shell），因此 `/etc/profile` 可能会重置 `PATH`。
  OpenClaw 在 profile 加载完成后，通过一个内部环境变量（不经过 shell 插值）把 `env.PATH` 加到最前面；
  `tools.exec.pathPrepend` 在这里同样生效。
* `host=node`：只会将你显式传入的环境变量覆盖项发送给节点。只有在 exec 调用已经设置了 `env.PATH` 时，
  `tools.exec.pathPrepend` 才会生效。无头节点宿主只接受在其宿主 PATH 前追加的 `PATH`（不允许整体替换）。
  macOS 节点会完全丢弃 `PATH` 覆盖项。

按智能体进行节点绑定（在配置中使用智能体列表索引）：

```bash
openclaw config get agents.list
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
```

Control UI：在 “Nodes” 选项卡中有一个名为 “Exec 节点绑定” 的小面板，用于配置相同的设置。

<div id="session-overrides-exec">
  ## 会话级覆盖（`/exec`）
</div>

使用 `/exec` 为 `host`、`security`、`ask` 和 `node` 设置**会话级**默认值。
发送不带参数的 `/exec` 以显示当前值。

示例：

```
/exec host=gateway security=allowlist ask=on-miss node=mac-1
```

<div id="authorization-model">
  ## 授权模型
</div>

`/exec` 仅对**已授权的发送方**（通道允许列表/配对加上 `commands.useAccessGroups`）生效。
它**只会更新会话状态**，不会写入配置。要彻底禁用 exec，请通过工具策略禁止它（`tools.deny: ["exec"]` 或按智能体配置）。除非你显式设置 `security=full` 且 `ask=off`，否则主机审批仍然适用。

<div id="exec-approvals-companion-app-node-host">
  ## Exec 审批（配套应用 / 节点主机）
</div>

沙箱中的智能体可以要求在 `exec` 在 Gateway 或节点主机上执行之前对每个请求进行审批。
有关策略、允许列表和 UI 流程，参见 [Exec approvals](/zh/tools/exec-approvals)。

当需要审批时，exec 工具会立即返回
`status: "approval-pending"` 和一个审批 ID。一旦审批通过（或被拒绝 / 超时），
Gateway 会发出系统事件（`Exec finished` / `Exec denied`）。如果在 `tools.exec.approvalRunningNoticeMs` 之后命令仍在运行，
会发出一条单次的 `Exec running` 通知。

<div id="allowlist-safe-bins">
  ## 允许列表 + 安全二进制程序
</div>

允许列表策略只匹配**解析后的二进制可执行文件路径**（不会按 basename 匹配）。当
`security=allowlist` 时，只有当管道中的每一段都是允许列表中的条目或安全二进制程序（safe bin）时，
shell 命令才会被自动允许。在允许列表模式下，命令串联（`;`、`&&`、`||`）和重定向会被拒绝。

<div id="examples">
  ## 示例
</div>

前台运行：

```json
{"tool":"exec","command":"ls -la"}
```

背景与投票：

```json
{"tool":"exec","command":"npm run build","yieldMs":1000}
{"tool":"process","action":"poll","sessionId":"<id>"}
```

发送按键（tmux 风格）：

```json
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["Enter"]}
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["C-c"]}
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["Up","Up","Enter"]}
```

提交（只发送变更请求）：

```json
{"tool":"process","action":"submit","sessionId":"<id>"}
```

粘贴（默认用方括号括起来）：

```json
{"tool":"process","action":"paste","sessionId":"<id>","text":"line1\nline2\n"}
```

<div id="apply_patch-experimental">
  ## apply_patch（实验特性）
</div>

`apply_patch` 是 `exec` 的一个子工具，用于以结构化方式编辑多个文件。
需要显式启用它：

```json5
{
  tools: {
    exec: {
      applyPatch: { enabled: true, allowModels: ["gpt-5.2"] }
    }
  }
}
```

备注：

* 仅适用于 OpenAI/OpenAI Codex 模型。
* 工具策略仍然适用；`allow: ["exec"]` 会隐式允许 `apply_patch`。
* 配置项位于 `tools.exec.applyPatch` 下。
