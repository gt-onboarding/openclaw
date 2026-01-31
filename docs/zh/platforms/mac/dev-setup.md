---
title: 开发环境搭建
summary: "面向参与开发 OpenClaw macOS 应用的开发者的环境搭建指南"
read_when:
  - 搭建 macOS 开发环境时
---

<div id="macos-developer-setup">
  # macOS 开发环境设置
</div>

本指南介绍从源代码构建并运行 OpenClaw 的 macOS 应用程序所需的步骤。

<div id="prerequisites">
  ## 前提条件
</div>

在构建应用之前，请确保已安装以下依赖：

1.  **Xcode 26.2+**：用于 Swift 开发。
2.  **Node.js 22+ 和 pnpm**：用于 Gateway、CLI 和打包脚本。

<div id="1-install-dependencies">
  ## 1. 安装依赖项
</div>

安装整个项目所需的依赖：

```bash
pnpm install
```


<div id="2-build-and-package-the-app">
  ## 2. 构建并打包应用
</div>

要构建 macOS 应用并将其打包为 `dist/OpenClaw.app`，运行：

```bash
./scripts/package-mac-app.sh
```

如果你没有 Apple Developer ID 证书，脚本会自动使用 **临时签名（ad-hoc signing）**（`-`）。

关于开发运行模式、签名标志和 Team ID 故障排查，请参阅 macOS 应用的 README：
https://github.com/openclaw/openclaw/blob/main/apps/macos/README.md

> **注意**：使用临时签名的应用可能会触发安全提示。如果应用在启动后立即崩溃并显示“Abort trap 6”错误，请查看 [故障排查](#troubleshooting) 部分。


<div id="3-install-the-cli">
  ## 3. 安装 CLI
</div>

macOS 应用程序需要全局安装 `openclaw` CLI，用于管理后台任务。

**安装方式（推荐）：**

1. 打开 OpenClaw 应用。
2. 前往 **General** 设置选项卡。
3. 点击 **“Install CLI”**。

或者手动安装：

```bash
npm install -g openclaw@<version>
```


<div id="troubleshooting">
  ## 故障排查
</div>

<div id="build-fails-toolchain-or-sdk-mismatch">
  ### 构建失败：工具链或 SDK 不匹配
</div>

macOS 应用构建需要最新的 macOS SDK 和 Swift 6.2 工具链。

**系统依赖项（必需）：**

* **通过“软件更新”提供的最新 macOS 版本**（Xcode 26.2 SDK 的要求）
* **Xcode 26.2**（Swift 6.2 工具链）

**检查：**

```bash
xcodebuild -version
xcrun swift --version
```

如果版本不匹配，请更新 macOS/Xcode，并重新进行构建。


<div id="app-crashes-on-permission-grant">
  ### 在授予权限时应用崩溃
</div>

如果在尝试允许 **Speech Recognition** 或 **Microphone** 访问权限时应用程序发生崩溃，可能是由于 TCC 缓存损坏或签名不匹配导致的。

**修复方法：**

1. 重置 TCC 权限：
   ```bash
   tccutil reset All bot.molt.mac.debug
   ```
2. 如果这仍然不起作用，在 [`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) 中临时修改 `BUNDLE_ID`，以强制让 macOS 将其视为一个“全新应用”。

<div id="gateway-starting-indefinitely">
  ### Gateway 一直处于 &quot;Starting...&quot; 状态
</div>

如果 Gateway 状态始终显示 &quot;Starting...&quot;，请检查是否有僵尸进程占用了该端口：

```bash
openclaw gateway status
openclaw gateway stop

# 如果你没有使用 LaunchAgent(开发模式/手动运行),请查找监听器:
lsof -nP -iTCP:18789 -sTCP:LISTEN
```

如果是手动启动的进程占用了该端口，请停止该进程（Ctrl+C）。如果仍无效，最后可以杀掉你在上面找到的那个 PID。
