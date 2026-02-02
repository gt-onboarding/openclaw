---
title: 更新
summary: "安全地更新 OpenClaw（全局安装或从源码安装），以及回滚策略"
read_when:
  - 更新 OpenClaw
  - 更新后出现问题
---

<div id="updating">
  # 更新
</div>

OpenClaw 正在快速迭代（尚未到 “1.0” 阶段）。像对待生产基础设施发布一样处理更新：更新 → 运行检查 → 重启（或使用 `openclaw update`，它会自动重启） → 验证。

<div id="recommended-re-run-the-website-installer-upgrade-in-place">
  ## 推荐方式：重新运行网站安装程序（就地升级）
</div>

**首选**的更新方式是从网站重新运行安装程序。它会检测现有安装、执行就地升级，并在需要时运行 `openclaw doctor`。

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

Notes:

* 如果你不希望再次运行引导向导，请添加 `--no-onboard`。
* 对于 **源码安装（source installs）**，使用：
  ```bash
  curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method git --no-onboard
  ```
  仅当本地仓库干净时，安装器才会执行 `git pull --rebase`。
* 对于 **全局安装（global installs）**，脚本底层会使用 `npm install -g openclaw@latest`。
* 兼容性说明：`openclaw` 仍然可用，作为一个兼容层（shim）。

<div id="before-you-update">
  ## 在更新之前
</div>

* 先搞清楚你是如何安装的：**全局安装**（npm/pnpm）还是**从源码安装**（git clone）。
* 搞清楚你的 Gateway 是如何运行的：**前台终端**还是**受管服务**（launchd/systemd）。
* 为你的定制配置做一次快照备份：
  * 配置：`~/.openclaw/openclaw.json`
  * 凭据：`~/.openclaw/credentials/`
  * 工作区：`~/.openclaw/workspace`

<div id="update-global-install">
  ## 更新（全局安装）
</div>

全局安装方式（任选其一）：

```bash
npm i -g openclaw@latest
```

```bash
pnpm add -g openclaw@latest
```

我们**不**推荐将 Bun 用作 Gateway 的运行时（在 WhatsApp/Telegram 上存在 bug）。

若要切换更新通道（git + npm 安装方式）：

```bash
openclaw update --channel beta
openclaw update --channel dev
openclaw update --channel stable
```

若要为一次性安装指定标签或版本，请使用 `--tag <dist-tag|version>`。

有关通道语义和发布说明，参见 [Development channels](/zh/install/development-channels)。

注意：使用 npm 安装时，Gateway 在启动时会在日志中输出更新提示（会检查当前通道标签）。可通过 `update.checkOnStart: false` 禁用。

然后：

```bash
openclaw doctor
openclaw gateway restart
openclaw health
```

Notes:

* 当你的 Gateway 以服务形式运行时，应优先使用 `openclaw gateway restart`，而不是通过 PID 杀掉进程。
* 如果你需要锁定在某个特定版本，请参见下文的“回滚 / 版本锁定”部分。

<div id="update-openclaw-update">
  ## 更新（`openclaw update`）
</div>

对于**源码安装**（git checkout），建议优先使用：

```bash
openclaw update
```

它会运行一个相对安全的更新流程：

* 要求工作树是干净的。
* 切换到所选的通道（tag 或 branch）。
* 从配置的上游（dev 通道）fetch 并 rebase。
* 安装依赖、构建、构建 Control UI，并运行 `openclaw doctor`。
* 默认重启 Gateway（使用 `--no-restart` 跳过重启）。

如果你是通过 **npm/pnpm** 安装的（没有 git 元数据），`openclaw update` 会尝试通过你的包管理器进行更新。如果它无法检测到安装位置，请改用 “Update (global install)”。

<div id="update-control-ui-rpc">
  ## 更新（Control UI / RPC）
</div>

Control UI 提供 **Update &amp; Restart**（RPC：`update.run`）。它会：

1. 运行与 `openclaw update` 相同的源码更新流程（仅执行 git checkout）。
2. 写入一个带有结构化报告（包含 stdout/stderr 尾部内容）的重启标记。
3. 重启 Gateway，并向最近一个活跃会话发送该报告。

如果 rebase 失败，Gateway 会终止并在未应用更新的情况下重启。

<div id="update-from-source">
  ## 从源码更新
</div>

在仓库检出目录中执行：

首选方式：

