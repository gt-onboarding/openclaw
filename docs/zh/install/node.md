---
title: "Node.js + npm（PATH 环境自检）"
summary: "Node.js + npm 安装自检：版本、PATH 和全局安装"
read_when:
  - "你已经安装了 OpenClaw，但执行 `openclaw` 时提示“command not found”"
  - "你正在一台新机器上配置 Node.js/npm"
  - "npm install -g ... 因权限或 PATH 问题而失败"
---

<div id="nodejs-npm-path-sanity">
  # Node.js + npm（PATH 正常性检查）
</div>

OpenClaw 的运行时基线是 **Node 22+**。

如果你能运行 `npm install -g openclaw@latest`，但后来看到 `openclaw: command not found`，几乎可以肯定是一个 **PATH** 配置问题：npm 安装全局可执行文件的目录没有被加入到你当前 shell 的 PATH 中。

<div id="quick-diagnosis">
  ## 快速诊断
</div>

运行：

```bash
node -v
npm -v
npm prefix -g
echo "$PATH"
```

如果在执行 `echo "$PATH"` 的输出中 **没有** 出现 `$(npm prefix -g)/bin`（macOS/Linux）或 `$(npm prefix -g)`（Windows），你的 shell 就无法找到全局安装的 npm 可执行文件（包括 `openclaw`）。


<div id="fix-put-npms-global-bin-dir-on-path">
  ## 解决方案：将 npm 的全局 bin 目录添加到 PATH 中
</div>

1. 查找你的 npm 全局前缀路径：

```bash
npm prefix -g
```

2. 将全局 npm bin 目录添加到你的 shell 启动文件中：

* zsh：`~/.zshrc`
* bash：`~/.bashrc`

示例（将路径替换为你运行 `npm prefix -g` 时的输出结果）：

```bash
# macOS / Linux
export PATH="/path/from/npm/prefix/bin:$PATH"
```

然后打开一个**新的终端窗口**（或者在 zsh 中运行 `rehash`，在 bash 中运行 `hash -r`）。

在 Windows 上，将 `npm prefix -g` 的输出结果添加到你的 PATH 环境变量中。


<div id="fix-avoid-sudo-npm-install-g-permission-errors-linux">
  ## 解决方案：避免使用 `sudo npm install -g` / 权限错误（Linux）
</div>

如果运行 `npm install -g ...` 时出现 `EACCES` 错误，将 npm 的全局前缀改为用户可写的目录：

```bash
mkdir -p "$HOME/.npm-global"
npm config set prefix "$HOME/.npm-global"
export PATH="$HOME/.npm-global/bin:$PATH"
```

将 `export PATH=...` 这一行写入你的 shell 启动文件，使其持久生效。


<div id="recommended-node-install-options">
  ## 推荐的 Node 安装方式
</div>

如果你按下面的方式安装 Node/npm，可以尽量减少遇到问题的概率：

- 保持 Node 版本及时更新（22+）
- 确保全局 npm bin 目录稳定可用，并且在新打开的 shell 中已经加入 PATH

常见选择：

- macOS：使用 Homebrew（`brew install node`）或版本管理器
- Linux：使用你偏好的版本管理器，或发行版支持的、能提供 Node 22+ 的安装方式
- Windows：官方 Node 安装程序、`winget`，或 Windows 上的 Node 版本管理器

如果你使用版本管理器（nvm/fnm/asdf/等），请确保它已经在你日常使用的 shell 中（zsh 或 bash）完成初始化，这样在你运行安装命令时，它所设置的 PATH 就会生效。