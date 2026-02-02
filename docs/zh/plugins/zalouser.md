---
title: Zalouser
summary: "Zalo Personal 插件：通过 zca-cli 实现二维码登录与消息收发（插件安装 + 渠道配置 + CLI + 工具）"
read_when:
  - 你希望在 OpenClaw 中接入对 Zalo Personal（非官方）的支持
  - 你正在配置或开发 zalouser 插件
---

<div id="zalo-personal-plugin">
  # Zalo 个人账号（插件）
</div>

通过插件为 OpenClaw 提供 Zalo 个人账号支持，使用 `zca-cli` 实现对普通 Zalo 用户账号的自动化控制。

> **警告：** 非官方的自动化操作可能导致账号被暂停或封禁。请自行承担风险。

<div id="naming">
  ## 命名
</div>

Channel id 为 `zalouser`，以明确表明这是用于自动化**个人 Zalo 用户账号**（非官方）。我们保留 `zalo`，以便将来可能用于官方 Zalo api 集成。

<div id="where-it-runs">
  ## 运行位置
</div>

此插件在 **Gateway 进程内** 运行。

如果你使用远程 Gateway，请在 **运行 Gateway 的那台机器上** 安装和配置它，然后重启 Gateway。

<div id="install">
  ## 安装
</div>

<div id="option-a-install-from-npm">
  ### 方式 A：使用 npm 安装
</div>

```bash
openclaw plugins install @openclaw/zalouser
```

然后重启 Gateway。


<div id="option-b-install-from-a-local-folder-dev">
  ### 选项 B：从本地文件夹安装（开发环境）
</div>

```bash
openclaw plugins install ./extensions/zalouser
cd ./extensions/zalouser && pnpm install
```

然后重启 Gateway。


<div id="prerequisite-zca-cli">
  ## 前提条件：zca-cli
</div>

Gateway 机器的 `PATH` 中必须包含 `zca`：

```bash
zca --version
```


<div id="config">
  ## 配置
</div>

频道配置位于 `channels.zalouser` 下（而不是 `plugins.entries.*`）：

```json5
{
  channels: {
    zalouser: {
      enabled: true,
      dmPolicy: "pairing"
    }
  }
}
```


<div id="cli">
  ## CLI
</div>

```bash
openclaw channels login --channel zalouser
openclaw channels logout --channel zalouser
openclaw channels status --probe
openclaw message send --channel zalouser --target <threadId> --message "Hello from OpenClaw"
openclaw directory peers list --channel zalouser --query "name"
```


<div id="agent-tool">
  ## Agent 代理工具
</div>

Tool name: `zalouser`

Actions: `send`, `image`, `link`, `friends`, `groups`, `me`, `status`