---
title: 认证监控
summary: "监控模型提供方的 OAuth 过期情况"
read_when:
  - 配置认证过期监控或告警
  - 自动化执行 Claude Code / Codex 的 OAuth 刷新检查
---

<div id="auth-monitoring">
  # 认证监控
</div>

OpenClaw 通过 `openclaw models status` 命令公开 OAuth 过期相关的健康检查信息。请使用该命令实现自动化与告警；针对手机相关工作流，脚本只是可选的补充。

<div id="preferred-cli-check-portable">
  ## 推荐：CLI 检测（可移植）
</div>

```bash
openclaw models status --check
```

退出码：

* `0`: 正常
* `1`: 凭证已过期或缺失
* `2`: 即将过期（24 小时内）

在 cron/systemd 中可直接使用，无需额外脚本。


<div id="optional-scripts-ops-phone-workflows">
  ## 可选脚本（运维 / 手机流程）
</div>

这些脚本位于 `scripts/` 目录下，并且是**可选**的。它们假设你可以通过 SSH 访问
Gateway 主机，并针对 systemd + Termux 做了优化。

- `scripts/claude-auth-status.sh` 现在使用 `openclaw models status --json` 作为
  可信数据源（如果 CLI 不可用则回退为直接读取文件），所以请确保在定时任务中
  `openclaw` 已包含在 `PATH` 中。
- `scripts/auth-monitor.sh`：cron/systemd 定时器目标脚本；发送告警（通过 ntfy 或手机）。
- `scripts/systemd/openclaw-auth-monitor.{service,timer}`：systemd 用户定时器单元。
- `scripts/claude-auth-status.sh`：Claude Code + OpenClaw 认证检查脚本（full/json/simple）。
- `scripts/mobile-reauth.sh`：通过 SSH 的引导式重新认证流程。
- `scripts/termux-quick-auth.sh`：一键小组件：显示状态并打开认证 URL。
- `scripts/termux-auth-widget.sh`：完整的引导式小组件认证流程。
- `scripts/termux-sync-widget.sh`：将 Claude Code 凭据同步到 OpenClaw。

如果你不需要手机自动化或 systemd 定时器，可以跳过这些脚本。