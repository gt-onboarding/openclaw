---
title: Doctor
summary: "Doctor 命令：健康检查、配置迁移和修复步骤"
read_when:
  - 添加或修改 Doctor 迁移
  - 引入不向后兼容的配置变更
---

<div id="doctor">
  # Doctor
</div>

`openclaw doctor` 是 OpenClaw 的修复和迁移工具。它用于修复过期的配置和状态、检查运行状况，并提供可操作的修复步骤。

<div id="quick-start">
  ## 快速上手
</div>

```bash
openclaw doctor
```

<div id="headless-automation">
  ### 无头模式 / 自动化
</div>

```bash
openclaw doctor --yes
```

在适用情况下（包括重启/服务/沙箱修复步骤），自动接受默认设置，无需交互确认。

```bash
openclaw doctor --repair
```

在安全的情况下，自动应用推荐的修复操作（包括修复和重启），无需确认。

```bash
openclaw doctor --repair --force
```

同时执行更强力的修复操作（会覆盖自定义的 Supervisor 配置）。

```bash
openclaw doctor --non-interactive
```

在无交互模式下运行，仅应用安全迁移（配置规范化 + 磁盘上状态迁移）。跳过所有需要人工确认的重启 / 服务 / 沙箱操作。
检测到遗留状态时，将自动执行相应的状态迁移。

```bash
openclaw doctor --deep
```

扫描系统服务中是否存在额外部署的 Gateway 实例（launchd/systemd/schtasks）。

如果你想在写入更改之前先查看修改内容，请先打开配置文件：

```bash
cat ~/.openclaw/openclaw.json
```

<div id="what-it-does-summary">
  ## 它做什么（概要）
</div>

* 针对 git 安装的可选预检更新（仅交互模式）。
* UI 协议版本检查（当协议 schema 更新时会重新构建 Control UI）。
* 健康检查 + 重启提示。
* 技能状态汇总（可用/缺失/被阻止）。
* 针对遗留值的配置规范化。
* OpenCode Zen 提供方覆盖警告（`models.providers.opencode`）。
* 旧版磁盘状态迁移（会话/智能体目录/WhatsApp 认证）。
* 状态完整性和权限检查（会话、对话记录、状态目录）。
* 本地运行时的配置文件权限检查（chmod 600）。
* 模型认证健康检查：校验 OAuth 过期时间，可刷新将要过期的 token，并报告认证配置的冷却/禁用状态。
* 额外工作区目录检测（`~/openclaw`）。
* 启用沙箱时的沙箱镜像修复。
* 旧版服务迁移和多余 Gateway 实例检测。
* Gateway 运行时检查（服务已安装但未运行；缓存的 launchd label）。
* 渠道状态警告（从正在运行的 Gateway 探测）。
* 服务管理器配置审计（launchd/systemd/schtasks），可选自动修复。
* Gateway 运行时最佳实践检查（Node vs Bun，版本管理器路径）。
* Gateway 端口冲突诊断（默认 `18789`）。
* 针对 open DM 策略的安全警告。
* 当未设置 `gateway.auth.token` 时的 Gateway 认证警告（本地模式；可协助生成 token）。
* Linux 上的 systemd linger 检查。
* 源码安装检查（pnpm 工作区不匹配、缺失 UI 资源、缺失 tsx 可执行文件）。
* 写入更新后的配置和向导元数据。

<div id="detailed-behavior-and-rationale">
  ## 详细行为与设计考量
</div>

<div id="0-optional-update-git-installs">
  ### 0) 可选更新（git 安装版本）
</div>

如果这是一个通过 git 检出的版本，并且 doctor 以交互方式运行，它会在运行 doctor 之前提示你先执行更新（fetch/rebase/build）。

<div id="1-config-normalization">
  ### 1) 配置规范化
</div>

如果配置中包含旧版的值结构形式（例如仅有 `messages.ackReaction`
而没有为各个渠道提供单独覆盖的配置），doctor 会将它们归一化为当前的
schema。

<div id="2-legacy-config-key-migrations">
  ### 2) 旧版配置键迁移
</div>

当配置中包含已弃用的键时，其他命令将拒绝运行，并提示你先运行
`openclaw doctor`。

Doctor 将会：

* 说明发现了哪些旧版配置键。
* 展示它应用的迁移操作。
* 使用更新后的 schema 重写 `~/.openclaw/openclaw.json`。

Gateway 在启动时如果检测到旧版配置格式，也会自动运行 doctor 迁移，
因此过时配置会在无需人工干预的情况下自动修复。

当前迁移包括：

