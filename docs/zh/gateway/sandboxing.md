---
summary: "OpenClaw 沙箱机制工作原理：模式、scope、工作区访问权限和镜像"
title: 沙箱机制
read_when: "当你需要系统性了解沙箱机制，或者需要调优 agents.defaults.sandbox 时阅读。"
status: active
---

<div id="sandboxing">
  # 沙箱运行（Sandboxing）
</div>

OpenClaw 可以在 **Docker 容器中运行工具**，以减小潜在影响范围。
这是一项**可选**功能，由配置项（`agents.defaults.sandbox` 或
`agents.list[].sandbox`）控制。若关闭沙箱运行，工具会在宿主机上运行。
Gateway 始终运行在宿主机上；启用沙箱后，工具执行会在隔离的沙箱环境中进行。

这并不是一个完美的安全边界，但在模型做出不当操作时，确实能大幅限制其对文件系统和进程的访问。

<div id="what-gets-sandboxed">
  ## 会被沙箱隔离的内容
</div>

* 工具执行（`exec`、`read`、`write`、`edit`、`apply_patch`、`process` 等）。
* 可选的沙箱浏览器（`agents.defaults.sandbox.browser`）。
  * 默认情况下，当浏览器工具需要时，沙箱浏览器会自动启动（确保 CDP 可访问）。
    可通过 `agents.defaults.sandbox.browser.autoStart` 和 `agents.defaults.sandbox.browser.autoStartTimeoutMs` 进行配置。
  * `agents.defaults.sandbox.browser.allowHostControl` 允许沙箱内的会话显式地控制宿主机上的浏览器。
  * 可选的允许列表用于限制 `target: "custom"`：`allowedControlUrls`、`allowedControlHosts`、`allowedControlPorts`。

未被沙箱隔离的内容：

* Gateway 进程本身。
* 任何被显式允许在宿主机上运行的工具（例如 `tools.elevated`）。
  * **提权的 exec 在宿主机上运行，并绕过沙箱。**
  * 如果沙箱关闭，`tools.elevated` 不会改变执行行为（本来就在宿主机上）。参见 [Elevated Mode](/zh/tools/elevated)。

<div id="modes">
  ## 模式
</div>

`agents.defaults.sandbox.mode` 控制**何时**使用沙箱：

* `"off"`：完全不使用沙箱。
* `"non-main"`：仅对**非 main** 会话使用沙箱（如果你希望普通聊天在宿主机上运行，这是默认推荐值）。
* `"all"`：所有会话都在沙箱中运行。

注意：`"non-main"` 是基于 `session.mainKey`（默认值为 `"main"`），而不是智能体 ID。
群组/频道会话会使用它们各自的 key，因此会被视为非 main，并会在沙箱中运行。

<div id="scope">
  ## 作用范围
</div>

`agents.defaults.sandbox.scope` 控制**会创建多少个容器**：

* `"session"`（默认）：每个会话一个容器。
* `"agent"`：每个智能体一个容器。
* `"shared"`：所有沙箱化的会话共享一个容器。

<div id="workspace-access">
  ## 工作区访问
</div>

`agents.defaults.sandbox.workspaceAccess` 用来控制**沙箱可以访问到哪些内容**：

* `"none"`（默认）：工具只能看到位于 `~/.openclaw/sandboxes` 下的沙箱工作区。
* `"ro"`：以只读方式将智能体工作区挂载到 `/agent`（禁用 `write`/`edit`/`apply_patch`）。
* `"rw"`：以读写方式将智能体工作区挂载到 `/workspace`。

传入媒体会被复制到当前激活的沙箱工作区（`media/inbound/*`）。
关于技能的说明：`read` 工具的根目录位于沙箱内部。当 `workspaceAccess: "none"` 时，
OpenClaw 会将符合条件的技能镜像到沙箱工作区（`.../skills`），
以便可以被读取。使用 `"rw"` 时，工作区内的技能可以从
`/workspace/skills` 读取。

<div id="custom-bind-mounts">
  ## 自定义绑定挂载
</div>

`agents.defaults.sandbox.docker.binds` 会将额外的宿主机目录挂载到容器中。
格式：`host:container:mode`（例如 `"/home/user/source:/source:rw"`）。

全局和按智能体配置的绑定会被**合并**（而不是互相替换）。在 `scope: "shared"` 下，按智能体配置的绑定会被忽略。

示例（只读源目录 + docker 套接字）：

