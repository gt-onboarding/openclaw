---
title: 插件
summary: "CLI `openclaw plugins` 参考（列出、安装、启用/禁用、诊断）"
read_when:
  - 当你需要安装或管理在 Gateway 进程内运行的插件时
  - 当你需要排查插件加载失败问题时
---

<div id="openclaw-plugins">
  # `openclaw plugins`
</div>

管理 Gateway 插件/扩展（在同一进程内加载）。

相关内容：

* 插件系统：[插件](/zh/plugin)
* 插件清单与模式：[插件清单](/zh/plugins/manifest)
* 安全加固：[安全](/zh/gateway/security)

<div id="commands">
  ## 命令
</div>

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins doctor
openclaw plugins update <id>
openclaw plugins update --all
```

内置插件会随 OpenClaw 一起提供，但默认处于禁用状态。使用 `plugins enable`
来启用它们。

所有插件都必须包含一个 `openclaw.plugin.json` 文件，并在其中内嵌一个 JSON Schema
（`configSchema`，即使为空）。缺失或无效的清单文件或 Schema 会阻止插件加载，并导致配置校验失败。

<div id="install">
  ### 安装
</div>

```bash
openclaw plugins install <path-or-spec>
```

安全提示：对待插件安装要像运行代码一样谨慎。优先使用固定版本。

支持的归档格式：`.zip`、`.tgz`、`.tar.gz`、`.tar`。

使用 `--link` 可以避免复制本地目录（会将其添加到 `plugins.load.paths`）：

```bash
openclaw plugins install -l ./my-plugin
```

<div id="update">
  ### 更新
</div>

```bash
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins update <id> --dry-run
```

更新仅适用于通过 npm 安装的插件（记录在 `plugins.installs` 中）。
