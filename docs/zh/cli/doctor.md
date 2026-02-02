---
title: 诊断
summary: "用于 `openclaw doctor` 的 CLI 参考（健康检查 + 引导式修复）"
read_when:
  - 当你遇到连接/身份验证问题并希望获得引导式修复时
  - 当你完成更新后想做一次基本状态检查时
---

<div id="openclaw-doctor">
  # `openclaw doctor`
</div>

为 Gateway 和各个通道提供健康检查与快速修复。

相关内容：

* 故障排查：[Troubleshooting](/zh/gateway/troubleshooting)
* 安全审计：[Security](/zh/gateway/security)

<div id="examples">
  ## 示例
</div>

```bash
openclaw doctor
openclaw doctor --repair
openclaw doctor --deep
```

Notes:

* 只有当 stdin 是 TTY 且 **未** 设置 `--non-interactive` 时，才会运行交互式提示（例如钥匙串 / OAuth 修复）。无头模式运行（如 cron、Telegram、无终端）会跳过这些提示。
* `--fix`（`--repair` 的别名）会将备份写入 `~/.openclaw/openclaw.json.bak`，并删除未知的配置键，同时逐条列出每一项删除。

<div id="macos-launchctl-env-overrides">
  ## macOS：`launchctl` 环境变量覆盖
</div>

如果你之前运行过 `launchctl setenv OPENCLAW_GATEWAY_TOKEN ...`（或 `...PASSWORD`），该值会优先生效于你的配置文件，并可能导致持续的“未授权”（unauthorized）错误。

```bash
launchctl getenv OPENCLAW_GATEWAY_TOKEN
launchctl getenv OPENCLAW_GATEWAY_PASSWORD

launchctl unsetenv OPENCLAW_GATEWAY_TOKEN
launchctl unsetenv OPENCLAW_GATEWAY_PASSWORD
```
