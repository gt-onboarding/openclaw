---
title: 疑难排查
summary: "疑难排查中心：症状 → 检查 → 修复"
read_when:
  - 你看到错误并想知道如何修复
  - 安装程序显示“成功”，但 CLI 无法工作
---

<div id="troubleshooting">
  # 故障排查
</div>

<div id="first-60-seconds">
  ## 前 60 秒
</div>

按顺序执行以下操作：

```bash
openclaw status
openclaw status --all
openclaw gateway probe
openclaw logs --follow
openclaw doctor
```

如果 Gateway 可访问，则执行深度探测：

```bash
openclaw status --deep
```

<div id="common-it-broke-cases">
  ## 常见“出问题”的情形
</div>

<div id="openclaw-command-not-found">
  ### `openclaw: command not found`
</div>

几乎总是 Node/npm 的 PATH 环境变量配置问题。从这里开始排查：

* [安装（Node/npm PATH 检查）](/zh/install#nodejs--npm-path-sanity)

<div id="installer-fails-or-you-need-full-logs">
  ### 安装程序失败（或你需要完整日志）
</div>

以详细输出模式重新运行安装程序，以查看完整的调用栈和 npm 输出：

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --verbose
```

对于 Beta 安装：

```bash
curl -fsSL https://openclaw.bot/install.sh | bash -s -- --beta --verbose
```

你也可以设置 `OPENCLAW_VERBOSE=1`，而不是使用该命令行参数。

<div id="gateway-unauthorized-cant-connect-or-keeps-reconnecting">
  ### Gateway 显示“unauthorized”、无法连接，或频繁重连
</div>

* [Gateway 故障排查](/zh/gateway/troubleshooting)
* [Gateway 身份验证](/zh/gateway/authentication)

<div id="control-ui-fails-on-http-device-identity-required">
  ### Control UI 在 HTTP 下无法使用（需要设备身份验证）
</div>

* [Gateway 故障排查](/zh/gateway/troubleshooting)
* [Control UI](/zh/web/control-ui#insecure-http)

<div id="docsopenclawai-shows-an-ssl-error-comcastxfinity">
  ### `docs.openclaw.ai` 显示 SSL 错误（Comcast/Xfinity）
</div>

部分使用 Comcast/Xfinity 的连接会通过 Xfinity Advanced Security 阻止访问 `docs.openclaw.ai`。
请禁用 Advanced Security，或将 `docs.openclaw.ai` 添加到允许列表后再重试。

* Xfinity Advanced Security 帮助文档：https://www.xfinity.com/support/articles/using-xfinity-xfi-advanced-security
* 快速排查：尝试使用手机热点或 VPN，以确认是否为运营商（ISP）级别的过滤

<div id="service-says-running-but-rpc-probe-fails">
  ### 服务显示为正在运行，但 RPC 探测失败
</div>

* [Gateway 故障排查](/zh/gateway/troubleshooting)
* [后台进程 / 服务](/zh/gateway/background-process)

<div id="modelauth-failures-rate-limit-billing-all-models-failed">
  ### 模型/认证错误（速率限制、计费、“所有模型均失败”）
</div>

* [模型](/zh/cli/models)
* [OAuth / 认证概念](/zh/concepts/oauth)

<div id="model-says-model-not-allowed">
  ### `/model` 显示 `model not allowed`
</div>

这通常意味着 `agents.defaults.models` 被配置为一个允许列表。当它非空时，
只有这些提供方/模型键可以被选择。

* 检查允许列表：`openclaw config get agents.defaults.models`
* 添加你想要的模型（或清空允许列表），然后重试 `/model`
* 使用 `/models` 浏览被允许的提供方/模型

<div id="when-filing-an-issue">
  ### 提交 Issue 时
</div>

粘贴一份经过脱敏的报告：

```bash
openclaw status --all
```

如果可以，请附上执行 `openclaw logs --follow` 时得到的相关日志尾部输出。
