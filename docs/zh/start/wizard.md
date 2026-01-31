---
title: 向导
summary: "CLI 入门向导：为 Gateway、工作区、通道和技能提供引导式配置"
read_when:
  - 运行或配置入门向导时
  - 配置新主机或新设备时
---

<div id="onboarding-wizard-cli">
  # Onboarding Wizard（CLI）
</div>

Onboarding 向导是你在 macOS、Linux 或 Windows（通过 WSL2，强烈推荐）上
设置 OpenClaw 的**推荐**方式。
它会在一个引导式流程中，帮你配置本地 Gateway 或连接到远程 Gateway，
并一并设置渠道、技能和工作区的默认配置。

主要入口：

```bash
openclaw onboard
```

以最快速度开始首次对话：打开 Control UI（无需设置任何渠道）。运行
`openclaw dashboard`，然后在浏览器中开始聊天。文档：[Dashboard](/zh/web/dashboard)。

后续重新配置：

```bash
openclaw configure
```

推荐：先设置一个 Brave Search API 密钥，这样智能体就可以使用 `web_search`
（`web_fetch` 在没有密钥的情况下也能使用）。最简单的做法是运行 `openclaw configure --section web`，
这会将密钥保存到 `tools.web.search.apiKey`。文档参见：[Web 工具](/zh/tools/web)。

<div id="quickstart-vs-advanced">
  ## QuickStart 与 Advanced
</div>

向导以 **QuickStart**（默认）和 **Advanced**（完全控制）两种模式开始。

**QuickStart** 会保留默认值：

* 本地 Gateway（回环地址）
* 默认工作区（或现有工作区）
* Gateway 端口 **18789**
* Gateway 认证 **Token**（自动生成，即使仅使用回环地址）
* 通过 Tailscale 对外暴露 **关闭**
* Telegram + WhatsApp 私信默认使用 **允许列表**（系统会提示你输入手机号）

**Advanced** 会展示每个步骤（模式、工作区、Gateway、通道、守护进程、技能）。

<div id="what-the-wizard-does">
  ## 向导的功能
</div>

**本地模式（默认）** 将引导你完成以下步骤：

* 模型/认证（OpenAI Code（Codex）订阅 OAuth、Anthropic API key（推荐）或 setup-token（粘贴），以及 MiniMax/GLM/Moonshot/AI Gateway 选项）
* 工作区位置 + 初始化文件
* Gateway 设置（port/bind/auth/Tailscale）
* 提供方（Telegram、WhatsApp、Discord、Google Chat、Mattermost（插件）、Signal）
* 守护进程安装（LaunchAgent / systemd 用户单元）
* 健康检查
* 技能（推荐）

**远程模式** 仅配置本地客户端以连接到运行在其他位置的 Gateway。
它 **不会** 在远程主机上安装或更改任何内容。

要添加更多相互隔离的智能体（独立工作区 + 会话 + 认证），请使用：

```bash
openclaw agents add <name>
```

提示：`--json` 并**不**表示进入非交互模式。用于脚本时请使用 `--non-interactive`（以及 `--workspace`）。

<div id="flow-details-local">
  ## 流程详情（本地）
</div>

1. **现有配置检测**
   * 如果存在 `~/.openclaw/openclaw.json`，则选择 **Keep / Modify / Reset**。
   * 重新运行向导时，除非你显式选择 **Reset**
     （或传入 `--reset`），否则**不会**清空任何内容。
   * 如果配置无效或包含旧版键名，向导会停止并要求你先运行
     `openclaw doctor` 然后再继续。
   * Reset 使用 `trash`（绝不使用 `rm`），并提供以下 scope：
     * 仅配置
     * 配置 + 凭据 + 会话
     * 完全重置（同时移除工作区）

