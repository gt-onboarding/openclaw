---
title: 入门
summary: "新手指南：从零到第一条消息（向导、认证、渠道、配对）"
read_when:
  - 从零开始进行首次安装配置
  - 你希望以最快路径完成从安装 → 上手引导 → 第一条消息的流程
---

<div id="getting-started">
  # 入门
</div>

目标：尽可能快地从 **零** → **第一个可用聊天**（带合理的默认配置）。

最快的聊天方式：打开 Control UI（不需要配置任何渠道）。运行 `openclaw dashboard`
在浏览器中聊天，或者在 Gateway 主机上打开 `http://127.0.0.1:18789/`。
文档参考：[Dashboard](/zh/web/dashboard) 和 [Control UI](/zh/web/control-ui)。

推荐路径：使用 **CLI 引导向导**（`openclaw onboard`）。它会配置：

* 模型/认证（推荐使用 OAuth）
* Gateway 设置
* 渠道（WhatsApp/Telegram/Discord/Mattermost（插件）/…）
* 配对默认值（安全私信）
* 工作区初始化 + 技能
* 可选的后台服务

如果你需要更深入的参考页面，请跳转到：[Wizard](/zh/start/wizard)、[Setup](/zh/start/setup)、[Pairing](/zh/start/pairing)、[Security](/zh/gateway/security)。

关于沙箱的说明：`agents.defaults.sandbox.mode: "non-main"` 会使用 `session.mainKey`（默认为 `"main"`），
因此群组/频道会话会在沙箱中运行。如果你希望主智能体始终在宿主机上运行，
请为该智能体显式设置覆盖配置：

```json
{
  "routing": {
    "agents": {
      "main": {
        "workspace": "~/.openclaw/workspace",
        "sandbox": { "mode": "off" }
      }
    }
  }
}
```

<div id="0-prereqs">
  ## 0) 前置条件
</div>

* Node `>=22`
* `pnpm`（可选；如果你要从源码构建，建议使用）
* **推荐：** 用于 Web 搜索的 Brave Search API 密钥。最简单的方式是：
  `openclaw configure --section web`（会存储到 `tools.web.search.apiKey`）。
  参见 [Web tools](/zh/tools/web)。

macOS：如果你计划构建应用，请安装 Xcode / CLT。只运行 CLI 和 Gateway 的话，Node 就足够了。
Windows：请使用 **WSL2**（推荐 Ubuntu）。强烈建议使用 WSL2；原生 Windows 尚未测试，问题更多，工具兼容性也更差。先安装 WSL2，然后在 WSL2 中执行下面的 Linux 步骤。参见 [Windows (WSL2)](/zh/platforms/windows)。

<div id="1-install-the-cli-recommended">
  ## 1) 安装 CLI（推荐）
</div>

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

安装选项（安装方式、非交互模式、从 GitHub 安装）：[安装](/zh/install)。

Windows（PowerShell）：

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

另一种方式（全局安装）：

```bash
npm install -g openclaw@latest
```

```bash
pnpm add -g openclaw@latest
```

<div id="2-run-the-onboarding-wizard-and-install-the-service">
  ## 2）运行入门向导（并安装服务）
</div>

```bash
openclaw onboard --install-daemon
```

你需要在此步骤中做出以下选择：

* **本地 vs 远程** Gateway
* **认证（Auth）**：OpenAI Code（Codex）订阅（OAuth）或 API key。对于 Anthropic，我们推荐使用 API key；也支持 `claude setup-token`。
* **提供方（Providers）**：WhatsApp 扫码登录、Telegram/Discord 机器人 token、Mattermost 插件 token 等。
* **守护进程（Daemon）**：以后台服务方式安装（launchd/systemd；WSL2 使用 systemd）
  * **运行时（Runtime）**：Node.js（推荐；WhatsApp/Telegram 必需）。不推荐使用 Bun。
* **Gateway token**：向导会默认生成一个（即使是在回环地址上）并将其存储在 `gateway.auth.token` 中。

向导文档：[Wizard](/zh/start/wizard)

<div id="auth-where-it-lives-important">
  ### 认证：存放位置（重要）
</div>

* **推荐的 Anthropic 配置方式：** 设置一个 API key（向导可以为服务使用进行保存）。如果你想复用 Claude Code 凭据，也支持 `claude setup-token`。

* OAuth 凭据（旧版导入）：`~/.openclaw/credentials/oauth.json`

