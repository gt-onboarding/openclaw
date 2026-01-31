---
title: 卸载
summary: "完全卸载 OpenClaw（CLI、服务、状态、工作区）"
read_when:
  - 你想从机器上移除 OpenClaw
  - 卸载后 Gateway 服务仍然在运行
---

<div id="uninstall">
  # 卸载
</div>

有两种路径：

- **简单路径**：当仍然安装着 `openclaw` 时使用。
- **手动移除服务**：当 CLI 已被移除但服务仍在运行时使用。

<div id="easy-path-cli-still-installed">
  ## 简单方式（仍安装有 CLI）
</div>

推荐使用内置卸载程序：

```bash
openclaw uninstall
```

非交互模式（自动化 / npx）：

```bash
openclaw uninstall --all --yes --non-interactive
npx -y openclaw uninstall --all --yes --non-interactive
```

手动步骤（效果相同）：

1. 停止 Gateway 服务：

```bash
openclaw gateway stop
```

2. 卸载 Gateway 服务（launchd/systemd/schtasks）：

```bash
openclaw gateway uninstall
```

3. 删除状态和配置文件：

```bash
rm -rf "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}"
```

如果你将 `OPENCLAW_CONFIG_PATH` 设置为状态目录之外的自定义路径，也请删除该文件。

4. 删除你的工作区（可选，会删除智能体文件）：

```bash
rm -rf ~/.openclaw/workspace
```

5. 卸载 CLI（根据你之前的安装方式选择对应的命令）：

```bash
npm rm -g openclaw
pnpm remove -g openclaw
bun remove -g openclaw
```

6. 如果你安装过 macOS 应用：

```bash
rm -rf /Applications/OpenClaw.app
```

注意事项：

* 如果你使用了 profile 配置（`--profile` / `OPENCLAW_PROFILE`），请针对每个状态目录重复执行步骤 3（默认是 `~/.openclaw-<profile>`）。
* 在远程模式下，状态目录位于 **Gateway 主机** 上，因此也需要在该主机上执行步骤 1–4。


<div id="manual-service-removal-cli-not-installed">
  ## 手动移除服务（未安装 CLI）
</div>

如果 Gateway 服务仍在运行，但系统中已没有 `openclaw` 可执行文件，请使用本方法。

<div id="macos-launchd">
  ### macOS（launchd）
</div>

默认的标签（label）为 `bot.molt.gateway`（或 `bot.molt.&lt;profile&gt;`；旧版的 `com.openclaw.*` 可能仍然存在）：

```bash
launchctl bootout gui/$UID/bot.molt.gateway
rm -f ~/Library/LaunchAgents/bot.molt.gateway.plist
```

如果你使用了 profile，请将标签和 plist 名称改为 `bot.molt.&lt;profile&gt;`。如果存在任何遗留的 `com.openclaw.*` plist 文件，请将其删除。


<div id="linux-systemd-user-unit">
  ### Linux（systemd 用户级单元）
</div>

默认的单元名称为 `openclaw-gateway.service`（或 `openclaw-gateway-<profile>.service`）：

```bash
systemctl --user disable --now openclaw-gateway.service
rm -f ~/.config/systemd/user/openclaw-gateway.service
systemctl --user daemon-reload
```


<div id="windows-scheduled-task">
  ### Windows（计划任务）
</div>

默认的任务名称为 `OpenClaw Gateway`（或 `OpenClaw Gateway (<profile>)`）。
任务脚本位于你的 state 目录中。

```powershell
schtasks /Delete /F /TN "OpenClaw Gateway"
Remove-Item -Force "$env:USERPROFILE\.openclaw\gateway.cmd"
```

如果你使用了某个 profile，请删除匹配的任务名称以及 `~\.openclaw-<profile>\gateway.cmd`。


<div id="normal-install-vs-source-checkout">
  ## 常规安装与源码检出
</div>

<div id="normal-install-installsh-npm-pnpm-bun">
  ### 常规安装（install.sh / npm / pnpm / bun）
</div>

如果你使用 `https://openclaw.bot/install.sh` 或 `install.ps1`，CLI 是通过执行 `npm install -g openclaw@latest` 安装的。
使用 `npm rm -g openclaw` 将其卸载（如果是通过 `pnpm` 或 `bun` 安装，则分别使用 `pnpm remove -g` / `bun remove -g`）。

<div id="source-checkout-git-clone">
  ### 源码检出（git clone）
</div>

如果你是从仓库检出后运行（`git clone` + `openclaw ...` / `bun run openclaw ...`）：

1) 在删除仓库之前，先卸载 Gateway 服务（使用上面的简便方式或手动移除服务）。
2) 删除仓库目录。
3) 按上文所示删除状态数据和工作区。