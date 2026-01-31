---
title: 安装程序
summary: "安装脚本（install.sh + install-cli.sh）的工作原理、参数和自动化"
read_when:
  - 你想要了解 `openclaw.bot/install.sh`
  - 你想要实现自动化安装（CI / 无头模式）
  - 你想要从 GitHub 检出的代码进行安装
---

<div id="installer-internals">
  # 安装器内部机制
</div>

OpenClaw 提供了两个安装脚本（由 `openclaw.ai` 提供）：

* `https://openclaw.bot/install.sh` — “推荐”的安装器（默认使用全局 npm 安装；也可以从 GitHub 检出安装）
* `https://openclaw.bot/install-cli.sh` — 对非 root 用户友好的 CLI 安装器（安装到带有自带 Node 运行时的前缀目录中）
* `https://openclaw.ai/install.ps1` — Windows PowerShell 安装器（默认使用 npm；也可以选择通过 git 安装）

要查看当前的参数/行为，请运行：

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --help
```

Windows（PowerShell）帮助：

```powershell
& ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -?
```

如果安装程序已完成但在新终端中找不到 `openclaw`，通常是 Node.js/npm 的 PATH 配置问题。参见：[安装](/zh/install#nodejs--npm-path-sanity)。

<div id="installsh-recommended">
  ## install.sh（推荐）
</div>

它总体上会执行的操作：

* 检测操作系统（macOS / Linux / WSL）。
* 确保 Node.js **22+**（macOS 通过 Homebrew；Linux 通过 NodeSource）。
* 选择安装方式：
  * `npm`（默认）：`npm install -g openclaw@latest`
  * `git`：克隆/构建源码检出并安装一个包装脚本
* 在 Linux 上：在需要时通过将 npm 的 prefix 切换为 `~/.npm-global` 来避免全局 npm 权限错误。
* 如果是在升级已有安装：运行 `openclaw doctor --non-interactive`（尽最大努力完成）。
* 对于 git 安装：在安装/更新后运行 `openclaw doctor --non-interactive`（尽最大努力完成）。
* 通过将 `SHARP_IGNORE_GLOBAL_LIBVIPS=1` 设为默认值，缓解 `sharp` 原生安装中的常见坑（避免针对系统 libvips 进行构建）。

如果你*希望*让 `sharp` 链接到全局安装的 libvips（或者你在调试），请设置：

```bash
SHARP_IGNORE_GLOBAL_LIBVIPS=0 curl -fsSL https://openclaw.bot/install.sh | bash
```

<div id="discoverability-git-install-prompt">
  ### 可发现性 / “git install” 提示
</div>

如果你在**已经位于 OpenClaw 源码检出目录内**时运行安装脚本（通过同时存在 `package.json` 和 `pnpm-workspace.yaml` 检测），它会提示你：

* 更新并使用当前检出目录（`git`）
* 或迁移到全局 npm 安装（`npm`）

在非交互式场景（无 TTY / 使用 `--no-prompt`）下，你必须传入 `--install-method git|npm`（或设置 `OPENCLAW_INSTALL_METHOD`），否则脚本会以退出码 `2` 退出。

<div id="why-git-is-needed">
  ### 为什么需要 Git
</div>

对于 `--install-method git` 安装方式（clone / pull），需要使用 Git。

对于通过 `npm` 安装的情况，Git *通常* 不是必需的，但在某些环境中最终仍然会需要它（例如，当某个包或依赖是通过 git URL 获取时）。安装脚本目前会确保系统中已安装 Git，以避免在全新安装的发行版上出现 `spawn git ENOENT` 这类意外错误。

<div id="why-npm-hits-eacces-on-fresh-linux">
  ### 为什么 npm 在全新安装的 Linux 上会遇到 `EACCES`
</div>

在某些 Linux 环境中（特别是在通过系统包管理器或 NodeSource 安装 Node 之后），npm 的全局前缀会指向一个由 root 拥有的目录。此时执行 `npm install -g ...` 会因为 `EACCES` / `mkdir` 权限错误而失败。

`install.sh` 通过将前缀切换为以下路径来缓解这个问题：

* `~/.npm-global`（并在存在时将其添加到 `~/.bashrc` / `~/.zshrc` 的 `PATH` 中）

<div id="install-clish-non-root-cli-installer">
  ## install-cli.sh（非 root CLI 安装器）
</div>

此脚本会将 `openclaw` 安装到指定的安装前缀目录（默认：`~/.openclaw`），并在该前缀下安装一个专用的 Node 运行时环境，因此可以在你不希望改动系统 Node/npm 的机器上使用。

帮助：

```bash
curl -fsSL https://openclaw.bot/install-cli.sh | bash -s -- --help
```

<div id="installps1-windows-powershell">
  ## install.ps1 (Windows PowerShell)
</div>

该脚本主要执行以下操作：

* 确保已安装 Node.js **22+**（通过 winget/Chocolatey/Scoop 或手动安装）。
* 选择安装方式：
  * `npm`（默认）：`npm install -g openclaw@latest`
  * `git`：克隆/构建源码并安装封装脚本
* 在升级和 git 安装时运行 `openclaw doctor --non-interactive`（尽最大努力执行）。

示例：

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex -InstallMethod git
```

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex -InstallMethod git -GitDir "C:\\openclaw"
```

环境变量：

* `OPENCLAW_INSTALL_METHOD=git|npm`
* `OPENCLAW_GIT_DIR=...`

Git 要求：

如果你选择 `-InstallMethod git` 且系统中未安装 Git，安装程序会输出 Git for Windows 链接（`https://git-scm.com/download/win`）并退出。

常见 Windows 问题：

* **npm error spawn git / ENOENT**：安装 Git for Windows，重新打开 PowerShell，然后重新运行安装程序。
* **&quot;openclaw&quot; is not recognized**：你的 npm 全局 bin 目录不在 PATH 环境变量中。大多数系统使用
  `%AppData%\\npm`。你也可以运行 `npm config get prefix`，然后将该路径后加上 `\\bin` 添加到 PATH，接着重新打开 PowerShell。