* 认证配置（OAuth + API keys）：`~/.openclaw/agents/<agentId>/agent/auth-profiles.json`

无界面/服务器环境提示：先在一台普通机器上完成 OAuth，然后将 `oauth.json` 复制到 Gateway 主机上。

<div id="3-start-the-gateway">
  ## 3) 启动 Gateway
</div>

如果你在引导安装过程中已经安装了该服务，Gateway 现在应该已经在运行：

```bash
openclaw gateway status
```

手动运行（前台）：

```bash
openclaw gateway --port 18789 --verbose
```

Dashboard（本地回环）：`http://127.0.0.1:18789/`
如果已配置令牌，请将其粘贴到 Control UI 的设置中（将存储为 `connect.params.auth.token`）。

⚠️ **Bun 警告（WhatsApp + Telegram）：** Bun 与这些通道存在已知问题。如果你使用 WhatsApp 或 Telegram，请使用 **Node** 运行 Gateway。

<div id="35-quick-verify-2-min">
  ## 3.5) 快速验证（约 2 分钟）
</div>

```bash
openclaw status
openclaw health
openclaw security audit --deep
```

<div id="4-pair-connect-your-first-chat-surface">
  ## 4）配对并连接你的第一个聊天渠道
</div>

<div id="whatsapp-qr-login">
  ### WhatsApp（扫码登录）
</div>

```bash
openclaw channels login
```

在 WhatsApp 中依次进入 设置 → 已链接的设备，然后扫码。

WhatsApp 文档：[WhatsApp](/zh/channels/whatsapp)

<div id="telegram-discord-others">
  ### Telegram / Discord / 其他
</div>

向导可以为你写入令牌/配置文件。若你更喜欢手动配置，请从以下内容开始：

* Telegram: [Telegram](/zh/channels/telegram)
* Discord: [Discord](/zh/channels/discord)
* Mattermost（插件）: [Mattermost](/zh/channels/mattermost)

**Telegram 私信提示：** 你的第一条私信会返回一个配对码。请在下一步中批准它，否则机器人将不会回复。

<div id="5-dm-safety-pairing-approvals">
  ## 5) 私信安全（配对审批）
</div>

默认行为：来自未知会话的私信会返回一个短码，在审批通过之前消息不会被处理。
如果你的第一条私信没有收到回复，请手动批准这次配对：

```bash
openclaw pairing list whatsapp
openclaw pairing approve whatsapp <code>
```

配对文档：[配对](/zh/start/pairing)

<div id="from-source-development">
  ## 从源码运行（开发环境）
</div>

如果你正在开发或修改 OpenClaw 本身，请从源码运行：

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # 首次运行时自动安装 UI 依赖项
pnpm build
openclaw onboard --install-daemon
```

如果你还没有进行全局安装，请在代码仓库中通过运行 `pnpm openclaw ...` 来执行初始化步骤。
`pnpm build` 也会同时打包 A2UI 资源；如果你只需要执行这一步，请使用 `pnpm canvas:a2ui:bundle`。

Gateway（来自此代码仓库）：

```bash
node openclaw.mjs gateway --port 18789 --verbose
```

<div id="7-verify-end-to-end">
  ## 7) 进行端到端验证
</div>

在新的终端中，发送一条测试消息：

```bash
openclaw message send --target +15555550123 --message "Hello from OpenClaw"
```

如果 `openclaw health` 显示“no auth configured”，请回到向导中配置 OAuth/密钥认证——否则智能体将无法响应。

提示：`openclaw status --all` 是最适合复制粘贴分享的只读调试报告。
健康检查：`openclaw health`（或 `openclaw status --deep`）会向正在运行的 Gateway 请求一份健康状态快照。

<div id="next-steps-optional-but-great">
  ## 后续步骤（可选，但非常推荐）
</div>

* macOS 菜单栏应用 + 语音唤醒：[macOS app](/zh/platforms/macos)
* iOS/Android 节点（Canvas/相机/语音）：[Nodes](/zh/nodes)
* 远程访问（SSH 隧道 / Tailscale Serve）：[Remote access](/zh/gateway/remote) 和 [Tailscale](/zh/gateway/tailscale)
* 常驻运行 / VPN 部署：[Remote access](/zh/gateway/remote)、[exe.dev](/zh/platforms/exe-dev)、[Hetzner](/zh/platforms/hetzner)、[macOS remote](/zh/platforms/mac/remote)