* `routing.allowFrom` → `channels.whatsapp.allowFrom`
* `routing.groupChat.requireMention` → `channels.whatsapp/telegram/imessage.groups."*".requireMention`
* `routing.groupChat.historyLimit` → `messages.groupChat.historyLimit`
* `routing.groupChat.mentionPatterns` → `messages.groupChat.mentionPatterns`
* `routing.queue` → `messages.queue`
* `routing.bindings` → 顶层 `bindings`
* `routing.agents`/`routing.defaultAgentId` → `agents.list` + `agents.list[].default`
* `routing.agentToAgent` → `tools.agentToAgent`
* `routing.transcribeAudio` → `tools.media.audio.models`
* `bindings[].match.accountID` → `bindings[].match.accountId`
* `identity` → `agents.list[].identity`
* `agent.*` → `agents.defaults` + `tools.*`（tools/elevated/exec/sandbox/subagents）
* `agent.model`/`allowedModels`/`modelAliases`/`modelFallbacks`/`imageModelFallbacks`
  → `agents.defaults.models` + `agents.defaults.model.primary/fallbacks` + `agents.defaults.imageModel.primary/fallbacks`

<div id="2b-opencode-zen-provider-overrides">
  ### 2b) OpenCode Zen 提供方覆写
</div>

如果你手动添加了 `models.providers.opencode`（或 `opencode-zen`），它会
覆写来自 `@mariozechner/pi-ai` 的内置 OpenCode Zen 目录。这可能会
强制所有模型都使用单一 API，或者将成本清零。Doctor 会发出警告，方便你
移除该覆写并恢复按模型划分的 API 路由和成本。

<div id="3-legacy-state-migrations-disk-layout">
  ### 3) 旧有状态迁移（磁盘目录布局）
</div>

Doctor 可以将旧版磁盘布局迁移到当前结构：

* 会话存储 + 对话记录：
  * 从 `~/.openclaw/sessions/` 迁移到 `~/.openclaw/agents/<agentId>/sessions/`
* Agent 代理目录：
  * 从 `~/.openclaw/agent/` 迁移到 `~/.openclaw/agents/<agentId>/agent/`
* WhatsApp 认证状态（Baileys）：
  * 从旧路径 `~/.openclaw/credentials/*.json`（`oauth.json` 除外）
  * 迁移到 `~/.openclaw/credentials/whatsapp/<accountId>/...`（默认账户 ID：`default`）

这些迁移以尽力为原则，并且是幂等的；如果保留了任何旧目录作为备份，doctor 会发出警告。Gateway/CLI 也会在启动时自动迁移旧会话和 Agent 目录，从而使历史记录 / 认证信息 / 模型都存放到按智能体划分的路径中，而无需手动运行 doctor。WhatsApp 认证则特意只通过 `openclaw doctor` 进行迁移。

<div id="4-state-integrity-checks-session-persistence-routing-and-safety">
  ### 4) 状态完整性检查（会话持久化、路由与安全）
</div>

state 目录是运行时的“脑干”。如果它消失，你会丢失会话、凭据、日志和配置（除非你在其他地方有备份）。

doctor 会检查：

* **缺少 state 目录**：警告将发生灾难性的状态丢失，提示你重新创建该目录，并提醒无法恢复已丢失的数据。
* **state 目录权限**：校验目录是否可写；提供修复权限的选项（并在检测到所有者/用户组不匹配时给出 `chown` 提示）。
* **缺少会话目录**：`sessions/` 和会话存储目录是必须的，用于持久化历史并避免 `ENOENT` 崩溃。
* **转录不匹配**：当最近的会话条目缺少对应的转录文件时发出警告。
* **主会话“单行 JSONL”**：当主会话的转录文件只有一行时会标记出来（历史没有持续累积）。
* **多个 state 目录**：当在不同的 home 目录下存在多个 `~/.openclaw` 文件夹，或 `OPENCLAW_STATE_DIR` 指向其他位置时发出警告（历史可能在不同安装之间被分裂）。
* **远程模式提醒**：如果 `gateway.mode=remote`，doctor 会提醒你在远程主机上运行（state 存在于远程主机上）。
* **配置文件权限**：如果 `~/.openclaw/openclaw.json` 对用户组/所有用户可读，会发出警告，并提供将权限收紧到 `600` 的选项。

<div id="5-model-auth-health-oauth-expiry">
  ### 5) 模型认证健康状况（OAuth 过期）
</div>

Doctor 会检查认证存储中的 OAuth 配置档案，在令牌即将过期或已过期时发出警告，并在安全的情况下尝试刷新它们。如果 Anthropic Claude Code 配置档案已失效，它会建议运行 `claude setup-token`（或粘贴一个 setup-token）。仅在交互式（TTY）运行时才会出现刷新提示；`--non-interactive` 会跳过刷新尝试。

Doctor 还会报告暂时不可用的认证配置档案，原因包括：

* 短暂冷却时间（速率限制 / 超时 / 认证失败）
* 更长期的禁用（计费 / 授信额度失败）

