---
title: Nix
summary: "使用 Nix 以声明式方式安装 OpenClaw"
read_when:
  - 你希望安装过程可复现且支持回滚
  - 你已经在使用 Nix/NixOS/Home Manager
  - 你希望所有内容都锁定版本，并通过声明式方式统一管理
---

<div id="nix-installation">
  # 使用 Nix 安装
</div>

推荐的在 Nix 环境中运行 OpenClaw 的方式是通过 **[nix-openclaw](https://github.com/openclaw/nix-openclaw)** —— 一个开箱即用的 Home Manager 模块。

<div id="quick-start">
  ## 快速开始
</div>

将以下内容粘贴到你的 AI Agent 代理（Claude、Cursor 等）中：

```text
I want to set up nix-openclaw on my Mac.
Repository: github:openclaw/nix-openclaw

What I need you to do:
1. Check if Determinate Nix is installed (if not, install it)
2. Create a local flake at ~/code/openclaw-local using templates/agent-first/flake.nix
3. Help me create a Telegram bot (@BotFather) and get my chat ID (@userinfobot)
4. Set up secrets (bot token, Anthropic key) - plain files at ~/.secrets/ is fine
5. Fill in the template placeholders and run home-manager switch
6. Verify: launchd running, bot responds to messages

Reference the nix-openclaw README for module options.
```

> **📦 完整指南：[github.com/openclaw/nix-openclaw](https://github.com/openclaw/nix-openclaw)**
>
> nix-openclaw 仓库是 Nix 安装的权威参考来源。本页只是一个快速概览。

<div id="what-you-get">
  ## 你将获得
</div>

* Gateway + macOS 应用 + 工具（whisper、spotify、摄像头）——全部版本已锁定
* 可在重启后自动恢复的 launchd 服务
* 具有声明式配置的插件系统
* 即时回滚：`home-manager switch --rollback`

***

<div id="nix-mode-runtime-behavior">
  ## Nix 模式运行时行为
</div>

当设置 `OPENCLAW_NIX_MODE=1` 时（使用 nix-openclaw 时会自动设置）：

OpenClaw 支持一种 **Nix 模式**，该模式使配置结果具备确定性，并禁用自动安装流程。
通过导出以下环境变量来启用它：

```bash
OPENCLAW_NIX_MODE=1
```

在 macOS 上，GUI 应用程序不会自动继承 shell 的环境变量。你也可以通过 defaults 命令启用 Nix 模式：

```bash
defaults write bot.molt.mac openclaw.nixMode -bool true
```

<div id="config-state-paths">
  ### 配置与状态路径
</div>

OpenClaw 从 `OPENCLAW_CONFIG_PATH` 读取 JSON5 配置文件，并将可变数据存储在 `OPENCLAW_STATE_DIR` 中。

* `OPENCLAW_STATE_DIR`（默认值：`~/.openclaw`）
* `OPENCLAW_CONFIG_PATH`（默认值：`$OPENCLAW_STATE_DIR/openclaw.json`）

在 Nix 环境下运行时，应将这些变量显式设置为由 Nix 管理的位置，以避免将运行时状态和配置写入不可变存储。

<div id="runtime-behavior-in-nix-mode">
  ### Nix 模式下的运行时行为
</div>

* 自动安装和自我更新流程被禁用
* 缺少依赖项时会显示 Nix 特有的修复提示信息
* 当启用时，UI 会显示只读 Nix 模式横幅

<div id="packaging-note-macos">
  ## 打包说明（macOS）
</div>

macOS 的打包流程预期在以下位置存在一个固定的 Info.plist 模板：

```
apps/macos/Sources/OpenClaw/Resources/Info.plist
```

[`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) 会将此模板复制到应用 bundle 中，并填充/更新动态字段
（bundle ID、version/build、Git SHA、Sparkle keys）。这样可以让 plist 在 SwiftPM
打包和 Nix 构建中保持确定性（这些构建方式不依赖完整的 Xcode 工具链）。

<div id="related">
  ## 相关
</div>

* [nix-openclaw](https://github.com/openclaw/nix-openclaw) — 完整配置指南
* [Wizard](/zh/start/wizard) — 非 Nix 的 CLI 配置
* [Docker](/zh/install/docker) — 容器化部署