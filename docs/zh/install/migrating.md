---
title: 迁移
summary: "将 OpenClaw 安装从一台机器迁移到另一台机器"
read_when:
  - 你正在将 OpenClaw 迁移到新的笔记本电脑或服务器
  - 你希望保留会话、认证配置以及通道登录状态（如 WhatsApp 等）
---

<div id="migrating-openclaw-to-a-new-machine">
  # 将 OpenClaw 迁移到新机器
</div>

本指南介绍如何在**不重新执行初始引导（onboarding）流程**的情况下，将一台机器上的 OpenClaw Gateway 迁移到另一台机器。

从概念上讲，迁移非常简单：

* 拷贝 **state 目录**（`$OPENCLAW_STATE_DIR`，默认：`~/.openclaw/`）——其中包含配置（config）、认证信息（auth）、会话（sessions）以及通道状态（channel state）。
* 拷贝你的 **workspace**（默认：`~/.openclaw/workspace/`）——其中包含你的智能体文件（memory、prompts 等）。

但在 **profiles**、**权限** 和 **部分拷贝** 上有一些常见的坑。

<div id="before-you-start-what-you-are-migrating">
  ## 在开始之前（要迁移的内容）
</div>

<div id="1-identify-your-state-directory">
  ### 1) 确定你的状态目录（state directory）
</div>

大多数安装使用默认路径：

* **状态目录：** `~/.openclaw/`

但如果你使用了下面的方式，路径可能会不同：

* `--profile <name>`（通常会变成 `~/.openclaw-<profile>/`）
* `OPENCLAW_STATE_DIR=/some/path`

如果你不确定，请在**旧**机器上运行：

```bash
openclaw status
```

在输出中查找 `OPENCLAW_STATE_DIR` / 配置文件（profile）的相关信息。如果你运行了多个 Gateway，请针对每个 profile 重复执行此操作。

<div id="2-identify-your-workspace">
  ### 2) 确认你的工作区
</div>

常见默认位置：

* `~/.openclaw/workspace/`（推荐工作区）
* 你自己创建的自定义文件夹

你的工作区是存放 `MEMORY.md`、`USER.md` 和 `memory/*.md` 等文件的地方。

<div id="3-understand-what-you-will-preserve">
  ### 3) 理解你会保留哪些内容
</div>

如果你同时复制 **状态目录** 和 **工作区**，你会保留：

* Gateway 配置（`openclaw.json`）
* 认证配置文件 / API 密钥 / OAuth 令牌
* 会话历史 + 智能体状态
* 通道状态（例如 WhatsApp 登录/会话）
* 你的工作区文件（memory、技能笔记等）

如果你 **只** 复制工作区（例如通过 Git），你 **不会** 保留：

* 会话
* 凭据
* 通道登录

这些内容位于 `$OPENCLAW_STATE_DIR` 下。

<div id="migration-steps-recommended">
  ## 推荐迁移步骤
</div>

<div id="step-0-make-a-backup-old-machine">
  ### 步骤 0 — 创建备份（旧机器）
</div>

在**旧**机器上，先停止 Gateway，防止在复制过程中文件发生变更：

```bash
openclaw gateway stop
```

（可选，但推荐）归档 state 目录和工作区：

```bash
# 如果使用配置文件或自定义位置,请调整路径
cd ~
tar -czf openclaw-state.tgz .openclaw

tar -czf openclaw-workspace.tgz .openclaw/workspace
```

如果你有多个配置文件/状态目录（例如 `~/.openclaw-main`、`~/.openclaw-work`），请分别归档每一个目录。

<div id="step-1-install-openclaw-on-the-new-machine">
  ### 步骤 1 — 在新机器上安装 OpenClaw
</div>

在**新**机器上安装 CLI（如有需要，再安装 Node）：

* 参见：[安装](/zh/install)

此时，即使初始化流程创建了一个全新的 `~/.openclaw/` 目录也没问题 —— 你会在下一步把它覆盖掉。

