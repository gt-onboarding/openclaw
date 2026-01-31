---
title: 子进程
summary: "macOS 上 Gateway 的生命周期（launchd）"
read_when:
  - 将 macOS 应用与 Gateway 生命周期集成
---

<div id="gateway-lifecycle-on-macos">
  # macOS 上的 Gateway 生命周期
</div>

macOS 应用默认**通过 launchd 管理 Gateway**，而不是将 Gateway 作为子进程
启动。它会先尝试连接已在配置端口上运行的 Gateway；如果没有可达的实例，
则通过外部 `openclaw` CLI 启用 launchd 服务（应用本身不内置 Gateway 运行时）。
这样可以在登录时可靠地自动启动，并在崩溃后自动重启。

子进程模式（由应用直接启动 Gateway）目前**未在使用**。如果你需要与 UI 更紧密的
耦合，请在终端中手动运行 Gateway。

<div id="default-behavior-launchd">
  ## 默认行为（launchd）
</div>

* 应用会为每个用户安装一个名为 `bot.molt.gateway` 的 LaunchAgent
  （在使用 `--profile`/`OPENCLAW_PROFILE` 时则为 `bot.molt.<profile>`；同时仍兼容旧版的 `com.openclaw.*`）。
* 当启用 Local 模式时，应用会确保该 LaunchAgent 已加载，
  并在需要时启动 Gateway。
* 日志会写入 launchd 的 Gateway 日志路径（可在调试设置中查看）。

常用命令：

```bash
launchctl kickstart -k gui/$UID/bot.molt.gateway
launchctl bootout gui/$UID/bot.molt.gateway
```

在运行命名配置文件时，将该标签替换为 `bot.molt.&lt;profile&gt;`。


<div id="unsigned-dev-builds">
  ## 未签名开发构建
</div>

`scripts/restart-mac.sh --no-sign` 用于在你没有签名密钥时进行本地快速构建。为防止 launchd 指向未签名的 relay 二进制文件，它会：

* 写入 `~/.openclaw/disable-launchagent` 标记文件。

以签名方式运行 `scripts/restart-mac.sh` 时，如果该标记文件存在，将清除此覆盖设置。要手动重置：

```bash
rm ~/.openclaw/disable-launchagent
```


<div id="attach-only-mode">
  ## 仅附加模式
</div>

要强制 macOS 应用**始终不安装或管理 launchd**，请在启动时加入
`--attach-only`（或 `--no-launchd`）。这会创建 `~/.openclaw/disable-launchagent`，
因此应用只会连接到已在运行的 Gateway。你也可以在 Debug Settings 中切换同样的行为。

<div id="remote-mode">
  ## 远程模式
</div>

远程模式不会启动本地 Gateway。应用程序会通过 SSH 隧道连接到远程主机，并通过该隧道与其通信。

<div id="why-we-prefer-launchd">
  ## 为何我们更倾向使用 launchd
</div>

- 登录时自动启动。
- 内置的重启 / KeepAlive 机制。
- 可预测的日志与进程管理。

如果未来再次需要真正的子进程模式，应将其文档化为一个单独、明确的仅限开发使用的模式。