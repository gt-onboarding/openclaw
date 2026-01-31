---
title: 安装
summary: "安装 OpenClaw（推荐安装方式、全局安装或从源码安装）"
read_when:
  - 安装 OpenClaw
  - 你希望从 GitHub 安装
---

<div id="install">
  # 安装
</div>

除非有特殊原因，否则请使用安装程序。它会安装 CLI 并引导你完成首次配置。

<div id="quick-install-recommended">
  ## 快速安装（推荐）
</div>

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

Windows（PowerShell）：

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

下一步（如果你跳过了入门引导）：

```bash
openclaw onboard --install-daemon
```

<div id="system-requirements">
  ## 系统要求
</div>

* **Node.js &gt;= 22**
* macOS、Linux，或通过 WSL2 运行的 Windows
* 仅在从源代码构建时才需要 `pnpm`

<div id="choose-your-install-path">
  ## 选择你的安装方式
</div>

<div id="1-installer-script-recommended">
  ### 1) 安装脚本（推荐）
</div>

通过 npm 全局安装 `openclaw` 并启动上手引导流程。

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

安装命令参数：

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --help
```

详细信息：[安装器内部机制](/zh/install/installer)。

非交互模式（跳过引导流程）：

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --no-onboard
```

<div id="2-global-install-manual">
  ### 2) 全局安装（手动）
</div>

如果你已经安装了 Node.js：

```bash
npm install -g openclaw@latest
```

如果你在系统中已全局安装了 libvips（在 macOS 上通过 Homebrew 安装的情况很常见），但 `sharp` 安装失败，可以强制使用预编译的二进制包：

```bash
SHARP_IGNORE_GLOBAL_LIBVIPS=1 npm install -g openclaw@latest
```

如果你看到 `sharp: Please add node-gyp to your dependencies`，要么安装构建工具（macOS：Xcode CLT + `npm install -g node-gyp`），要么使用上面的 `SHARP_IGNORE_GLOBAL_LIBVIPS=1` 变通方案来跳过本机构建。

或者：

```bash
pnpm add -g openclaw@latest
```

然后：

```bash
openclaw onboard --install-daemon
```

<div id="3-from-source-contributorsdev">
  ### 3) 从源码安装（贡献者/开发者）
</div>

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # 首次运行时自动安装 UI 依赖项
pnpm build
openclaw onboard --install-daemon
```

提示：如果你还没有全局安装，可以通过 `pnpm openclaw ...` 来运行本仓库中的命令。

<div id="4-other-install-options">
  ### 4) 其他安装选项
</div>

* Docker：[Docker](/zh/install/docker)
* Nix：[Nix](/zh/install/nix)
* Ansible：[Ansible](/zh/install/ansible)
* Bun（仅限 CLI）：[Bun](/zh/install/bun)

<div id="after-install">
  ## 安装完成后
</div>

* 运行上手向导：`openclaw onboard --install-daemon`
* 快速检查：`openclaw doctor`
* 检查 Gateway 运行状况：`openclaw status` + `openclaw health`
* 打开仪表盘：`openclaw dashboard`

<div id="install-method-npm-vs-git-installer">
  ## 安装方法：npm 与 git（安装器）
</div>

安装器支持两种安装方式：

* `npm`（默认）：`npm install -g openclaw@latest`
* `git`：从 GitHub 克隆代码并构建，然后在源码检出目录中运行

<div id="cli-flags">
  ### CLI 选项
</div>

```bash
# 显式指定 npm
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method npm

# 从 GitHub 安装（源码检出）
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --install-method git
```

常用参数：

* `--install-method npm|git`
* `--git-dir <path>`（默认：`~/openclaw`）
* `--no-git-update`（在使用已有代码检出目录时跳过 `git pull`）
* `--no-prompt`（禁用交互提示；在 CI/自动化中必须使用）
* `--dry-run`（仅打印将要执行的操作，不做任何更改）
* `--no-onboard`（跳过初始化引导流程）

<div id="environment-variables">
  ### 环境变量
</div>

对应的环境变量（便于自动化使用）：

* `OPENCLAW_INSTALL_METHOD=git|npm`
* `OPENCLAW_GIT_DIR=...`
* `OPENCLAW_GIT_UPDATE=0|1`
* `OPENCLAW_NO_PROMPT=1`
* `OPENCLAW_DRY_RUN=1`
* `OPENCLAW_NO_ONBOARD=1`
* `SHARP_IGNORE_GLOBAL_LIBVIPS=0|1`（默认：`1`；避免让 `sharp` 依赖系统的 libvips 进行构建）

<div id="troubleshooting-openclaw-not-found-path">
  ## 故障排查：无法找到 `openclaw`（PATH）
</div>

快速检查：

```bash
node -v
npm -v
npm prefix -g
echo "$PATH"
```

如果在执行 `echo "$PATH"` 的输出中 **没有** 出现 `$(npm prefix -g)/bin`（macOS/Linux）或 `$(npm prefix -g)`（Windows），说明你的 shell 无法找到全局 npm 可执行文件（包括 `openclaw`）。

解决方法：把它添加到你的 shell 启动脚本中（zsh：`~/.zshrc`，bash：`~/.bashrc`）：

```bash
# macOS / Linux
export PATH="$(npm prefix -g)/bin:$PATH"
```

在 Windows 上，将 `npm prefix -g` 的输出添加到你的 PATH 环境变量。

然后打开一个新的终端（或在 zsh 中运行 `rehash` / 在 bash 中运行 `hash -r`）。

<div id="update-uninstall">
  ## 更新 / 卸载
</div>

* 更新：[更新](/zh/install/updating)
* 迁移到新机器：[迁移](/zh/install/migrating)
* 卸载：[卸载](/zh/install/uninstall)