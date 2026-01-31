---
title: 模型
summary: "用于 `openclaw models` 的 CLI 参考（status/list/set/scan、别名、回退、认证）"
read_when:
  - 当你需要更改默认模型或查看提供方认证/授权状态时
  - 当你需要扫描可用模型/提供方并调试认证配置文件时
---

<div id="openclaw-models">
  # `openclaw models`
</div>

模型发现、扫描与配置（默认模型、备用模型、认证配置文件）。

相关内容：

* 提供方与模型：[模型](/zh/providers/models)
* 提供方认证配置：[入门指南](/zh/start/getting-started)

<div id="common-commands">
  ## 常用命令
</div>

```bash
openclaw models status
openclaw models list
openclaw models set <model-or-alias>
openclaw models scan
```

`openclaw models status` 会显示解析后的默认/回退配置，以及认证（auth）总览。
当提供方使用快照可用时，OAuth/令牌状态部分会包含
提供方使用情况的小节标题。
添加 `--probe` 以对每个已配置的提供方配置文件执行实时认证探测。
探测会发起真实请求（可能会消耗令牌并触发速率限制）。
使用 `--agent <id>` 检查某个已配置智能体的模型/认证状态。省略时，
该命令会优先使用 `OPENCLAW_AGENT_DIR`/`PI_CODING_AGENT_DIR`（如果已设置），否则使用
已配置的默认智能体。

注意：

* `models set <model-or-alias>` 接受 `provider/model` 或别名。
* 模型引用会通过在**第一个** `/` 处分割来解析。如果模型 ID 自身包含 `/`（OpenRouter 风格），则需要包含提供方前缀（示例：`openrouter/moonshotai/kimi-k2`）。
* 如果你省略了提供方，OpenClaw 会将输入视为**默认提供方**的别名或模型（仅在模型 ID 中没有 `/` 时有效）。

<div id="models-status">
  ### `models status`
</div>

选项：

* `--json`
* `--plain`
* `--check`（退出码 1=已过期/缺失，2=即将过期）
* `--probe`（对已配置的认证配置进行实时探测）
* `--probe-provider <name>`（探测单个提供方）
* `--probe-profile <id>`（可重复传入，或使用逗号分隔的配置文件 ID 列表）
* `--probe-timeout <ms>`
* `--probe-concurrency <n>`
* `--probe-max-tokens <n>`
* `--agent <id>`（已配置的智能体 ID；会覆盖 `OPENCLAW_AGENT_DIR`/`PI_CODING_AGENT_DIR`）

<div id="aliases-fallbacks">
  ## 别名与后备
</div>

```bash
openclaw models aliases list
openclaw models fallbacks list
```

<div id="auth-profiles">
  ## 认证配置集
</div>

```bash
openclaw models auth add
openclaw models auth login --provider <id>
openclaw models auth setup-token
openclaw models auth paste-token
```

`models auth login` 会运行提供方插件的身份验证流程（OAuth/API key）。使用
`openclaw plugins list` 查看已安装的提供方列表。

注意：

* `setup-token` 会提示你输入 setup-token 值（可在任意机器上运行 `claude setup-token` 生成）。
* `paste-token` 接受在其他地方或通过自动化流程生成的 token 字符串。
