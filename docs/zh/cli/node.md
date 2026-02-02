---
title: 节点
summary: "`openclaw node`（无头节点主机）CLI 参考"
read_when:
  - 运行无头节点主机时
  - 为 system.run 配对非 macOS 节点时
---

<div id="openclaw-node">
  # `openclaw node`
</div>

运行一个连接到 Gateway WebSocket 的**无头节点主机**，并在此机器上暴露
`system.run` / `system.which` 接口。

<div id="why-use-a-node-host">
  ## 为什么要使用节点主机？
</div>

当你希望智能体在你的网络中**在其他机器上运行命令**，但又不想在这些机器上安装完整的 macOS 配套应用时，就可以使用节点主机。

常见用例：

* 在远程 Linux/Windows 机器上运行命令（构建服务器、实验室机器、NAS）。
* 将执行保持在 Gateway 侧的**沙箱**中，但把已批准的运行委派到其他主机。
* 为自动化或 CI 节点提供轻量级的无头执行目标。

执行仍然由节点主机上的 **exec 执行审批**和按智能体配置的允许列表进行保护，因此你可以让命令访问保持在明确限定的 scope 内。

<div id="browser-proxy-zero-config">
  ## 浏览器代理（零配置）
</div>

当节点上未禁用 `browser.enabled` 时，节点主机会自动提供一个浏览器代理。这样智能体就可以在该节点上使用浏览器自动化功能，而无需额外配置。

如有需要，你可以在节点上将其禁用：

```json5
{
  nodeHost: {
    browserProxy: {
      enabled: false
    }
  }
}
```

<div id="run-foreground">
  ## 前台运行
</div>

```bash
openclaw node run --host <gateway-host> --port 18789
```

Options:

* `--host <host>`: Gateway WebSocket 主机（默认：`127.0.0.1`）
* `--port <port>`: Gateway WebSocket 端口（默认：`18789`）
* `--tls`: 为与 Gateway 的连接启用 TLS
* `--tls-fingerprint <sha256>`: 预期的 TLS 证书指纹（sha256）
* `--node-id <id>`: 覆盖节点 ID（并清除配对 token）
* `--display-name <name>`: 覆盖节点显示名称

<div id="service-background">
  ## 服务（后台运行）
</div>

将无头节点主机安装为用户级服务。

```bash
openclaw node install --host <gateway-host> --port 18789
```

选项：

* `--host <host>`: Gateway WebSocket 主机（默认：`127.0.0.1`）
* `--port <port>`: Gateway WebSocket 端口（默认：`18789`）
* `--tls`: 为与 Gateway 的连接启用 TLS
* `--tls-fingerprint <sha256>`: 期望的 TLS 证书指纹（sha256）
* `--node-id <id>`: 覆盖节点 ID（会清除配对令牌）
* `--display-name <name>`: 覆盖节点显示名称
* `--runtime <runtime>`: 服务运行时环境（`node` 或 `bun`）
* `--force`: 如果已安装则重新安装/覆盖

管理服务：

```bash
openclaw node status
openclaw node stop
openclaw node restart
openclaw node uninstall
```

使用 `openclaw node run` 以前台方式运行节点（不以服务形式运行）。

服务命令支持 `--json` 参数，以生成机器可读的输出。

<div id="pairing">
  ## 配对
</div>

首次连接时会在 Gateway 上创建一个待处理的节点配对请求。
通过以下方式批准：

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
```

运行该节点的主机会将其节点 ID、token、显示名称以及 Gateway 连接信息存储在
`~/.openclaw/node.json` 中。

<div id="exec-approvals">
  ## 执行授权
</div>

`system.run` 受本地执行授权控制：

* `~/.openclaw/exec-approvals.json`
* [执行授权](/zh/tools/exec-approvals)
* `openclaw approvals --node <id|name|ip>`（从 Gateway 侧进行编辑）