---
title: 首次使用引导
summary: "OpenClaw（macOS 应用）的首次运行引导流程"
read_when:
  - 设计 macOS 首次使用引导助手时
  - 实现认证或身份设置功能时
---

<div id="onboarding-macos-app">
  # 初始引导（macOS 应用）
</div>

本文档描述 **当前** 的首次运行引导流程。目标是提供顺畅的「第 0 天（Day 0）」体验：选择 Gateway 的运行位置、配置认证、运行向导，并让智能体自行完成初始化引导。

<div id="page-order-current">
  ## 页面顺序（当前）
</div>

1. 欢迎 + 安全提示
2. **Gateway 选择**（本地 / 远程 / 稍后配置）
3. **身份验证（Anthropic OAuth）** — 仅本地
4. **设置向导**（由 Gateway 驱动）
5. **权限**（TCC 提示）
6. **CLI**（可选）
7. **上手引导对话**（独立会话）
8. 就绪

<div id="1-local-vs-remote">
  ## 1) 本地 vs 远程
</div>

**Gateway** 运行在哪里？

* **本地（此 Mac）：** 引导向导可以在本机运行 OAuth 流程并将凭据写入本地。
* **远程（通过 SSH/Tailnet）：** 引导向导**不会**在本机运行 OAuth；凭据必须已经存在于 Gateway 所在主机上。
* **稍后配置：** 跳过设置，让应用保持未配置状态。

Gateway 身份验证提示：

* 向导现在即使对 loopback 回环接口也会生成 **token**，因此本地 WS 客户端必须进行身份验证。
* 如果你禁用身份验证，任何本地进程都可以连接；仅在完全可信的机器上这样做。
* 对多台机器访问或非 loopback 绑定使用 **token**。

<div id="2-local-only-auth-anthropic-oauth">
  ## 2）仅本地身份验证（Anthropic OAuth）
</div>

macOS 应用支持 Anthropic OAuth（Claude Pro/Max）。流程如下：

* 打开浏览器以完成 OAuth（PKCE）授权
* 提示用户粘贴 `code#state` 值
* 将凭据写入 `~/.openclaw/credentials/oauth.json`

其他提供方（如 OpenAI、自定义 API）目前通过环境变量或配置文件进行配置。

<div id="3-setup-wizard-gatewaydriven">
  ## 3) 设置向导（由 Gateway 驱动）
</div>

该应用可以运行与 CLI 相同的设置向导。这样可以让入门流程与 Gateway 端行为保持一致，并避免在 SwiftUI 中重复实现逻辑。

<div id="4-permissions">
  ## 4) 权限
</div>

首次使用时会请求以下 TCC 权限：

* 通知
* 辅助功能
* 屏幕录制
* 麦克风/语音识别
* 自动化（AppleScript）

<div id="5-cli-optional">
  ## 5) CLI（可选）
</div>

该应用可以通过 npm/pnpm 安装全局 `openclaw` CLI，这样终端中的工作流和 launchd 任务即可开箱即用。

<div id="6-onboarding-chat-dedicated-session">
  ## 6) 入门聊天（专用会话）
</div>

完成设置后，应用会打开一个专用的入门聊天会话，让智能体可以
自我介绍并引导后续操作。这样可以将首次运行的引导内容与你的
日常对话区分开来。

<div id="agent-bootstrap-ritual">
  ## Agent 引导仪式
</div>

在首次运行某个智能体时，OpenClaw 会创建并引导一个工作区（默认 `~/.openclaw/workspace`）：

* 预置 `AGENTS.md`、`BOOTSTRAP.md`、`IDENTITY.md`、`USER.md`
* 运行一个简短的问答仪式（一次提一个问题）
* 将身份信息和偏好写入 `IDENTITY.md`、`USER.md`、`SOUL.md`
* 完成后删除 `BOOTSTRAP.md`，以确保该流程只运行一次

<div id="optional-gmail-hooks-manual">
  ## 可选：Gmail 钩子（手动）
</div>

目前 Gmail Pub/Sub 的设置需要手动完成。请使用：

```bash
openclaw webhooks gmail setup --account you@gmail.com
```

详情请参见 [/automation/gmail-pubsub](/zh/automation/gmail-pubsub)。

<div id="remote-mode-notes">
  ## 远程模式说明
</div>

当 Gateway 在另一台机器上运行时，凭据和工作区文件会保存在
**那台主机上**。如果你在远程模式下需要 OAuth，请在以下位置创建：

* `~/.openclaw/credentials/oauth.json`
* `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`

以上路径均是指 Gateway 所在主机上的位置。