2. **模型 / 认证**
   * **Anthropic API key（推荐）**：如果存在 `ANTHROPIC_API_KEY` 就直接使用，否则会提示你输入 key，然后保存以供守护进程使用。
   * **Anthropic OAuth（Claude Code CLI）**：在 macOS 上，向导会检查钥匙串条目 &quot;Claude Code-credentials&quot;（选择 &quot;Always Allow&quot; 以避免 launchd 启动时被阻塞）；在 Linux/Windows 上，如果存在 `~/.claude/.credentials.json`，则会复用它。
   * **Anthropic token（粘贴 setup-token）**：在任意机器上运行 `claude setup-token`，然后粘贴该 token（你可以为它命名；留空 = 默认）。
   * **OpenAI Code（Codex）订阅（Codex CLI）**：如果存在 `~/.codex/auth.json`，向导可以复用它。
   * **OpenAI Code（Codex）订阅（OAuth）**：浏览器流程；粘贴 `code#state`。
     * 当模型未设置或为 `openai/*` 时，将 `agents.defaults.model` 设置为 `openai-codex/gpt-5.2`。
   * **OpenAI API key**：如果存在 `OPENAI_API_KEY` 就直接使用，否则会提示你输入 key，然后将其保存到 `~/.openclaw/.env`，以便 launchd 读取。
   * **OpenCode Zen（多模型代理）**：提示输入 `OPENCODE_API_KEY`（或 `OPENCODE_ZEN_API_KEY`，从 https://opencode.ai/auth 获取）。
   * **API key**：为你保存该 key。
   * **Vercel AI Gateway（多模型代理）**：提示输入 `AI_GATEWAY_API_KEY`。
   * 更多详情：[Vercel AI Gateway](/zh/providers/vercel-ai-gateway)
   * **MiniMax M2.1**：配置会自动生成。
   * 更多详情：[MiniMax](/zh/providers/minimax)
   * **Synthetic（兼容 Anthropic）**：提示输入 `SYNTHETIC_API_KEY`。
   * 更多详情：[Synthetic](/zh/providers/synthetic)
   * **Moonshot（Kimi K2）**：配置会自动生成。
   * **Kimi Code**：配置会自动生成。
   * 更多详情：[Moonshot AI (Kimi + Kimi Code)](/zh/providers/moonshot)
   * **Skip**：暂不配置任何认证。
   * 从已检测到的选项中选择一个默认模型（或手动输入 provider/model）。
   * 向导会运行一次模型检查，如果配置的模型未知或缺少认证，会发出警告。

* OAuth 凭据存放在 `~/.openclaw/credentials/oauth.json`；认证配置文件存放在 `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`（API keys + OAuth）。
  * 更多详情：[/concepts/oauth](/zh/concepts/oauth)

3. **Workspace**
   * 默认 `~/.openclaw/workspace`（可配置）。
   * 为智能体引导启动流程准备所需的工作区文件。
   * 完整工作区布局 + 备份指南：[Agent workspace](/zh/concepts/agent-workspace)

4. **Gateway**
   * 端口、绑定地址、认证模式、通过 Tailscale 暴露等。
   * 认证建议：即使是回环地址（loopback）也保持使用 **Token**，这样本地 WS 客户端也必须先通过认证。
   * 只有在你完全信任所有本地进程时才应禁用认证。
   * 非回环地址绑定仍然需要认证。

5. **Channels**
   * [WhatsApp](/zh/channels/whatsapp)：可选二维码登录。
   * [Telegram](/zh/channels/telegram)：bot token。
   * [Discord](/zh/channels/discord)：bot token。
   * [Google Chat](/zh/channels/googlechat)：service account JSON + webhook audience。
   * [Mattermost](/zh/channels/mattermost)（插件）：bot token + base URL。
   * [Signal](/zh/channels/signal)：可选安装 `signal-cli` + 账号配置。
   * [iMessage](/zh/channels/imessage)：本地 `imsg` CLI 路径 + 数据库访问权限。
   * 私信（DM）安全性：默认使用配对。首次私信会发送一个代码；通过 `openclaw pairing approve <channel> <code>` 来批准，或使用允许列表。

6. **Daemon install**
   * macOS：LaunchAgent
     * 需要一个已登录的用户会话；无头环境请使用自定义 LaunchDaemon（未随项目提供）。
   * Linux（以及通过 WSL2 的 Windows）：systemd 用户单元
     * 向导会尝试通过 `loginctl enable-linger <user>` 启用 lingering，以便在注销后 Gateway 仍保持运行。
     * 可能会提示需要 sudo（写入 `/var/lib/systemd/linger`）；它会先尝试在不使用 sudo 的情况下执行。
   * **运行时选择：**Node（推荐；WhatsApp/Telegram 必需）。不推荐使用 Bun。

7. **Health check**
   * 启动 Gateway（如有需要）并运行 `openclaw health`。
   * 提示：`openclaw status --deep` 会在状态输出中添加 Gateway 健康检查探针（需要 Gateway 可达）。

8. **Skills (recommended)**
   * 读取可用技能并检查其依赖。
   * 允许你选择包管理器：**npm / pnpm**（不推荐 Bun）。
   * 安装可选依赖（部分在 macOS 上通过 Homebrew 安装）。

9. **Finish**
   * 输出总结和后续步骤，包括针对 iOS/Android/macOS 的应用以获取额外功能。

* 如果未检测到 GUI，向导会打印用于访问 Control UI 的 SSH 端口转发说明，而不是直接打开浏览器。
  * 如果缺少 Control UI 资源文件，向导会尝试构建它们；回退方案是运行 `pnpm ui:build`（自动安装 UI 依赖）。

<div id="remote-mode">
  ## 远程模式
</div>

远程模式会将本地客户端配置为连接到其他位置的 Gateway。

你需要配置：

* 远程 Gateway URL（`ws://...`）
* 如果远程 Gateway 需要身份验证，则配置令牌（推荐）

注意：

