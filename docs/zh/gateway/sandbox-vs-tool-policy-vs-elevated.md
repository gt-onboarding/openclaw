---
title: 沙箱 vs 工具策略 vs 提权执行
summary: "工具为何被阻止：沙箱运行时、工具允许/拒绝策略，以及提权执行门控机制"
read_when: "当你遇到 “sandbox jail” 或看到工具/提权被拒绝，并且想确切知道要修改哪个配置键。"
status: active
---

<div id="sandbox-vs-tool-policy-vs-elevated">
  # 沙箱 vs 工具策略 vs 提权
</div>

OpenClaw 有三个相关但不同的控制机制：

1. **沙箱**（`agents.defaults.sandbox.*` / `agents.list[].sandbox.*`）决定**工具在哪里运行**（Docker vs 宿主机）。
2. **工具策略**（`tools.*`、`tools.sandbox.tools.*`、`agents.list[].tools.*`）决定**哪些工具可用/被允许**。
3. **提权**（`tools.elevated.*`、`agents.list[].tools.elevated.*`）是一个**仅用于执行的应急通道**，用于在启用沙箱时在宿主机上运行。

<div id="quick-debug">
  ## 快速调试
</div>

使用 inspector 查看 OpenClaw *实际* 在做什么：

```bash
openclaw sandbox explain
openclaw sandbox explain --session agent:main:main
openclaw sandbox explain --agent work
openclaw sandbox explain --json
```

It prints:

* 有效的沙箱模式/scope/工作区访问权限
* 当前会话目前是否在沙箱中（main vs non-main）
* 有效的沙箱工具允许/拒绝规则（以及它来自 agent/global/default 中的哪一层）
* 提权门控规则和 fix-it 键路径

<div id="sandbox-where-tools-run">
  ## 沙箱：工具运行的位置
</div>

沙箱由 `agents.defaults.sandbox.mode` 控制：

* `"off"`：所有内容都在宿主机上运行。
* `"non-main"`：只有非主会话会被放入沙箱（对群组/频道来说是常见的“惊喜”行为）。
* `"all"`：所有内容都在沙箱中运行。

完整矩阵（scope、工作区挂载、镜像）参见 [Sandboxing](/zh/gateway/sandboxing)。

<div id="bind-mounts-security-quick-check">
  ### 绑定挂载（安全快速检查）
</div>

* `docker.binds` 会*刺穿*沙箱文件系统：你挂载的任何内容都会按照你设置的模式（`:ro` 或 `:rw`）在容器内可见。
* 如果省略模式，默认是读写；对源码/机密文件优先使用 `:ro`。
* `scope: "shared"` 会忽略各智能体的绑定（只应用全局绑定）。
* 绑定 `/var/run/docker.sock` 实质上是把宿主机的控制权交给沙箱；只在你明确需要时这么做。
* 工作区访问（`workspaceAccess: "ro"`/`"rw"`）与绑定模式相互独立。

<div id="tool-policy-which-tools-existare-callable">
  ## 工具策略：有哪些工具 / 可被调用
</div>

需要关注的有几个层级：

* **工具配置**：`tools.profile` 和 `agents.list[].tools.profile`（基础允许列表）
* **提供方工具配置**：`tools.byProvider[provider].profile` 和 `agents.list[].tools.byProvider[provider].profile`
* **全局 / 按智能体的工具策略**：`tools.allow`/`tools.deny` 和 `agents.list[].tools.allow`/`agents.list[].tools.deny`
* **提供方工具策略**：`tools.byProvider[provider].allow/deny` 和 `agents.list[].tools.byProvider[provider].allow/deny`
* **沙箱工具策略**（仅在沙箱启用时生效）：`tools.sandbox.tools.allow`/`tools.sandbox.tools.deny` 和 `agents.list[].tools.sandbox.tools.*`

经验规则：

* `deny` 始终优先生效。
* 如果 `allow` 非空，其它所有内容都视为被阻止。
* 工具策略是硬性停止点：`/exec` 不能覆盖已被拒绝的 `exec` 工具。
* `/exec` 只会为已授权发送方修改会话默认值；它不会授予工具访问权限。
  提供方工具键名既可以是 `provider`（例如 `google-antigravity`），也可以是 `provider/model`（例如 `openai/gpt-5.2`）。

<div id="tool-groups-shorthands">
  ### 工具分组（简写）
</div>

工具策略（global、智能体、沙箱）支持使用 `group:*` 条目，将其展开为多个工具：

```json5
{
  tools: {
    sandbox: {
      tools: {
        allow: ["group:runtime", "group:fs", "group:sessions", "group:memory"]
      }
    }
  }
}
```

可用分组：

* `group:runtime`: `exec`, `bash`, `process`
* `group:fs`: `read`, `write`, `edit`, `apply_patch`
* `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
* `group:memory`: `memory_search`, `memory_get`
* `group:ui`: `browser`, `canvas`
* `group:automation`: `cron`, `gateway`
* `group:messaging`: `message`
* `group:nodes`: `nodes`
* `group:openclaw`: 所有内置 OpenClaw 工具（不包括提供方插件）

<div id="elevated-exec-only-run-on-host">
  ## 提权：仅限 `exec` 的“在宿主机上运行”
</div>

提权 **不会**授予额外工具；它只影响 `exec`。

* 如果你在沙箱中，`/elevated on`（或带有 `elevated: true` 的 `exec`）会在宿主机上运行（仍可能需要审批）。
* 使用 `/elevated full` 可在本会话中跳过 exec 审批。
* 如果你已经直接在宿主机上运行，提权实际上是一次无效果操作（no-op，仍然受限）。
* 提权 **不是**按 skill 划分 scope 的，也**不会**覆盖工具的 allow/deny 配置。
* `/exec` 与提权是分离的。它只为已授权发送方调整按会话的 exec 默认行为。

门控条件（Gates）：

* 启用：`tools.elevated.enabled`（以及可选的 `agents.list[].tools.elevated.enabled`）
* 发送方允许列表：`tools.elevated.allowFrom.<provider>`（以及可选的 `agents.list[].tools.elevated.allowFrom.<provider>`）

参见 [提权模式](/zh/tools/elevated)。

<div id="common-sandbox-jail-fixes">
  ## 常见沙箱“监禁”问题修复
</div>

<div id="tool-x-blocked-by-sandbox-tool-policy">
  ### “Tool X 被沙箱工具策略阻止”
</div>

修复方式（任选其一）：

* 禁用沙箱：`agents.defaults.sandbox.mode=off`（或针对单个智能体 `agents.list[].sandbox.mode=off`）
* 在沙箱中允许该工具：
  * 从 `tools.sandbox.tools.deny` 中移除它（或针对单个智能体 `agents.list[].tools.sandbox.tools.deny`）
  * 或将它添加到 `tools.sandbox.tools.allow` 中（或针对单个智能体的 allow 列表）

<div id="i-thought-this-was-main-why-is-it-sandboxed">
  ### “我以为这是 main，为什么会被沙箱隔离？”
</div>

在 `"non-main"` 模式下，群组/频道的 key *不是* main。使用主会话 key（可通过 `sandbox explain` 查看），或者将模式切换为 `"off"`。