<div id="6-hooks-model-validation">
  ### 6) Hooks 模型验证
</div>

如果配置了 `hooks.gmail.model`，doctor 会根据目录和允许列表验证该模型引用，并在无法解析或不被允许时发出警告。

<div id="7-sandbox-image-repair">
  ### 7) 沙箱镜像修复
</div>

当启用沙箱时，doctor 会检查 Docker 镜像，如果当前镜像缺失，则会提示你构建或切换到旧镜像名称。

<div id="8-gateway-service-migrations-and-cleanup-hints">
  ### 8) Gateway 服务迁移与清理提示
</div>

Doctor 会检测遗留的 Gateway 服务（launchd/systemd/schtasks），并
提供移除它们并使用当前 gateway 端口安装 OpenClaw 服务的选项。
它还可以扫描额外的类似 Gateway 的服务并输出清理建议。
以 profile 命名的 OpenClaw Gateway 服务被视为一等公民，不会
被标记为「多余」。

<div id="9-security-warnings">
  ### 9) 安全警告
</div>

当某个提供方在没有允许列表的情况下将私信策略设置为 open（表示允许任何用户发送消息），或者某个策略被配置成危险方式时，Doctor 会发出警告。

<div id="10-systemd-linger-linux">
  ### 10) systemd linger（Linux）
</div>

如果以 systemd 用户服务的方式运行，doctor 会确保启用了 linger，这样在注销后 Gateway 仍会继续运行。

<div id="11-skills-status">
  ### 11) 技能状态
</div>

Doctor 会为当前工作区快速输出一份可用 / 缺失 / 被阻止的技能摘要。

<div id="12-gateway-auth-checks-local-token">
  ### 12) Gateway 认证检查（本地令牌）
</div>

当本地 Gateway 上缺少 `gateway.auth` 时，Doctor 会发出警告并提示你生成令牌。使用 `openclaw doctor --generate-gateway-token` 可在自动化场景中强制创建令牌。

<div id="13-gateway-health-check-restart">
  ### 13) Gateway 健康检查与重启
</div>

Doctor 会运行一次健康检查，并在 Gateway 状态异常时提示你是否重启。

<div id="14-channel-status-warnings">
  ### 14) 通道状态警告
</div>

如果 Gateway 运行正常，doctor 会执行通道状态检查，并输出包含修复建议的警告信息。

<div id="15-supervisor-config-audit-repair">
  ### 15) Supervisor 配置审计与修复
</div>

Doctor 会检查已安装的 supervisor 配置（launchd/systemd/schtasks），
查找缺失或已过时的默认设置（例如 systemd 的 network-online 依赖和
重启延迟）。当发现不匹配时，它会建议进行更新，并可以将服务文件/任务
重写为当前默认值。

注意：

* 在重写 supervisor 配置前，`openclaw doctor` 会进行提示。
* `openclaw doctor --yes` 会接受默认的修复选项。
* `openclaw doctor --repair` 会在不进行交互提示的情况下应用推荐修复。
* `openclaw doctor --repair --force` 会覆盖自定义的 supervisor 配置。
* 你始终可以通过 `openclaw gateway install --force` 强制进行完整重写。

<div id="16-gateway-runtime-port-diagnostics">
  ### 16) Gateway 运行时与端口诊断
</div>

Doctor 会检查服务运行状态（PID、上次退出状态），并在服务已安装但实际未运行时发出警告。它还会检查 Gateway 监听端口（默认 `18789`）是否存在端口冲突，并报告可能的原因（Gateway 已在运行、SSH 隧道）。

<div id="17-gateway-runtime-best-practices">
  ### 17) Gateway 运行时最佳实践
</div>

当 Gateway 服务运行在 Bun 上，或使用由版本管理器管理的 Node 路径
（`nvm`、`fnm`、`volta`、`asdf` 等）时，Doctor 会发出警告。WhatsApp 和 Telegram 渠道需要 Node，
并且在升级后，由版本管理器提供的路径可能会失效，因为该服务不会加载你的 shell 初始化脚本。
如果存在系统级 Node 安装（通过 Homebrew/apt/choco 等），Doctor 会提示你迁移到该系统级安装。

<div id="18-config-write-wizard-metadata">
  ### 18) 配置写入与向导元数据
</div>

Doctor 会持久化所有配置更改，并写入向导元数据以记录本次 Doctor 运行。

<div id="19-workspace-tips-backup-memory-system">
  ### 19) 工作区提示（备份 + 记忆系统）
</div>

当缺少工作区记忆系统时，Doctor 会建议创建一个；如果工作区尚未纳入 git 版本控制，
则会输出备份提示信息。

关于工作区结构和 git 备份的完整指南（推荐使用私有 GitHub 或 GitLab），请参见
[/concepts/agent-workspace](/zh/concepts/agent-workspace)。