```json5
{
  agents: {
    defaults: {
      sandbox: {
        docker: {
          binds: [
            "/home/user/source:/source:ro",
            "/var/run/docker.sock:/var/run/docker.sock"
          ]
        }
      }
    },
    list: [
      {
        id: "build",
        sandbox: {
          docker: {
            binds: ["/mnt/cache:/cache:rw"]
          }
        }
      }
    ]
  }
}
```

安全注意事项：

* 绑定会绕过沙箱文件系统：它们会按你设置的模式（`:ro` 或 `:rw`）暴露宿主机路径。
* 敏感挂载（例如 `docker.sock`、机密信息、SSH 密钥）应使用 `:ro`，除非确有必要允许写入。
* 如果你只需要对工作区的只读访问，请配合使用 `workspaceAccess: "ro"`；绑定的模式仍然是独立的。
* 参见 [Sandbox vs Tool Policy vs Elevated](/zh/gateway/sandbox-vs-tool-policy-vs-elevated)，了解绑定如何与工具策略和提权执行交互。

<div id="images-setup">
  ## 镜像与环境准备
</div>

默认镜像：`openclaw-sandbox:bookworm-slim`

先构建一次：

```bash
scripts/sandbox-setup.sh
```

注意：默认镜像**不**包含 Node。如果某个 skill 需要 Node（或其他运行时），可以构建自定义镜像，或者通过 `sandbox.docker.setupCommand` 安装（需要网络出站访问 + 可写的 root 目录 + root 用户）。

沙箱浏览器镜像：

```bash
scripts/sandbox-browser-setup.sh
```

默认情况下，沙箱容器运行时**无网络连接**。
可以通过 `agents.defaults.sandbox.docker.network` 覆盖该设置。

Docker 的安装方式以及容器化部署的 Gateway 请参见：
[Docker](/zh/install/docker)

<div id="setupcommand-one-time-container-setup">
  ## setupCommand（一次性容器设置）
</div>

`setupCommand` 会在沙箱容器创建后**只运行一次**（不会在每次运行时都执行）。
它通过 `sh -lc` 在容器内部执行。

路径：

* 全局：`agents.defaults.sandbox.docker.setupCommand`
* 每个智能体：`agents.list[].sandbox.docker.setupCommand`

常见陷阱：

* 默认的 `docker.network` 为 `"none"`（无出站流量），因此安装软件包会失败。
* `readOnlyRoot: true` 会禁止写入；请将其设置为 `readOnlyRoot: false`，或预制自定义镜像。
* 进行软件包安装时，`user` 必须为 root（省略 `user` 或设置为 `user: "0:0"`）。
* 沙箱中的执行**不会**继承宿主机的 `process.env`。对于技能的 api 密钥，请使用
  `agents.defaults.sandbox.docker.env`（或自定义镜像）。

<div id="tool-policy-escape-hatches">
  ## 工具策略与逃生通道
</div>

工具允许/拒绝策略仍然会在沙箱规则之前生效。如果某个工具在全局或在单个智能体级别被拒绝，启用沙箱也不会让它重新生效。

`tools.elevated` 是一个显式的逃生通道，会在宿主机上运行 `exec`。
`/exec` 指令只对已授权发送方生效，并在会话级别持久生效；若要彻底禁用
`exec`，请使用工具策略的拒绝规则（参见 [Sandbox vs Tool Policy vs Elevated](/zh/gateway/sandbox-vs-tool-policy-vs-elevated)）。

调试方法：

* 使用 `openclaw sandbox explain` 来检查生效的沙箱模式、工具策略和修复相关的配置键。
* 参见 [Sandbox vs Tool Policy vs Elevated](/zh/gateway/sandbox-vs-tool-policy-vs-elevated) 了解“为什么这个被拦截了？”的思维模型。
  务必保持严格锁定。

<div id="multi-agent-overrides">
  ## 多智能体级别的覆盖配置
</div>

每个智能体都可以覆盖沙箱和工具配置：
`agents.list[].sandbox` 和 `agents.list[].tools`（以及用于沙箱工具策略的 `agents.list[].tools.sandbox.tools`）。
优先级规则参见 [多智能体沙箱与工具](/zh/multi-agent-sandbox-tools)。

<div id="minimal-enable-example">
  ## 最小启用配置示例
</div>

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        scope: "session",
        workspaceAccess: "none"
      }
    }
  }
}
```

<div id="related-docs">
  ## 相关文档
</div>

* [沙箱配置](/zh/gateway/configuration#agentsdefaults-sandbox)
* [多智能体沙箱与工具](/zh/multi-agent-sandbox-tools)
* [安全](/zh/gateway/security)