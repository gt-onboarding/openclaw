---
title: 故障排查
summary: "常见 OpenClaw 故障的快速排查指南"
read_when:
  - 在排查运行时问题或故障时
---

<div id="troubleshooting">
  # 故障排查 🔧
</div>

当 OpenClaw 出现异常时，可以按照下面的方法来修复。

如果你只想要一个快速排查步骤，请先查看 FAQ 中的 [前 60 秒](/zh/help/faq#first-60-seconds-if-somethings-broken)。本页会更深入地介绍运行时故障和诊断方法。

按提供方分类的故障排查入口：[/channels/troubleshooting](/zh/channels/troubleshooting)

<div id="status-diagnostics">
  ## 状态与诊断
</div>

快速排查命令（按顺序）：

| 命令 | 它告诉你什么 | 何时使用 |
|---|---|---|
| `openclaw status` | 本地摘要：操作系统与更新状态、Gateway 可达性/模式、服务状态、智能体/会话、提供方配置状态 | 首次检查，用于快速总览 |
| `openclaw status --all` | 完整本地诊断（只读、可粘贴、相对安全可分享），包括日志尾部内容 | 需要分享调试报告时 |
| `openclaw status --deep` | 运行 Gateway 健康检查（包括提供方探测；需要 Gateway 可达） | 当“已配置”并不等于“正常工作”时 |
| `openclaw gateway probe` | Gateway 发现与可达性检查（本地与远程目标） | 当你怀疑自己探测的是错误的 Gateway 时 |
| `openclaw channels status --probe` | 向正在运行的 Gateway 请求通道状态（并可选执行探测） | Gateway 可达但通道行为异常时 |
| `openclaw gateway status` | 系统服务管理器状态（launchd/systemd/schtasks）、运行时 PID/退出码、最近一次 Gateway 错误 | 服务“看起来已加载”但实际上没有任何东西在运行时 |
| `openclaw logs --follow` | 实时日志（定位运行时问题的最有效信号） | 当你需要实际的失败原因时 |

**分享输出：**优先使用 `openclaw status --all`（会对令牌做脱敏处理）。如果粘贴 `openclaw status` 输出，建议先设置 `OPENCLAW_SHOW_SECRETS=0`（关闭令牌预览）。

另见：[健康检查](/zh/gateway/health) 和 [日志记录](/zh/logging)。

<div id="common-issues">
  ## 常见问题
</div>

<div id="no-api-key-found-for-provider-anthropic">
  ### 未找到提供方 &quot;anthropic&quot; 的 API 密钥
</div>

这意味着**该智能体的认证存储为空**，或者缺少 Anthropic 凭据。
认证是**按智能体隔离**的，因此新建智能体不会继承主智能体的密钥。

解决方式：

* 重新运行引导配置流程，并为该智能体选择 **Anthropic**。
* 或者在 **Gateway 主机** 上粘贴一个 setup-token：
  ```bash
  openclaw models auth setup-token --provider anthropic
  ```
* 或者将主智能体目录中的 `auth-profiles.json` 复制到新智能体目录中。

验证：

```bash
openclaw models status
```

<div id="oauth-token-refresh-failed-anthropic-claude-subscription">
  ### OAuth 令牌刷新失败（Anthropic Claude 订阅）
</div>

这意味着已存储的 Anthropic OAuth 令牌已过期且刷新失败。
如果你使用的是 Claude 订阅（没有 API 密钥），最可靠的解决办法是
切换为 **Claude Code setup-token**，然后将其粘贴到 **Gateway 主机** 上。

**推荐方式（setup-token）：**

```bash
# 在 gateway 主机上运行（粘贴 setup-token）
openclaw models auth setup-token --provider anthropic
openclaw models status
```

如果你是在别处生成令牌：

```bash
openclaw models auth paste-token --provider anthropic
openclaw models status
```

更多详细信息：[Anthropic](/zh/providers/anthropic) 和 [OAuth](/zh/concepts/oauth)。

<div id="control-ui-fails-on-http-device-identity-required-connect-failed">
  ### Control UI 在 HTTP 下失败（&quot;device identity required&quot; / &quot;connect failed&quot;）
</div>

如果你通过明文 HTTP 打开仪表盘（例如 `http://<lan-ip>:18789/` 或
`http://<tailscale-ip>:18789/`），浏览器会运行在**非安全上下文**中，并
阻止 WebCrypto，因此无法生成设备身份。

**解决方法：**

* 优先通过 [Tailscale Serve](/zh/gateway/tailscale) 使用 HTTPS。
* 或者在 Gateway 所在主机本地访问：`http://127.0.0.1:18789/`。
* 如果你必须使用 HTTP，将 `gateway.controlUi.allowInsecureAuth: true` 设置为 true，
  并使用 Gateway token（仅使用 token；无设备身份/配对）。参见
  [Control UI](/zh/web/control-ui#insecure-http)。

<div id="ci-secrets-scan-failed">
  ### CI 机密扫描失败
</div>

这表示 `detect-secrets` 发现了尚未纳入基线的新疑似机密信息。
请按照[机密扫描](/zh/gateway/security#secret-scanning-detect-secrets)中的说明操作。

<div id="service-installed-but-nothing-is-running">
  ### 服务已安装但没有任何进程在运行
</div>

如果 Gateway 服务已安装但其进程立即退出，服务状态可能会显示为“已加载”，但实际上并没有任何进程在运行。

**排查：**

```bash
openclaw gateway status
openclaw doctor
```

Doctor/service 会显示运行时状态（PID/最近一次退出）以及日志相关提示。

**日志：**

* 优先使用：`openclaw logs --follow`
* 文件日志（始终写入）：`/tmp/openclaw/openclaw-YYYY-MM-DD.log`（或你配置的 `logging.file`）
* macOS LaunchAgent（如已安装）：`$OPENCLAW_STATE_DIR/logs/gateway.log` 和 `gateway.err.log`
* Linux systemd（如已安装）：`journalctl --user -u openclaw-gateway[-<profile>].service -n 200 --no-pager`
* Windows：`schtasks /Query /TN "OpenClaw Gateway (<profile>)" /V /FO LIST`

**启用更详细的日志：**

* 提高文件日志详细程度（持久化 JSONL）：
  ```json
  { "logging": { "level": "debug" } }
  ```
* 提升控制台详细程度（仅 TTY 输出）：
  ```json
  { "logging": { "consoleLevel": "debug", "consoleStyle": "pretty" } }
  ```
* 小提示：`--verbose` 只影响**控制台**输出。文件日志仍由 `logging.level` 控制。

完整的日志格式、配置和访问说明见 [/logging](/zh/logging)。

<div id="gateway-start-blocked-set-gatewaymodelocal">
  ### &quot;Gateway start blocked: set gateway.mode=local&quot;
</div>

这意味着配置文件已存在，但 `gateway.mode` 未设置（或未设为 `local`），因此
Gateway 拒绝启动。

**解决方法（推荐）：**

* 运行向导并将 Gateway 运行模式设置为 **Local**：
  ```bash
  openclaw configure
  ```
* 或者直接设置：
  ```bash
  openclaw config set gateway.mode local
  ```

**如果你打算运行远程 Gateway：**

* 配置远程 URL 并保持 `gateway.mode=remote`：
  ```bash
  openclaw config set gateway.mode remote
  openclaw config set gateway.remote.url "wss://gateway.example.com"
  ```

**仅限临时/开发用途：** 添加 `--allow-unconfigured` 参数，在没有
`gateway.mode=local` 的情况下启动 gateway。

**还没有配置文件？** 运行 `openclaw setup` 创建初始配置，然后重新运行
gateway。

<div id="service-environment-path-runtime">
  ### 服务运行环境（PATH 与运行时）
</div>

Gateway 服务在一个**精简的 PATH** 下运行，以避免引入 shell/管理器的多余内容：

* macOS：`/opt/homebrew/bin`、`/usr/local/bin`、`/usr/bin`、`/bin`
* Linux：`/usr/local/bin`、`/usr/bin`、`/bin`

这会有意排除版本管理器（nvm/fnm/volta/asdf）和包管理器（pnpm/npm），因为该服务不会加载你的 shell 初始化脚本。像 `DISPLAY` 这样的运行时变量应放在 `~/.openclaw/.env` 中（由 Gateway 在早期加载）。
在 `host=gateway` 上运行的 Exec 会将你的登录 shell 的 `PATH` 合并进 exec 环境中，因此工具缺失通常意味着你的 shell 初始化脚本没有导出它们（或者需要设置 `tools.exec.pathPrepend`）。参见 [/tools/exec](/zh/tools/exec)。

WhatsApp 和 Telegram 通道需要 **Node**；Bun 不受支持。如果你的服务是用 Bun 或通过版本管理器提供的 Node 路径安装的，运行 `openclaw doctor` 以迁移到系统级 Node 安装。

<div id="skill-missing-api-key-in-sandbox">
  ### 技能在沙箱中缺少 API 密钥
</div>

**症状：** 技能在宿主机上正常工作，但在沙箱中因缺少 API 密钥而失败。

**原因：** 沙箱化执行是在 Docker 容器内运行的，**不会**继承宿主机的 `process.env` 环境变量。

**解决方法：**

* 设置 `agents.defaults.sandbox.docker.env`（或为单个智能体设置 `agents.list[].sandbox.docker.env`）
* 或者把密钥预置到你的自定义沙箱镜像中
* 然后运行 `openclaw sandbox recreate --agent <id>`（或 `--all`）

<div id="service-running-but-port-not-listening">
  ### 服务显示正在运行，但端口未在监听
</div>

如果服务报告为 **running**，但在 gateway 端口上没有任何监听，
很可能是 Gateway 拒绝了绑定。

**这里的 “running” 是什么意思**

* `Runtime: running` 表示你的进程管理器（launchd/systemd/schtasks）认为该进程仍然存活。
* `RPC probe` 表示 CLI 实际成功连接到了 gateway 的 WebSocket 并调用了 `status`。
* 始终以 `Probe target:` 和 `Config (service):` 组合为准，把它们看成“我们实际尝试的是什么”的那几行。

**检查要点：**

* 对于 `openclaw gateway` 和服务本身，`gateway.mode` 必须为 `local`。
* 如果你设置了 `gateway.mode=remote`，**CLI 默认** 会使用远程 URL。服务仍然可以在本地运行，但你的 CLI 可能在探测错误的位置。使用 `openclaw gateway status` 查看服务解析出的端口和探测目标（或传入 `--url`）。
* 当服务看起来在运行但端口仍关闭时，`openclaw gateway status` 和 `openclaw doctor` 会从日志中展示 **最近一次 gateway 错误**。
* 非回环绑定（`lan` / `tailnet` / `custom`，或在回环不可用时的 `auto`）需要鉴权：
  `gateway.auth.token`（或 `OPENCLAW_GATEWAY_TOKEN`）。
* `gateway.remote.token` 只用于远程 CLI 调用；它 **不会** 启用本地鉴权。
* `gateway.token` 会被忽略；请使用 `gateway.auth.token`。

**如果 `openclaw gateway status` 显示配置不匹配**

* `Config (cli): ...` 和 `Config (service): ...` 通常应该一致。
* 如果不一致，几乎可以肯定是你在编辑一个配置文件，而服务实际运行用的是另一个。
* 解决方法：在你希望服务使用的同一个 `--profile` / `OPENCLAW_STATE_DIR` 下重新运行 `openclaw gateway install --force`。

**如果 `openclaw gateway status` 报告服务配置问题**

* 进程管理器配置（launchd/systemd/schtasks）缺少当前默认配置。
* 解决方法：运行 `openclaw doctor` 来更新它（或运行 `openclaw gateway install --force` 进行完整重写）。

**如果 `Last gateway error:` 提到 “refusing to bind … without auth”**

* 你将 `gateway.bind` 设置为非回环模式（`lan` / `tailnet` / `custom`，或在回环不可用时的 `auto`），但没有配置鉴权。
* 解决方法：设置 `gateway.auth.mode` 和 `gateway.auth.token`（或导出 `OPENCLAW_GATEWAY_TOKEN`），然后重启服务。

**如果 `openclaw gateway status` 显示 `bind=tailnet` 但未发现 tailnet 网卡**

* gateway 尝试绑定到 Tailscale IP（100.64.0.0/10），但在主机上未检测到任何此类地址。
* 解决方法：在该机器上启动 Tailscale（或将 `gateway.bind` 改为 `loopback` / `lan`）。

**如果 `Probe note:` 表示探测使用了 loopback**

* 对于 `bind=lan` 这是预期行为：gateway 会监听在 `0.0.0.0`（所有网卡），而 loopback 仍然可以在本机连接。
* 对于远程客户端，请使用真实的 LAN IP（不是 `0.0.0.0`）加端口，并确保已配置鉴权。

<div id="address-already-in-use-port-18789">
  ### 端口已被占用（端口 18789）
</div>

这表示已经有进程在监听 Gateway 使用的该端口。

**检查：**

```bash
openclaw gateway status
```

它会显示监听端口以及可能的原因（Gateway 已在运行、SSH 隧道）。
如有需要，请停止该服务或选择其他端口。

<div id="extra-workspace-folders-detected">
  ### 检测到额外的工作区文件夹
</div>

如果你是从较早的安装版本升级而来，磁盘上可能还保留着 `~/openclaw`。
存在多个工作区目录会导致认证或状态发生令人困惑的漂移，因为
同一时间只有一个工作区是激活的。

**解决方法：** 只保留一个活动工作区，并将其余工作区归档或删除。参见
[Agent 代理工作区](/zh/concepts/agent-workspace#extra-workspace-folders)。

<div id="main-chat-running-in-a-sandbox-workspace">
  ### 主聊天会话在沙箱工作区中运行
</div>

现象：即使你预期看到的是宿主机工作区，`pwd` 或文件工具显示的却是 `~/.openclaw/sandboxes/...`。

**原因：** `agents.defaults.sandbox.mode: "non-main"` 是基于 `session.mainKey`（默认为 `"main"`）来判断的。
群组/频道会话使用它们自己的键，因此会被视为 non-main，从而获得沙箱工作区。

**修复选项：**

* 如果你希望某个智能体使用宿主机工作区：将 `agents.list[].sandbox.mode` 设为 `"off"`。
* 如果你希望在沙箱中访问宿主机工作区：为该智能体将 `workspaceAccess` 设为 `"rw"`。

<div id="agent-was-aborted">
  ### &quot;Agent 已中止&quot;
</div>

该智能体在响应过程中被中断。

**原因：**

* 用户发送了 `stop`、`abort`、`esc`、`wait` 或 `exit`
* 超过超时时间
* 进程崩溃

**解决方法：** 直接再发送一条消息，会话会继续。

<div id="agent-failed-before-reply-unknown-model-anthropicclaude-haiku-3-5">
  ### &quot;Agent failed before reply: Unknown model: anthropic/claude-haiku-3-5&quot;
</div>

OpenClaw 有意拒绝使用**较旧或不安全的模型**（尤其是那些更容易受到提示注入攻击的模型）。如果你看到这个错误，说明该模型已不再受支持。

**解决方法：**

* 为该提供方选择一个**最新**的模型，并更新你的配置或模型别名。
* 如果不确定当前可用的模型，可以运行 `openclaw models list` 或
  `openclaw models scan`，然后选择一个受支持的模型。
* 检查 Gateway 日志以获取详细的失败原因。

另见：[Models CLI](/zh/cli/models) 和 [Model providers](/zh/concepts/model-providers)。

<div id="messages-not-triggering">
  ### 消息未触发
</div>

**检查 1：** 消息发送方是否在允许列表中？

```bash
openclaw status
```

在输出结果中查找 `AllowFrom: ...`。

**检查 2：** 在群聊中，是否必须通过 @ 来提及？

```bash
# 消息必须匹配 mentionPatterns 或显式提及;默认值位于频道群组/公会中。
# 多智能体:`agents.list[].groupChat.mentionPatterns` 覆盖全局模式。
grep -n "agents\\|groupChat\\|mentionPatterns\\|channels\\.whatsapp\\.groups\\|channels\\.telegram\\.groups\\|channels\\.imessage\\.groups\\|channels\\.discord\\.guilds" \
  "${OPENCLAW_CONFIG_PATH:-$HOME/.openclaw/openclaw.json}"
```

**检查 3：** 查看日志

```bash
openclaw logs --follow
# 或者如果需要快速过滤:
tail -f "$(ls -t /tmp/openclaw/openclaw-*.log | head -1)" | grep "blocked\\|skip\\|unauthorized"
```

<div id="pairing-code-not-arriving">
  ### 未收到配对码
</div>

如果 `dmPolicy` 为 `pairing`，未知发送方应当会收到一条带有配对码的消息，并且在批准之前，他们的原始消息会被忽略。

**检查 1：** 是否已经存在待处理的配对请求？

```bash
openclaw pairing list <channel>
```

默认情况下，待处理的 DM 配对请求**每个频道最多 3 个**。如果列表已满，在已有请求被批准或过期之前，新请求不会生成配对码。

**检查 2：** 请求已经创建，但没有发送任何回复？

```bash
openclaw logs --follow | grep "pairing request"
```

**检查 3：** 确认该频道的 `dmPolicy` 未设置为 `open`/`allowlist`。

<div id="image-mention-not-working">
  ### 图片 + 提及不起作用
</div>

已知问题：当你发送只包含提及（没有其他文本）的图片消息时，WhatsApp 有时不会附带提及元数据。

**解决方法：** 在提及中顺带加上一点文本：

* ❌ `@openclaw` + 图片
* ✅ `@openclaw check this` + 图片

<div id="session-not-resuming">
  ### 会话无法恢复
</div>

**检查 1：** 会话文件是否存在？

```bash
ls -la ~/.openclaw/agents/<agentId>/sessions/
```

**检查 2：** 重置时间窗口是否设置得过短？

```json
{
  "session": {
    "reset": {
      "mode": "daily",
      "atHour": 4,
      "idleMinutes": 10080  // 7 天
    }
  }
}
```

**检查 3：** 是否有人发送了 `/new`、`/reset` 或触发了重置操作？

<div id="agent-timing-out">
  ### Agent 代理超时
</div>

默认超时时间为 30 分钟。对于长时间运行的任务：

```json
{
  "reply": {
    "timeoutSeconds": 3600  // 1 小时
  }
}
```

或者使用 `process` 工具将耗时较长的命令放到后台执行。

<div id="whatsapp-disconnected">
  ### WhatsApp 连接已断开
</div>

```bash
# Check local status (creds, sessions, queued events)
openclaw status
# 探测运行中的 Gateway + 频道(WA 连接 + Telegram + Discord API)
openclaw status --deep

# View recent connection events
openclaw logs --limit 200 | grep "connection\\|disconnect\\|logout"
```

**解决方法：** Gateway 启动后通常会自动重新连接。如果仍然不行，重启 Gateway 进程（按你当前的进程管理方式），或者手动启动并开启详细输出：

```bash
openclaw gateway --verbose
```

如果你当前已退出登录或已取消关联：

```bash
openclaw channels logout
trash "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}/credentials" # 如果 logout 无法完全清除所有内容
openclaw channels login --verbose       # re-scan QR
```

<div id="media-send-failing">
  ### 媒体发送失败
</div>

**检查 1：** 文件路径是否正确？

```bash
ls -la /path/to/your/image.jpg
```

**检查 2：** 是否太大？

* 图片：最大 6MB
* 音频/视频：最大 16MB
* 文档：最大 100MB

**检查 3：** 检查媒体日志

```bash
grep "media\\|fetch\\|download" "$(ls -t /tmp/openclaw/openclaw-*.log | head -1)" | tail -20
```

<div id="high-memory-usage">
  ### 内存占用高
</div>

OpenClaw 会将会话历史保存在内存中。

**解决方法：** 定期重启，或设置会话限制：

```json
{
  "session": {
    "historyLimit": 100  // 保留的最大消息数量
  }
}
```

<div id="common-troubleshooting">
  ## 常见问题排查
</div>

<div id="gateway-wont-start-configuration-invalid">
  ### “Gateway 无法启动 — 配置无效”
</div>

当配置中包含未知键名、格式错误的值或无效类型时，OpenClaw 会拒绝启动。
这是出于安全考虑的刻意设计。

使用 Doctor 诊断并修复：

```bash
openclaw doctor
openclaw doctor --fix
```

注意：

* `openclaw doctor` 会报告每一项无效配置。
* `openclaw doctor --fix` 会执行迁移和修复，并重写配置文件。
* 即使配置无效，`openclaw logs`、`openclaw health`、`openclaw status`、`openclaw gateway status` 和 `openclaw gateway probe` 等诊断命令仍然可以运行。

<div id="all-models-failed-what-should-i-check-first">
  ### “所有模型都失败了”——我首先应该检查什么？
</div>

* 正在尝试的提供方是否已配置有效的**凭证**（认证配置档案 + 环境变量）。
* **模型路由**：确认 `agents.defaults.model.primary` 及其回退项都是你有权限访问的模型。
* **Gateway 日志**：查看 `/tmp/openclaw/…` 中的日志，以获取精确的提供方错误信息。
* **模型状态**：在聊天中使用 `/model status`，或在 CLI 中运行 `openclaw models status`。

<div id="im-running-on-my-personal-whatsapp-number-why-is-self-chat-weird">
  ### 我用的是自己的 WhatsApp 号码 —— 为什么跟自己聊天怪怪的？
</div>

启用自聊模式，并把你自己的号码加入允许列表：

```json5
{
  channels: {
    whatsapp: {
      selfChatMode: true,
      dmPolicy: "allowlist",
      allowFrom: ["+15555550123"]
    }
  }
}
```

请参阅 [WhatsApp 配置](/zh/channels/whatsapp)。

<div id="whatsapp-logged-me-out-how-do-i-reauth">
  ### WhatsApp 把我退出登录了。如何重新登录？
</div>

再次运行登录命令并扫描二维码：

```bash
openclaw channels login
```

<div id="build-errors-on-main-whats-the-standard-fix-path">
  ### `main` 分支构建报错——标准修复路径是什么？
</div>

1. `git pull origin main && pnpm install`
2. `openclaw doctor`
3. 查看 GitHub issue 或 Discord
4. 临时解决方案：检出一个较早的提交

<div id="npm-install-fails-allow-build-scripts-missing-tar-or-yargs-what-now">
  ### npm install 失败（allow-build-scripts / 缺少 tar 或 yargs）。现在怎么办？
</div>

如果你是从源码运行，请使用仓库的包管理器：**pnpm**（推荐）。
该仓库声明了 `packageManager: "pnpm@…"`。

典型的恢复步骤：

```bash
git status   # 确保你在代码仓库根目录
pnpm install
pnpm build
openclaw doctor
openclaw gateway restart
```

原因：该仓库已配置使用 pnpm 作为包管理器。

<div id="how-do-i-switch-between-git-installs-and-npm-installs">
  ### 如何在 git 安装和 npm 安装之间切换？
</div>

使用**网站安装器**，通过标志（flag）选择安装方式。它会就地升级，并重写 Gateway 服务，使其指向新的安装目录。

切换**到 git 安装**：

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method git --no-onboard
```

切换为 **npm 全局**：

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

Notes:

* 只有在仓库干净（无未提交更改）时，git 流程才会执行 rebase。请先提交或暂存（stash）变更。
* 切换之后，运行：
  ```bash
  openclaw doctor
  openclaw gateway restart
  ```

<div id="telegram-block-streaming-isnt-splitting-text-between-tool-calls-why">
  ### 为什么 Telegram 分块流式传输不会在工具调用之间拆分文本？
</div>

分块流式传输只会发送**已完成的文本块**。常见导致你只看到一条消息的原因包括：

* `agents.defaults.blockStreamingDefault` 仍然是 `"off"`。
* `channels.telegram.blockStreaming` 被设置为 `false`。
* `channels.telegram.streamMode` 为 `partial` 或 `block` **且草稿流式传输已启用**
  （私聊 + 主题）。在这种情况下，草稿流式传输会禁用分块流式传输。
* 你的 `minChars` / 合并（coalesce）设置过高，导致分块被合并。
* 模型一次输出一个大文本块（中途没有刷新点）。

排查清单：

1. 将分块流式传输相关设置放在 `agents.defaults` 下，而不是配置根级（root）。
2. 如果你希望真正的多条消息分块回复，设置 `channels.telegram.streamMode: "off"`。
3. 在调试时使用更小的分块/合并阈值。

参见 [Streaming](/zh/concepts/streaming)。

<div id="discord-doesnt-reply-in-my-server-even-with-requiremention-false-why">
  ### 即使设置了 `requireMention: false`，Discord 在我的服务器里也没有回复。为什么？
</div>

`requireMention` 只控制频道通过允许列表 **之后** 是否需要提及 (@) 的限制。
默认情况下，`channels.discord.groupPolicy` 为 **allowlist**，因此必须显式启用各个 guild。
如果你设置了 `channels.discord.guilds.<guildId>.channels`，则只有列出的频道被允许；省略该字段则允许该 guild 中的所有频道。

排查清单：

1. 将 `channels.discord.groupPolicy: "open"` **或** 添加一个 guild 允许列表条目（以及可选的频道允许列表）。
2. 在 `channels.discord.guilds.<guildId>.channels` 中使用 **数字频道 ID**。
3. 将 `requireMention: false` 放在 `channels.discord.guilds` **下面**（全局或按频道）。
   顶层的 `channels.discord.requireMention` 不是受支持的配置键。
4. 确保机器人具有 **Message Content Intent** 和相应的频道权限。
5. 运行 `openclaw channels status --probe` 获取排查线索。

文档：[Discord](/zh/channels/discord)，[频道故障排除](/zh/channels/troubleshooting)。

<div id="cloud-code-assist-api-error-invalid-tool-schema-400-what-now">
  ### Cloud Code Assist API 报错：invalid tool schema (400)。接下来怎么办？
</div>

这几乎总是**工具 schema 兼容性**问题。Cloud Code Assist
端点只接受 JSON Schema 的一个严格子集。OpenClaw 已在当前 `main`
中对工具 schema 做清洗/规范化处理，但该修复尚未进入最新发行版（截至
2026 年 1 月 13 日）。

修复检查清单：

1. **更新 OpenClaw**：
   * 如果你可以从源码运行，拉取 `main` 并重启 Gateway。
   * 否则，请等待包含 schema 清洗器的下一个发行版。
2. 避免使用不被支持的关键字，比如 `anyOf/oneOf/allOf`、`patternProperties`、
   `additionalProperties`、`minLength`、`maxLength`、`format` 等。
3. 如果你定义自定义工具，请保持顶层 schema 为 `type: "object"`，并使用
   `properties` 和简单的枚举。

参见 [Tools](/zh/tools) 和 [TypeBox schemas](/zh/concepts/typebox)。

<div id="macos-specific-issues">
  ## macOS 特有问题
</div>

<div id="app-crashes-when-granting-permissions-speechmic">
  ### 授予语音/麦克风权限时应用崩溃
</div>

如果在你点击隐私弹窗中的 “Allow（允许）” 后，应用直接退出或显示 “Abort trap 6”：

**解决方案 1：重置 TCC 缓存**

```bash
tccutil reset All bot.molt.mac.debug
```

**修复方案 2：强制使用新的 Bundle ID**
如果重置无效，修改 [`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) 中的 `BUNDLE_ID`（例如添加 `.test` 后缀），然后重新构建。这样可以强制 macOS 将其视为一个新应用。

<div id="gateway-stuck-on-starting">
  ### Gateway 卡在 &quot;Starting...&quot;
</div>

该应用会通过端口 `18789` 连接到本地 Gateway。如果一直卡在这个状态：

**解决方法 1：先停止 supervisor（推荐）**
如果 Gateway 由 launchd 托管，直接杀掉该 PID 只会让它自动重启。先停止 supervisor：

```bash
openclaw gateway status
openclaw gateway stop
# 或者: launchctl bootout gui/$UID/bot.molt.gateway (替换为 bot.molt.<profile>;旧版 com.openclaw.* 仍然可用)
```

**解决方案 2：端口被占用（查找监听进程）**

```bash
lsof -nP -iTCP:18789 -sTCP:LISTEN
```

如果是未受监控的进程，先尝试平滑停止，再逐步升级处理方式：

```bash
kill -TERM <PID>
sleep 1
kill -9 <PID> # 最后的手段
```

**修复措施 3：检查 CLI 安装**
确保存有全局 `openclaw` CLI，并且其版本与 App 版本一致：

```bash
openclaw --version
npm install -g openclaw@<version>
```

<div id="debug-mode">
  ## 调试模式
</div>

获取更详细的日志：

```bash
# 在配置中开启跟踪日志:
#   ${OPENCLAW_CONFIG_PATH:-$HOME/.openclaw/openclaw.json} -> { logging: { level: "trace" } }
#
# 然后运行详细命令将调试输出镜像到 stdout:
openclaw gateway --verbose
openclaw channels login --verbose
```

<div id="log-locations">
  ## 日志位置
</div>

| 日志 | 位置 |
|-----|----------|
| Gateway 文件日志（结构化） | `/tmp/openclaw/openclaw-YYYY-MM-DD.log`（或 `logging.file`） |
| Gateway 服务日志（supervisor） | macOS：`$OPENCLAW_STATE_DIR/logs/gateway.log` + `gateway.err.log`（默认：`~/.openclaw/logs/...`；使用 profile 时为 `~/.openclaw-<profile>/logs/...`）<br />Linux：`journalctl --user -u openclaw-gateway[-<profile>].service -n 200 --no-pager`<br />Windows：`schtasks /Query /TN "OpenClaw Gateway (<profile>)" /V /FO LIST` |
| 会话文件 | `$OPENCLAW_STATE_DIR/agents/<agentId>/sessions/` |
| 媒体缓存 | `$OPENCLAW_STATE_DIR/media/` |
| 凭据 | `$OPENCLAW_STATE_DIR/credentials/` |

<div id="health-check">
  ## 运行状况检查
</div>

```bash
# Supervisor + probe target + config paths
openclaw gateway status
# 包含系统级扫描(遗留/额外服务、端口监听器)
openclaw gateway status --deep

# Is the gateway reachable?
openclaw health --json
# If it fails, rerun with connection details:
openclaw health --verbose

# Is something listening on the default port?
lsof -nP -iTCP:18789 -sTCP:LISTEN

# Recent activity (RPC log tail)
openclaw logs --follow
# Fallback if RPC is down
tail -20 /tmp/openclaw/openclaw-*.log
```

<div id="reset-everything">
  ## 全部重置
</div>

终极方案：

```bash
openclaw gateway stop
# If you installed a service and want a clean install:
# openclaw gateway uninstall

trash "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}"
openclaw channels login         # re-pair WhatsApp
openclaw gateway restart           # 或: openclaw gateway
```

⚠️ 这会丢失所有会话，并且需要重新与 WhatsApp 配对。

<div id="getting-help">
  ## 获取帮助
</div>

1. 先检查日志：`/tmp/openclaw/`（默认：`openclaw-YYYY-MM-DD.log`，或你在 `logging.file` 中配置的路径）
2. 在 GitHub 上搜索已存在的 issue
3. 新建一个 issue，并提供：
   * OpenClaw 版本
   * 相关日志片段
   * 复现步骤
   * 你的配置（请删除/遮蔽敏感信息！）

***

*“你试过把它关掉再开机吗？”* —— 每一个 IT 从业者

🦞🔧

<div id="browser-not-starting-linux">
  ### 浏览器无法启动（Linux）
</div>

如果你看到 `"Failed to start Chrome CDP on port 18800"`：

**最可能的原因：** 在 Ubuntu 上使用 Snap 安装的 Chromium。

**快速解决方法：** 改为安装 Google Chrome：

```bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb
```

然后在配置文件中进行如下设置：

```json
{
  "browser": {
    "executablePath": "/usr/bin/google-chrome-stable"
  }
}
```

**完整指南：** 请参见 [browser-linux-troubleshooting](/zh/tools/browser-linux-troubleshooting)
