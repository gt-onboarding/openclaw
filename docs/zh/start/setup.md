---
title: 设置
summary: "设置指南：在保持 OpenClaw 配置个性化的同时持续保持最新状态"
read_when:
  - 正在配置新机器
  - 你希望在不破坏个人配置的前提下获得“最新最强”的体验
---

<div id="setup">
  # 安装与配置
</div>

最后更新：2026-01-01

<div id="tldr">
  ## 快速概览
</div>

* **定制内容位于仓库之外：**`~/.openclaw/workspace`（工作区）+ `~/.openclaw/openclaw.json`（配置）。
* **稳定工作流：**安装 macOS 应用；让它运行自带的 Gateway。
* **前沿工作流：**使用 `pnpm gateway:watch` 自行运行 Gateway，然后让 macOS 应用以 Local 模式接入。

<div id="prereqs-from-source">
  ## 前置条件（从源码构建）
</div>

* Node.js `>=22`
* `pnpm`
* Docker（可选；仅用于容器化部署/端到端（e2e）测试——参见 [Docker](/zh/install/docker)）

<div id="tailoring-strategy-so-updates-dont-hurt">
  ## 定制策略（避免更新带来麻烦）
</div>

如果你想要“100% 为你量身定制”*同时*又希望后续更新轻松，请把你的自定义内容放在：

* **Config：** `~/.openclaw/openclaw.json`（JSON/类 JSON5）
* **Workspace：** `~/.openclaw/workspace`（技能、提示词、记忆；把它做成一个私有 git 仓库）

只需初始化一次：

```bash
openclaw setup
```

在此仓库中，使用本地 CLI 入口命令：

```bash
openclaw setup
```

如果你还没有进行全局安装，请运行 `pnpm openclaw setup`。

<div id="stable-workflow-macos-app-first">
  ## 稳定工作流程（优先使用 macOS 应用）
</div>

1. 安装并启动 **OpenClaw.app**（常驻菜单栏）。
2. 完成初始引导/权限检查清单（TCC 权限弹窗）。
3. 确保 Gateway 处于 **Local（本地）** 模式并已在运行（由应用托管）。
4. 绑定渠道（示例：WhatsApp）：

```bash
openclaw channels login
```

5. 基本检查：

```bash
openclaw health
```

如果你的构建中没有提供首次配置向导（onboarding）：

* 先运行 `openclaw setup`，再运行 `openclaw channels login`，最后手动启动 Gateway（`openclaw gateway`）。

<div id="bleeding-edge-workflow-gateway-in-a-terminal">
  ## 前沿工作流（在终端中运行 Gateway）
</div>

目标：在 TypeScript 版 Gateway 上开发、启用热重载，并保持 macOS 应用的 UI 持续连接。

<div id="0-optional-run-the-macos-app-from-source-too">
  ### 0)（可选）也从源码运行 macOS 应用
</div>

如果你也想使用最新开发版的 macOS 应用：

```bash
./scripts/restart-mac.sh
```

<div id="1-start-the-dev-gateway">
  ### 1) 启动开发环境下的 Gateway
</div>

```bash
pnpm install
pnpm gateway:watch
```

`gateway:watch` 以监听模式运行 Gateway，并在 TypeScript 代码发生变更时自动重新加载。

<div id="2-point-the-macos-app-at-your-running-gateway">
  ### 2) 将 macOS 应用指向正在运行的 Gateway
</div>

在 **OpenClaw.app** 中：

* 连接模式（Connection Mode）: **Local**
  应用会通过配置好的端口连接到正在运行的 Gateway。

<div id="3-verify">
  ### 3) 验证
</div>

* 在应用中 Gateway 的状态应显示为 **“Using existing gateway …”**
* 或通过 CLI：

```bash
openclaw health
```

<div id="common-footguns">
  ### 常见坑点
</div>

* **端口错误：** Gateway 的 WS 默认地址为 `ws://127.0.0.1:18789`；请确保 App 和 CLI 使用同一个端口。
* **状态存储位置：**
  * 凭据：`~/.openclaw/credentials/`
  * 会话：`~/.openclaw/agents/<agentId>/sessions/`
  * 日志：`/tmp/openclaw/`

<div id="credential-storage-map">
  ## 凭据存储映射
</div>

在调试身份验证或决定要备份哪些内容时使用：

* **WhatsApp**：`~/.openclaw/credentials/whatsapp/<accountId>/creds.json`
* **Telegram 机器人令牌**：配置/环境变量或 `channels.telegram.tokenFile`
* **Discord 机器人令牌**：配置/环境变量（暂不支持令牌文件）
* **Slack 令牌**：配置/环境变量（`channels.slack.*`）
* **配对允许列表**：`~/.openclaw/credentials/<channel>-allowFrom.json`
* **模型认证配置文件**：`~/.openclaw/agents/<agentId>/agent/auth-profiles.json`
* **旧版 OAuth 导入**：`~/.openclaw/credentials/oauth.json`
  更多详情：参见 [Security](/zh/gateway/security#credential-storage-map)。

<div id="updating-without-wrecking-your-setup">
  ## 更新（避免破坏现有环境）
</div>

* 把 `~/.openclaw/workspace` 和 `~/.openclaw/` 视为“你的东西”；不要把个人提示词/配置放进 `openclaw` 仓库。
* 更新源码：先 `git pull`，如有 lockfile 变更再执行 `pnpm install`，并继续使用 `pnpm gateway:watch`。

<div id="linux-systemd-user-service">
  ## Linux（systemd 用户服务）
</div>

在 Linux 上安装时，会使用 systemd 的 **用户**服务。默认情况下，systemd 会在注销或空闲时停止用户服务，从而终止 Gateway。初始化引导流程会尝试为你启用 lingering（可能会提示使用 sudo）。如果仍未启用，请运行：

```bash
sudo loginctl enable-linger $USER
```

对于需要长时间运行或支持多用户的服务器，建议使用 **system** 级服务，而不是
用户级服务（无需启用 lingering）。有关 systemd 相关说明，请参见 [Gateway runbook](/zh/gateway)。

<div id="related-docs">
  ## 相关文档
</div>

* [Gateway 运行手册](/zh/gateway)（启动参数、进程监管、端口）
* [Gateway 配置](/zh/gateway/configuration)（配置模式（schema）及示例）
* [Discord](/zh/channels/discord) 和 [Telegram](/zh/channels/telegram)（回复标签与 `replyToMode` 设置）
* [OpenClaw 助手设置](/zh/start/openclaw)
* [macOS 应用](/zh/platforms/macos)（Gateway 生命周期管理）