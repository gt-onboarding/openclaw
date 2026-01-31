---
title: AGENTS.default
summary: "用于个人助理配置的默认 OpenClaw Agent 代理使用说明和技能列表"
read_when:
  - 开启新的 OpenClaw 智能体会话时
  - 启用或审查默认技能时
---

<div id="agentsmd-openclaw-personal-assistant-default">
  # AGENTS.md — OpenClaw 个人助手（默认配置）
</div>

<div id="first-run-recommended">
  ## 首次运行（推荐）
</div>

OpenClaw 会为该智能体使用一个专用的工作区目录。默认路径：`~/.openclaw/workspace`（可通过 `agents.defaults.workspace` 配置）。

1. 创建工作区（如果尚不存在）：

```bash
mkdir -p ~/.openclaw/workspace
```

2. 将默认的工作区模板复制到工作区：

```bash
cp docs/reference/templates/AGENTS.md ~/.openclaw/workspace/AGENTS.md
cp docs/reference/templates/SOUL.md ~/.openclaw/workspace/SOUL.md
cp docs/reference/templates/TOOLS.md ~/.openclaw/workspace/TOOLS.md
```

3. 可选：如果你想使用个人助理技能列表，请将 AGENTS.md 替换为此文件：

```bash
cp docs/reference/AGENTS.default.md ~/.openclaw/workspace/AGENTS.md
```

4. 可选：通过设置 `agents.defaults.workspace` 来选择其他工作区（支持 `~`）：

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } }
}
```


<div id="safety-defaults">
  ## 默认安全策略
</div>

- 不要在聊天中直接输出整个目录或任何机密信息。
- 未被明确要求时，不要运行具有破坏性的命令。
- 不要向外部消息渠道发送不完整/流式回复（只发送最终回复）。

<div id="session-start-required">
  ## 会话开始（必做）
</div>

- 阅读 `SOUL.md`、`USER.md`、`memory.md`，以及 `memory/` 目录中今天和昨天的内容。
- 在回复之前完成上述操作。

<div id="soul-required">
  ## Soul（必需）
</div>

- `SOUL.md` 定义了身份、语气和边界，请保持其内容为最新状态。
- 如果你更改了 `SOUL.md`，要告知用户。
- 你在每次会话中都是一个全新的实例；会话连续性由这些文件来维持。

<div id="shared-spaces-recommended">
  ## 共享空间（推荐）
</div>

- 你不是用户的代言人；在群聊或公共频道中要格外谨慎。
- 不要分享敏感或个人数据、联系方式或内部备注。

<div id="memory-system-recommended">
  ## 记忆系统（推荐）
</div>

- 每日日志：`memory/YYYY-MM-DD.md`（如有需要，先创建 `memory/`）。
- 长期记忆：`memory.md`，用于存储持久的事实、偏好和决策。
- 在会话开始时，读取今天和昨天的日志以及 `memory.md`（如存在）。
- 记录：决策、偏好、约束、未完结事项（open loops）。
- 避免存储敏感/机密信息，除非用户明确要求。

<div id="tools-skills">
  ## 工具与技能
</div>

- 工具隶属于各个技能；需要时查阅每个技能的 `SKILL.md`。
- 将与环境相关的说明记录在 `TOOLS.md`（技能说明）中。

<div id="backup-tip-recommended">
  ## 备份提示（推荐）
</div>

如果你把这个工作区视作 Clawd 的“记忆”，建议将其初始化为一个 git 仓库（最好是私有仓库），这样 `AGENTS.md` 和你的记忆文件就能得到备份。

```bash
cd ~/.openclaw/workspace
git init
git add AGENTS.md
git commit -m "Add Clawd workspace"
# 可选:添加私有远程仓库并推送
```


<div id="what-openclaw-does">
  ## OpenClaw 的功能
</div>

- 运行 WhatsApp Gateway + Pi 编程智能体，这样助手就可以读取/写入聊天、获取上下文，并通过宿主 Mac 运行技能。
- macOS 应用管理权限（屏幕录制、通知、麦克风），并通过其内置二进制文件提供 `openclaw` CLI。
- 私聊默认汇总到智能体的 `main` 会话；群聊则保持隔离，使用 `agent:<agentId>:<channel>:group:<id>`（房间/频道：`agent:<agentId>:<channel>:channel:<id>`）；心跳会让后台任务保持存活。

<div id="core-skills-enable-in-settings-skills">
  ## 核心技能（在 Settings → Skills 中启用）
</div>

- **mcporter** — 用于管理外部技能后端的工具服务器运行时/CLI。
- **Peekaboo** — 快速截取 macOS 屏幕截图，可选启用 AI 视觉分析。
- **camsnap** — 从 RTSP/ONVIF 安防摄像头捕获画面、短片或运动告警。
- **oracle** — 支持 OpenAI 的 Agent CLI，包含会话回放和浏览器控制功能。
- **eightctl** — 从终端控制你的睡眠。
- **imsg** — 发送、read、流式传输 iMessage 和 SMS。
- **wacli** — WhatsApp CLI：同步、搜索、发送。
- **discord** — Discord 操作：表情反应、贴纸、投票。使用 `user:<id>` 或 `channel:<id>` 作为目标（仅使用纯数字 ID 会产生歧义）。
- **gog** — Google Suite CLI：Gmail、Calendar、Drive、Contacts。
- **spotify-player** — 终端版 Spotify 客户端，用于搜索/排队/控制播放。
- **sag** — ElevenLabs 语音，具 mac 风格的 `say` 体验；默认将音频流式输出到扬声器。
- **Sonos CLI** — 通过脚本控制 Sonos 音箱（发现/状态/播放/音量/分组）。
- **blucli** — 通过脚本播放、分组和自动化 BluOS 播放器。
- **OpenHue CLI** — 用于场景和自动化的 Philips Hue 灯光控制。
- **OpenAI Whisper** — 本地语音转文本，用于快速听写和语音信箱转录。
- **Gemini CLI** — 在终端使用 Google Gemini 模型进行快速问答。
- **bird** — X/Twitter CLI，无需浏览器即可发推、回复、阅读推文串和搜索。
- **agent-tools** — 用于自动化和辅助脚本的通用实用工具包。

<div id="usage-notes">
  ## 使用说明
</div>

- 编写脚本时优先使用 `openclaw` CLI；macOS 应用会处理权限。
- 在 Skills 选项卡中执行安装；如果二进制文件已存在，它会隐藏安装按钮。
- 保持心跳功能启用，这样助手可以安排提醒、监控收件箱并触发摄像头捕获。
- Canvas UI 以全屏方式运行并使用原生覆盖层。避免将关键控件放在左上角 / 右上角 / 底部边缘；在布局中添加明确的留白，不要依赖安全区域内边距（safe-area insets）。
- 对于基于浏览器的验证，使用 `openclaw browser`（tabs/status/screenshot）配合由 OpenClaw 管理的 Chrome 用户配置文件。
- 对于 DOM 检查，使用 `openclaw browser eval|query|dom|snapshot`（需要机器可读输出时配合 `--json`/`--out`）。
- 对于交互，使用 `openclaw browser click|type|hover|drag|select|upload|press|wait|navigate|back|evaluate|run`（click/type 需要快照引用；对 CSS 选择器使用 `evaluate`）。