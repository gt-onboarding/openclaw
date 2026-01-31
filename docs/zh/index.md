---
title: 索引
summary: "OpenClaw 的整体概览、功能和用途"
read_when:
  - 向新用户介绍 OpenClaw
---

<div id="openclaw">
  # OpenClaw 🦞
</div>

> *“去角质！去角质！”* —— 大概是某只太空龙虾

<p align="center">
  <picture>
    <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/openclaw/openclaw/main/docs/assets/openclaw-logo-text-dark.png" />

    <img src="https://raw.githubusercontent.com/openclaw/openclaw/main/docs/assets/openclaw-logo-text.png" alt="OpenClaw" width="500" />
  </picture>
</p>

<p align="center">
  <strong>适用于任意操作系统（包括树莓派）的 WhatsApp/Telegram/Discord/iMessage AI Agent 代理 Gateway。</strong><br />
  通过插件可扩展到 Mattermost 等更多平台。
  发送一条消息，即可获得智能体的响应——一切尽在你的口袋中。
</p>

<p align="center">
  <a href="https://github.com/openclaw/openclaw">GitHub</a> ·
  <a href="https://github.com/openclaw/openclaw/releases">Releases</a> ·
  <a href="/zh/">文档 Docs</a> ·
  <a href="/zh/start/openclaw">OpenClaw 助手设置</a>
</p>

