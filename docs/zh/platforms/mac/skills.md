---
title: 技能
summary: "macOS 技能设置 UI 与由 Gateway 管理的状态"
read_when:
  - 更新 macOS 技能设置 UI 时
  - 更改技能访问控制或安装行为时
---

<div id="skills-macos">
  # 技能（macOS）
</div>

macOS 应用通过 Gateway 提供 OpenClaw 技能；不会在本地解析这些技能。

<div id="data-source">
  ## 数据源
</div>

- `skills.status`（Gateway）会返回所有技能，以及其可用性和缺失的前置条件
  （包括针对内置技能的允许列表拦截信息）。
- 前置条件来自每个 `SKILL.md` 中的 `metadata.openclaw.requires` 字段。

<div id="install-actions">
  ## 安装操作
</div>

- `metadata.openclaw.install` 定义安装选项（brew/node/go/uv）。
- 应用会调用 `skills.install`，在 Gateway 主机上运行安装程序。
- 当提供多种安装方式时，Gateway 只会显示一个首选安装器
  （如果可用则优先使用 brew，否则使用来自 `skills.install` 的 Node 管理器，默认使用 npm）。

<div id="envapi-keys">
  ## 环境变量 / API 密钥
</div>

- 应用会在 `~/.openclaw/openclaw.json` 的 `skills.entries.<skillKey>` 下存储密钥。
- `skills.update` 会更新 `enabled`、`apiKey` 和 `env` 字段。

<div id="remote-mode">
  ## 远程模式
</div>

- 安装和配置更新在运行 Gateway 的主机上进行（而不是在本地 Mac 上）。