<div id="step-2-copy-the-state-dir-workspace-to-the-new-machine">
  ### 步骤 2 — 将状态目录和工作区复制到新机器
</div>

复制这 **两个**：

* `$OPENCLAW_STATE_DIR`（默认 `~/.openclaw/`）
* 你的工作区（默认 `~/.openclaw/workspace/`）

常用方式：

* 使用 `scp` 传输压缩包并解压
* 通过 SSH 使用 `rsync -a`
* 使用外置硬盘

复制完成后，确保：

* 已包含隐藏目录（例如 `.openclaw/`）
* 文件属主/权限正确，且归运行 Gateway 的用户所有

<div id="step-3-run-doctor-migrations-service-repair">
  ### 第 3 步 — 运行 Doctor（迁移 + 服务修复）
</div>

在**新**机器上运行：

```bash
openclaw doctor
```

Doctor 是一个“安全但无聊”的命令。它会修复服务、执行配置迁移，并在发现不匹配时发出警告。

然后：

```bash
openclaw gateway restart
openclaw status
```

<div id="common-footguns-and-how-to-avoid-them">
  ## 常见踩坑点（以及如何避免）
</div>

<div id="footgun-profile-state-dir-mismatch">
  ### 坑点：profile / state-dir 不匹配
</div>

如果你在旧 Gateway 上是通过 profile（或 `OPENCLAW_STATE_DIR`）运行的，而新 Gateway 使用了不同的 profile/state 目录，你会看到类似下面的现象：

* 配置更改没有生效
* 通道缺失 / 已退出登录
* 会话历史为空

修复方法：使用你迁移过来的 **同一个** profile/state 目录来运行 Gateway/服务，然后重新执行：

```bash
openclaw doctor
```

<div id="footgun-copying-only-openclawjson">
  ### 坑点：仅复制 `openclaw.json`
</div>

`openclaw.json` 远远不够。许多提供方会将状态存储在以下路径：

* `$OPENCLAW_STATE_DIR/credentials/`
* `$OPENCLAW_STATE_DIR/agents/<agentId>/...`

务必迁移整个 `$OPENCLAW_STATE_DIR` 目录。

<div id="footgun-permissions-ownership">
  ### 大坑：权限 / 所有权
</div>

如果你在复制文件时使用了 root 身份，或者切换过用户，Gateway 可能无法读取凭据/会话。

解决方法：确保 state 目录和工作区归运行 Gateway 的用户所有。

<div id="footgun-migrating-between-remotelocal-modes">
  ### 坑点：在远程/本地模式之间迁移
</div>

* 如果你的 UI（WebUI/TUI）指向一个**远程** Gateway，那么会话存储和工作区都归远程主机所有。
* 迁移你的笔记本电脑并不会迁移远程 Gateway 的状态。

如果你处于远程模式，请迁移**运行 Gateway 的主机**。

<div id="footgun-secrets-in-backups">
  ### 危险陷阱：备份中的敏感信息
</div>

`$OPENCLAW_STATE_DIR` 包含敏感信息（API 密钥、OAuth 令牌、WhatsApp 凭据）。请像对待生产环境中的机密一样对待备份：

* 加密存储
* 避免通过不安全的通道传输或共享
* 如怀疑发生泄露，应轮换密钥

<div id="verification-checklist">
  ## 验证清单
</div>

在新机器上检查并确认：

* `openclaw status` 显示 Gateway 正在运行
* 你的通道仍保持连接（例如 WhatsApp 无需重新配对）
* 控制面板可以打开并显示现有会话
* 你的工作区文件（memory、配置）均已存在

<div id="related">
  ## 相关内容
</div>

* [Doctor](/zh/gateway/doctor)
* [Gateway 故障排查](/zh/gateway/troubleshooting)
* [OpenClaw 会把数据存储在哪里？](/zh/help/faq#where-does-openclaw-store-its-data)