OpenClaw 将 WhatsApp（通过 WhatsApp Web / Baileys）、Telegram（Bot API / grammY）、Discord（Bot API / channels.discord.js）和 iMessage（imsg CLI）桥接到像 [Pi](https://github.com/badlogic/pi-mono) 这样的编程智能体。插件可以添加 Mattermost（Bot API + WebSocket）以及更多平台。
OpenClaw 同时为 OpenClaw 助手提供底层支撑。

<div id="start-here">
  ## 从这里开始
</div>

* **从零开始全新安装：** [快速上手](/zh/start/getting-started)
* **引导式配置（推荐）：** [向导](/zh/start/wizard)（`openclaw onboard`）
* **打开仪表盘（本地 Gateway）：** http://127.0.0.1:18789/（或 http://localhost:18789/）

如果 Gateway 正在同一台电脑上运行，该链接会立即在浏览器中打开 Control UI。
如果打不开，请先启动 Gateway：`openclaw gateway`。

<div id="dashboard-browser-control-ui">
  ## 仪表板（浏览器 Control UI）
</div>

仪表板是用于管理聊天、配置、节点、会话等的基于浏览器的 Control UI。
本地默认地址：http://127.0.0.1:18789/
远程访问：[Web 界面](/zh/web) 和 [Tailscale](/zh/gateway/tailscale)

<p align="center">
  <img src="/whatsapp-openclaw.jpg" alt="OpenClaw" width="420" />
</p>

<div id="how-it-works">
  ## 工作原理
</div>

```
WhatsApp / Telegram / Discord / iMessage (+ 插件)
        │
        ▼
  ┌───────────────────────────┐
  │          Gateway          │  ws://127.0.0.1:18789 (loopback-only)
  │     (单一来源)            │
  │                           │  http://<gateway-host>:18793
  │                           │    /__openclaw__/canvas/ (Canvas 主机)
  └───────────┬───────────────┘
              │
              ├─ Pi 智能体 (RPC)
              ├─ CLI (openclaw …)
              ├─ Chat UI (SwiftUI)
              ├─ macOS app (OpenClaw.app)
              ├─ iOS 节点,通过 Gateway WS + 配对
              └─ Android 节点,通过 Gateway WS + 配对
```

大多数操作都通过 **Gateway**（`openclaw gateway`）进行，这是一个长期运行的单一常驻进程，负责管理通道连接和 WebSocket 控制平面。

<div id="network-model">
  ## 网络模型
</div>

* **每台主机一个 Gateway（推荐）**：它是唯一被允许持有 WhatsApp Web 会话的进程。如果你需要应急 bot 或严格隔离，请使用隔离的配置文件和端口运行多个 Gateway；参见 [Multiple gateways](/zh/gateway/multiple-gateways)。
* **优先回环地址（Loopback-first）**：Gateway 的 WS 默认使用 `ws://127.0.0.1:18789`。
  * 向导现在默认会生成一个 gateway token（即使是回环地址也会生成）。
  * 对于 Tailnet 访问，运行 `openclaw gateway --bind tailnet --token ...`（非回环绑定必须提供 token）。
* **节点**：根据需要通过 WebSocket（LAN/Tailnet/SSH）连接到 Gateway；旧版 TCP 桥接已弃用/移除。
* **Canvas 主机**：在 `canvasHost.port`（默认 `18793`）上提供 HTTP 文件服务，为节点的 WebView 提供 `/__openclaw__/canvas/`；参见 [Gateway configuration](/zh/gateway/configuration)（`canvasHost`）。
* **远程使用**：通过 SSH 隧道或 Tailnet/VPN；参见 [Remote access](/zh/gateway/remote) 和 [Discovery](/zh/gateway/discovery)。

<div id="features-high-level">
  ## 功能（高层概览）
</div>

* 📱 **WhatsApp 集成** — 使用 Baileys 实现 WhatsApp Web 协议
* ✈️ **Telegram 机器人** — 通过 grammY 支持私信和群组
* 🎮 **Discord 机器人** — 通过 channels.discord.js 支持私信和公会频道
* 🧩 **Mattermost 机器人（插件）** — 机器人令牌 + WebSocket 事件
* 💬 **iMessage** — 本地 imsg CLI 集成（macOS）
* 🤖 **Agent 桥接** — Pi（RPC 模式），支持工具流式处理
* ⏱️ **流式输出 + 分块** — 块级流式输出 + Telegram 草稿流式输出细节（[/concepts/streaming](/zh/concepts/streaming)）
* 🧠 **多智能体路由** — 将提供方账户/对等方路由到隔离的智能体（工作区 + 每个智能体独立会话）
* 🔐 **订阅认证** — Anthropic（Claude Pro/Max）+ OpenAI（ChatGPT/Codex），通过 OAuth
* 💬 **会话** — 直接私聊合并到共享的 `main`（默认）；群组会话彼此隔离
* 👥 **群聊支持** — 默认基于 @ 提及；所有者可以切换 `/activation always|mention`
* 📎 **媒体支持** — 发送和接收图片、音频、文档
* 🎤 **语音消息** — 可选的转录 hook（钩子）
* 🖥️ **WebChat + macOS 应用** — 本地 UI + 菜单栏助手，用于运维和语音唤醒
* 📱 **iOS 节点** — 配对为节点，并提供 Canvas 画布
* 📱 **Android 节点** — 配对为节点，并提供 Canvas 画布 + Chat 聊天 + Camera 相机

注意：旧版 Claude/Codex/Gemini/Opencode 集成路径已移除；Pi 是唯一的代码类智能体路径。

<div id="quick-start">
  ## 快速开始
</div>

运行环境要求：**Node.js ≥ 22**。

```bash
# Recommended: global install (npm/pnpm)
npm install -g openclaw@latest
# or: pnpm add -g openclaw@latest

# Onboard + install the service (launchd/systemd user service)
openclaw onboard --install-daemon

# Pair WhatsApp Web (shows QR)
openclaw channels login

# 初始化后 Gateway 通过服务运行；也可手动运行：
openclaw gateway --port 18789
```

之后在 npm 安装和 git 安装之间切换非常简单：只需安装另一种安装方式的版本，然后运行 `openclaw doctor` 来更新 Gateway 服务的入口点。

从源码安装（开发环境）：

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # 首次运行时自动安装 UI 依赖项
pnpm build
openclaw onboard --install-daemon
```

如果你尚未进行全局安装，请在仓库目录中通过 `pnpm openclaw ...` 运行入门步骤。

多实例快速开始（可选）：

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

发送一条测试消息（需确保 Gateway 正在运行）：

```bash
openclaw message send --target +15555550123 --message "Hello from OpenClaw"
```

<div id="configuration-optional">
  ## 配置（可选）
</div>

配置文件位于 `~/.openclaw/openclaw.json`。

* 如果你**什么都不做**，OpenClaw 会在 RPC 模式下使用内置的 Pi 二进制可执行文件，并为每个发送方创建独立会话。
* 如果你想收紧权限，请从 `channels.whatsapp.allowFrom` 开始配置，并为群组设置提及规则。

示例：

```json5
{
  channels: {
    whatsapp: {
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } }
    }
  },
  messages: { groupChat: { mentionPatterns: ["@openclaw"] } }
}
```

<div id="docs">
  ## 文档
</div>

* 从这里开始：
  * [文档中心（所有页面链接）](/zh/start/hubs)
  * [帮助](/zh/help) ← *常见修复与故障排查*
  * [配置](/zh/gateway/configuration)
  * [配置示例](/zh/gateway/configuration-examples)
  * [斜杠命令](/zh/tools/slash-commands)
  * [多智能体路由](/zh/concepts/multi-agent)
  * [更新 / 回滚](/zh/install/updating)
  * [配对（DM + 节点）](/zh/start/pairing)
  * [Nix 模式](/zh/install/nix)
  * [OpenClaw 助手设置](/zh/start/openclaw)
  * [技能](/zh/tools/skills)
  * [技能配置](/zh/tools/skills-config)
  * [工作区模板](/zh/reference/templates/AGENTS)
  * [RPC 适配器](/zh/reference/rpc)
  * [Gateway 运行手册](/zh/gateway)
  * [节点（iOS/Android）](/zh/nodes)
  * [Web 界面（Control UI）](/zh/web)
  * [发现与传输](/zh/gateway/discovery)
  * [远程访问](/zh/gateway/remote)
* 提供方与体验：
  * [WebChat](/zh/web/webchat)
  * [Control UI（浏览器）](/zh/web/control-ui)
  * [Telegram](/zh/channels/telegram)
  * [Discord](/zh/channels/discord)
  * [Mattermost（插件）](/zh/channels/mattermost)
  * [iMessage](/zh/channels/imessage)
  * [群组](/zh/concepts/groups)
  * [WhatsApp 群消息](/zh/concepts/group-messages)
  * [媒体：图片](/zh/nodes/images)
  * [媒体：音频](/zh/nodes/audio)
* 配套应用：
  * [macOS 应用](/zh/platforms/macos)
  * [iOS 应用](/zh/platforms/ios)
  * [Android 应用](/zh/platforms/android)
  * [Windows（WSL2）](/zh/platforms/windows)
  * [Linux 应用](/zh/platforms/linux)
* 运维与安全：
  * [会话](/zh/concepts/session)
  * [Cron 任务](/zh/automation/cron-jobs)
  * [Webhooks](/zh/automation/webhook)
  * [Gmail 钩子（Pub/Sub）](/zh/automation/gmail-pubsub)
  * [安全](/zh/gateway/security)
  * [故障排查](/zh/gateway/troubleshooting)

<div id="the-name">
  ## 名称由来
</div>

**OpenClaw = CLAW + TARDIS** —— 因为每只太空龙虾都需要一台时空机器。

***

*“我们其实都只是在摆弄各自的提示词。”* —— 某个 AI，大概是在被 token 喂嗨的时候说的

<div id="credits">
  ## 致谢
</div>

* **Peter Steinberger** ([@steipete](https://twitter.com/steipete)) — 创建者、龙虾低语者
* **Mario Zechner** ([@badlogicc](https://twitter.com/badlogicgames)) — Pi 创建者、安全渗透测试专家
* **Clawd** — 那只坚持要有个更好名字的太空龙虾

<div id="core-contributors">
  ## 核心贡献者
</div>

* **Maxim Vovshin** (@Hyaxia, 36747317+Hyaxia@users.noreply.github.com) — Blogwatcher 技能
* **Nacho Iacovino** (@nachoiacovino, nacho.iacovino@gmail.com) — 位置信息解析（Telegram + WhatsApp）

<div id="license">
  ## 许可证
</div>

MIT — 像海里的龙虾一样自由 🦞

***

*“我们都只是在玩各自的提示词。”* —— 某个大概被 token 喂嗨了的 AI