```bash
openclaw update
```

手动方式（大致等同）：

```bash
git pull
pnpm install
pnpm build
pnpm ui:build # 首次运行时自动安装 UI 依赖项
openclaw doctor
openclaw health
```

Notes:

* 当你运行打包后的 `openclaw` 可执行文件（[`openclaw.mjs`](https://github.com/openclaw/openclaw/blob/main/openclaw.mjs)）或使用 Node 运行 `dist/` 时，`pnpm build` 才是相关的。
* 如果你从仓库检出的代码目录运行、而没有进行全局安装，对 CLI 命令请使用 `pnpm openclaw ...`。
* 如果你直接从 TypeScript 运行（`pnpm openclaw ...`），通常不需要重新构建，但**配置迁移仍然会生效** → 仍需运行 `openclaw doctor`。
* 在全局安装和基于 Git 的安装之间切换很简单：先安装另一种方式，然后运行 `openclaw doctor`，这样 Gateway 服务入口点就会被重写为当前的安装位置。

<div id="always-run-openclaw-doctor">
  ## 务必运行：`openclaw doctor`
</div>

Doctor 是用于“安全更新”的命令。它被刻意设计得很「无聊」：修复 + 迁移 + 警告。

注意：如果你是通过**源码安装**（git checkout）运行的，`openclaw doctor` 会先提示是否运行 `openclaw update`。

它通常会做这些事：

* 迁移已弃用的配置键 / 旧版配置文件路径。
* 审计私信（DM）策略，并对具有风险的 open 设置（允许从任何用户不受限制地接收消息）发出警告。
* 检查 Gateway 的健康状况，并在需要时提示重启。
* 检测并将旧版 Gateway 服务（launchd/systemd；旧版 schtasks）迁移为当前的 OpenClaw 服务。
* 在 Linux 上，确保启用 systemd 用户 lingering（使 Gateway 在注销后仍能保持运行）。

详细信息：[Doctor](/zh/gateway/doctor)

<div id="start-stop-restart-the-gateway">
  ## 启动 / 停止 / 重启 Gateway
</div>

CLI（适用于任意操作系统）：

```bash
openclaw gateway status
openclaw gateway stop
openclaw gateway restart
openclaw gateway --port 18789
openclaw logs --follow
```

如果你是通过服务管理器托管运行的（supervised）：

* macOS launchd（应用内置的 LaunchAgent）：`launchctl kickstart -k gui/$UID/bot.molt.gateway`（使用 `bot.molt.<profile>`；旧版的 `com.openclaw.*` 仍然可用）
* Linux systemd 用户服务：`systemctl --user restart openclaw-gateway[-<profile>].service`
* Windows（WSL2）：`systemctl --user restart openclaw-gateway[-<profile>].service`
  * 只有在已安装为服务的情况下，`launchctl`/`systemctl` 才能使用；否则请运行 `openclaw gateway install`。

运行手册及具体的服务标签（service label）：[Gateway runbook](/zh/gateway)

<div id="rollback-pinning-when-something-breaks">
  ## 回滚 / 版本锁定（出现问题时）
</div>

<div id="pin-global-install">
  ### 锁定版本（全局安装）
</div>

安装一个已知稳定可用的版本（将 `<version>` 替换为最后一个可正常工作的版本）：

```bash
npm i -g openclaw@<version>
```

```bash
pnpm add -g openclaw@<version>
```

提示：若要查看当前已发布的版本，请运行 `npm view openclaw version`。

然后重启并重新运行 doctor 命令：

```bash
openclaw doctor
openclaw gateway restart
```

<div id="pin-source-by-date">
  ### 按日期固定（源代码）
</div>

从某个日期选取一个提交（例如：“main 在 2026-01-01 时的状态”）：

```bash
git fetch origin
git checkout "$(git rev-list -n 1 --before=\"2026-01-01\" origin/main)"
```

然后重新安装依赖项并重启：

```bash
pnpm install
pnpm build
openclaw gateway restart
```

如果之后你想切换回最新版本：

```bash
git checkout main
git pull
```

<div id="if-youre-stuck">
  ## 如果遇到问题
</div>

* 再次运行 `openclaw doctor`，并仔细阅读输出内容（通常会直接告诉你应如何解决问题）。
* 查看：[故障排查](/zh/gateway/troubleshooting)
* 在 Discord 中提问：https://channels.discord.gg/clawd