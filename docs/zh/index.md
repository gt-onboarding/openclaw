---
title: Index
summary: "Top-level overview of OpenClaw, features, and purpose"
read_when:
  - Introducing OpenClaw to newcomers
---

<div id="openclaw">
  # OpenClaw 🦞
</div>

> _"EXFOLIATE! EXFOLIATE!"_ — A space lobster, probably

<p align="center">
    <picture>
        <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/openclaw/openclaw/main/docs/assets/openclaw-logo-text-dark.png">
        <img src="https://raw.githubusercontent.com/openclaw/openclaw/main/docs/assets/openclaw-logo-text.png" alt="OpenClaw" width="500">
    </picture>
</p>

<p align='center'>
  <strong>
    Any OS + WhatsApp/Telegram/Discord/iMessage gateway for AI agents (Pi).
  </strong>
  <br />
  Plugins add Mattermost and more. Send a message, get an agent response — from
  your pocket.
</p>

<p align='center'>
  <a href='https://github.com/openclaw/openclaw'>GitHub</a> ·
  <a href='https://github.com/openclaw/openclaw/releases'>Releases</a> ·
  <a href='/'>Docs</a> ·<a href='/start/openclaw'>OpenClaw assistant setup</a>
</p>

OpenClaw 将 WhatsApp(通过 WhatsApp Web / Baileys)、Telegram(Bot API / grammY)、Discord(Bot API / channels.discord.js)和 iMessage(imsg CLI)桥接到编程智能体,例如 [Pi](https://github.com/badlogic/pi-mono)。插件支持 Mattermost(Bot API + WebSocket)等更多平台。
OpenClaw 还为 OpenClaw 助手提供支持。


<div id="start-here">
  ## 从这里开始
</div>

- **从零全新安装：** [快速上手](/start/getting-started)
- **引导式安装（推荐）：** [向导](/start/wizard) (`openclaw onboard`)
- **打开控制面板（本地 Gateway）：** http://127.0.0.1:18789/（或 http://localhost:18789/）

如果 Gateway 在同一台电脑上运行，点击该链接会立即在浏览器中打开 Control UI。  
如果打不开，请先启动 Gateway：`openclaw gateway`。



<div id="dashboard-browser-control-ui">
  ## 仪表盘（浏览器 Control UI）
</div>

仪表盘是用于聊天、配置、节点、会话等的浏览器 Control UI。
本地默认访问地址：http://127.0.0.1:18789/
远程访问方式：[Web 界面](/web) 和 [Tailscale](/gateway/tailscale)

<p align="center">
  <img src="whatsapp-openclaw.jpg" alt="OpenClaw" width="420" />
</p>



<div id="how-it-works">
  ## 工作原理
</div>

```
WhatsApp / Telegram / Discord / iMessage (+ plugins)
        │
        ▼
  ┌───────────────────────────┐
  │          Gateway          │  ws://127.0.0.1:18789 (loopback-only)
  │     (single source)       │
  │                           │  http://<gateway-host>:18793
  │                           │    /__openclaw__/canvas/ (Canvas 主机)
  └───────────┬───────────────┘
              │
              ├─ Pi agent (RPC)
              ├─ CLI (openclaw …)
              ├─ Chat UI (SwiftUI)
              ├─ macOS app (OpenClaw.app)
              ├─ iOS node via Gateway WS + pairing
              └─ Android node via Gateway WS + pairing
```

大多数操作都通过 **Gateway**（`openclaw gateway`）进行，它是一个单个常驻进程，负责管理通道连接和 WebSocket 控制平面。


<div id="network-model">
  ## 网络模型
</div>

- **每个主机一个 Gateway（推荐）**：它是唯一被允许持有 WhatsApp Web 会话的进程。如果你需要应急 bot 或严格隔离，请运行多个 Gateway，并为它们配置隔离的配置文件和端口；参见 [Multiple gateways](/gateway/multiple-gateways)。
- **优先使用回环地址（Loopback-first）**：Gateway 的 WS 默认地址为 `ws://127.0.0.1:18789`。
  - 向导现在默认会生成一个 gateway token（即使是回环地址也会如此）。
  - 对于 Tailnet 访问，运行 `openclaw gateway --bind tailnet --token ...`（非回环绑定必须提供 token）。
- **节点（Nodes）**：通过 WebSocket 连接到 Gateway（按需使用 LAN/Tailnet/SSH）；旧版 TCP bridge 已弃用并移除。
- **Canvas 主机**：在 `canvasHost.port`（默认 `18793`）上运行 HTTP 文件服务器，为节点 WebView 提供 `/__openclaw__/canvas/`；参见 [Gateway configuration](/gateway/configuration)（`canvasHost`）。
- **远程使用**：通过 SSH 隧道或 Tailnet/VPN；参见 [Remote access](/gateway/remote) 和 [Discovery](/gateway/discovery)。



<div id="features-high-level">
  ## 功能概览（高层级）
</div>

- 📱 **WhatsApp 集成** — 使用 Baileys 实现 WhatsApp Web 协议
- ✈️ **Telegram 机器人** — 通过 grammY 支持私信和群组
- 🎮 **Discord 机器人** — 通过 channels.discord.js 支持私信和服务器频道
- 🧩 **Mattermost 机器人（插件）** — Bot token 与 WebSocket 事件集成
- 💬 **iMessage** — 本地 imsg CLI 集成（macOS）
- 🤖 **Agent bridge** — 基于 Pi 的 Agent 桥接（RPC 模式），支持工具流式传输
- ⏱️ **流式传输与分块** — 块级流式输出 + Telegram 草稿流式传输细节（[/concepts/streaming](/concepts/streaming)）
- 🧠 **多智能体路由** — 将提供方账号/对端路由到隔离的智能体（工作区 + 每智能体会话）
- 🔐 **订阅认证** — 通过 OAuth 集成 Anthropic（Claude Pro/Max）与 OpenAI（ChatGPT/Codex）
- 💬 **会话** — 直接聊天会折叠到共享的 `main`（默认）；群组会话相互隔离
- 👥 **群聊支持** — 默认基于提及；所有者可切换 `/activation always|mention`
- 📎 **媒体支持** — 发送与接收图片、音频、文档
- 🎤 **语音留言** — 可选转录 hook
- 🖥️ **WebChat + macOS 应用** — 本地 UI + 菜单栏助手，用于运维与语音唤醒
- 📱 **iOS 节点** — 作为节点配对并暴露 Canvas 画布界面
- 📱 **Android 节点** — 作为节点配对并暴露 Canvas + Chat + Camera 能力

注意：旧版 Claude/Codex/Gemini/Opencode 路径已被移除；Pi 是唯一的编码智能体路径。



<div id="quick-start">
  ## 快速开始
</div>

运行环境要求：**Node.js ≥ 22**。



```bash
# 推荐：全局安装（npm/pnpm）
npm install -g openclaw@latest
# 或：pnpm add -g openclaw@latest
```


# 初始化并安装服务（launchd/systemd 用户服务）
openclaw onboard --install-daemon



# 与 WhatsApp Web 配对（显示二维码）
openclaw channels login



# 完成初始引导后，Gateway 将通过服务运行；你仍然可以手动运行：

openclaw gateway --port 18789

````

后续在 npm 和 git 安装方式之间切换很简单:安装另一种方式,然后运行 `openclaw doctor` 更新 Gateway 服务入口点。

从源代码安装(开发):

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # 首次运行时自动安装 UI 依赖
pnpm build
openclaw onboard --install-daemon
````

如果你还没有进行全局安装，请在代码仓库中通过 `pnpm openclaw ...` 运行入门步骤。

多实例快速入门（可选）：

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

发送一条测试消息（需要 Gateway 正在运行）：

```bash
openclaw message send --target +15555550123 --message "Hello from OpenClaw"
```


<div id="credits">
  ## 配置（可选）
</div>

配置文件位于 `~/.openclaw/openclaw.json`。

* 如果你**什么都不配置**，OpenClaw 会在 RPC 模式下使用内置的 Pi 可执行文件，并按发送方创建独立会话。
* 如果你想收紧访问控制，可以从配置 `channels.whatsapp.allowFrom` 开始，并为（群组）设置 @ 提及规则。

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


<div id="core-contributors">
  ## 文档
</div>

- 从这里开始：
  - [文档中心（所有页面索引）](/start/hubs)
  - [帮助](/help) ← *常见修复与故障排查*
  - [配置](/gateway/configuration)
  - [配置示例](/gateway/configuration-examples)
  - [斜杠命令](/tools/slash-commands)
  - [多智能体路由](/concepts/multi-agent)
  - [更新 / 回滚](/install/updating)
  - [配对（私信 + 节点）](/start/pairing)
  - [Nix 模式](/install/nix)
  - [OpenClaw 助手设置](/start/openclaw)
  - [技能](/tools/skills)
  - [技能配置](/tools/skills-config)
  - [工作区模板](/reference/templates/AGENTS)
  - [RPC 适配器](/reference/rpc)
  - [Gateway 运行手册](/gateway)
  - [节点（iOS/Android）](/nodes)
  - [Web 界面（Control UI）](/web)
  - [发现与传输](/gateway/discovery)
  - [远程访问](/gateway/remote)
- 提供方与体验：
  - [WebChat](/web/webchat)
  - [Control UI（浏览器）](/web/control-ui)
  - [Telegram](/channels/telegram)
  - [Discord](/channels/discord)
  - [Mattermost（插件）](/channels/mattermost)
  - [iMessage](/channels/imessage)
  - [群组](/concepts/groups)
  - [WhatsApp 群消息](/concepts/group-messages)
  - [媒体：图像](/nodes/images)
  - [媒体：音频](/nodes/audio)
- 配套应用：
  - [macOS 应用](/platforms/macos)
  - [iOS 应用](/platforms/ios)
  - [Android 应用](/platforms/android)
  - [Windows（WSL2）](/platforms/windows)
  - [Linux 应用](/platforms/linux)
- 运维与安全：
  - [会话](/concepts/session)
  - [Cron 任务](/automation/cron-jobs)
  - [Webhooks](/automation/webhook)
  - [Gmail 钩子（Pub/Sub）](/automation/gmail-pubsub)
  - [安全](/gateway/security)
  - [故障排查](/gateway/troubleshooting)



<div id="license">
  ## 名称的由来
</div>

**OpenClaw = CLAW + TARDIS** —— 因为每只太空龙虾都需要一台时空机。

---

*“我们都只是在摆弄各自的提示词。”* —— 某个大概在 token 上嗨过头的 AI



## 鸣谢

- **Peter Steinberger** ([@steipete](https://twitter.com/steipete)) — 创建者，龙虾低语者
- **Mario Zechner** ([@badlogicc](https://twitter.com/badlogicgames)) — Pi 创建者，安全渗透测试工程师
- **Clawd** — 那只要求取个更好名字的太空龙虾



## 核心贡献者

- **Maxim Vovshin** (@Hyaxia, 36747317+Hyaxia@users.noreply.github.com) — Blogwatcher 技能
- **Nacho Iacovino** (@nachoiacovino, nacho.iacovino@gmail.com) — 位置解析（Telegram + WhatsApp）



## 许可协议

MIT — 像大海里的龙虾一样自由 🦞

---

*“我们其实都只是在玩各自的提示词。”* — 某个 AI，大概是 token 嗑多了