* 不会执行任何远程安装或守护进程修改。
* 如果 Gateway 只绑定在本机回环地址，请使用 SSH 隧道或 tailnet。
* 服务发现提示：
  * macOS：Bonjour（`dns-sd`）
  * Linux：Avahi（`avahi-browse`）

<div id="add-another-agent">
  ## 添加另一个智能体
</div>

使用 `openclaw agents add <name>` 来创建一个拥有自己工作区、
会话和认证配置的独立智能体。不带 `--workspace` 运行时会启动向导。

该命令会设置：

* `agents.list[].name`
* `agents.list[].workspace`
* `agents.list[].agentDir`

说明：

* 默认工作区路径为 `~/.openclaw/workspace-<agentId>`。
* 添加 `bindings` 以路由入站消息（向导可以为你完成这一操作）。
* 非交互式参数：`--model`、`--agent-dir`、`--bind`、`--non-interactive`。

<div id="noninteractive-mode">
  ## 非交互式模式
</div>

使用 `--non-interactive` 在自动化流程或脚本中运行向导：

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice apiKey \
  --anthropic-api-key "$ANTHROPIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback \
  --install-daemon \
  --daemon-runtime node \
  --skip-skills
```

添加 `--json` 以输出机器可读的摘要。

Gemini 示例：

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice gemini-api-key \
  --gemini-api-key "$GEMINI_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

Z.AI 示例：

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice zai-api-key \
  --zai-api-key "$ZAI_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

Vercel AI Gateway 示例：

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice ai-gateway-api-key \
  --ai-gateway-api-key "$AI_GATEWAY_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

Moonshot 示例：

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice moonshot-api-key \
  --moonshot-api-key "$MOONSHOT_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

构造示例：

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice synthetic-api-key \
  --synthetic-api-key "$SYNTHETIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

OpenCode Zen 示例：

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice opencode-zen \
  --opencode-zen-api-key "$OPENCODE_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

添加智能体（非交互）示例：

```bash
openclaw agents add work \
  --workspace ~/.openclaw/workspace-work \
  --model openai/gpt-5.2 \
  --bind whatsapp:biz \
  --non-interactive \
  --json
```

<div id="gateway-wizard-rpc">
  ## Gateway 向导 RPC
</div>

Gateway 通过 RPC（`wizard.start`、`wizard.next`、`wizard.cancel`、`wizard.status`）提供向导流程。
客户端（macOS 应用、Control UI）可以在无需重新实现引导逻辑的情况下渲染各个步骤。

<div id="signal-setup-signal-cli">
  ## Signal 配置（signal-cli）
</div>

向导可以从 GitHub Releases 页面安装 `signal-cli`：

* 下载相应的发行包。
* 将其存储在 `~/.openclaw/tools/signal-cli/<version>/` 下。
* 将 `channels.signal.cliPath` 写入你的配置。

注意：

* JVM 构建需要 **Java 21**。
* 在可用时会使用原生构建。
* Windows 使用 WSL2；signal-cli 会在 WSL2 中按照 Linux 安装流程进行。

<div id="what-the-wizard-writes">
  ## 向导会写入的内容
</div>

`~/.openclaw/openclaw.json` 中通常包含的字段：

* `agents.defaults.workspace`
* `agents.defaults.model` / `models.providers`（如果选择了 Minimax）
* `gateway.*`（模式、绑定、认证、Tailscale）
* `channels.telegram.botToken`、`channels.discord.token`、`channels.signal.*`、`channels.imessage.*`
* 当你在向导提示中选择启用时（如果可能，名称会解析为 ID），用于 Slack/Discord/Matrix/Microsoft Teams 的频道允许列表。
* `skills.install.nodeManager`
* `wizard.lastRunAt`
* `wizard.lastRunVersion`
* `wizard.lastRunCommit`
* `wizard.lastRunCommand`
* `wizard.lastRunMode`

`openclaw agents add` 会写入 `agents.list[]` 和可选的 `bindings`。

WhatsApp 凭据存放在 `~/.openclaw/credentials/whatsapp/<accountId>/`。
会话存储在 `~/.openclaw/agents/<agentId>/sessions/` 下。

部分频道以插件形式提供。当你在初始引导过程中选择其中之一时，在配置之前，向导
会提示你先安装它（通过 npm 或本地路径）。

<div id="related-docs">
  ## 相关文档
</div>

* macOS 应用上手： [Onboarding](/zh/start/onboarding)
* 配置参考：[Gateway 配置](/zh/gateway/configuration)
* 提供方：[WhatsApp](/zh/channels/whatsapp), [Telegram](/zh/channels/telegram), [Discord](/zh/channels/discord), [Google Chat](/zh/channels/googlechat), [Signal](/zh/channels/signal), [iMessage](/zh/channels/imessage)
* 技能：[Skills](/zh/tools/skills), [技能配置](/zh/tools/skills-config)