---
title: Qwen
summary: "在 OpenClaw 中使用 Qwen OAuth（免费档位）"
read_when:
  - 你想在 OpenClaw 中使用 Qwen
  - 你想通过 OAuth 使用 Qwen Coder 的免费档位
---

<div id="qwen">
  # Qwen
</div>

Qwen 为 Qwen Coder 和 Qwen Vision 模型提供免费层级的 OAuth 流程
（每天 2,000 个请求，受 Qwen 速率限制约束）。

<div id="enable-the-plugin">
  ## 启用插件
</div>

```bash
openclaw plugins enable qwen-portal-auth
```

启用后重启 Gateway。

<div id="authenticate">
  ## 身份验证
</div>

```bash
openclaw models auth login --provider qwen-portal --set-default
```

这将运行 Qwen 的设备代码 OAuth 授权流程，并在你的 `models.json` 中写入一个提供方配置条目（外加一个用于快速切换的 `qwen` 别名）。

<div id="model-ids">
  ## 模型 ID
</div>

* `qwen-portal/coder-model`
* `qwen-portal/vision-model`

使用以下命令切换模型：

```bash
openclaw models set qwen-portal/coder-model
```

<div id="reuse-qwen-code-cli-login">
  ## 复用 Qwen Code CLI 登录状态
</div>

如果你已经使用 Qwen Code CLI 登录，OpenClaw 在加载认证存储时会从
`~/.qwen/oauth_creds.json` 中同步凭证。你仍然需要一个
`models.providers.qwen-portal` 条目（使用上面的登录命令来创建它）。

<div id="notes">
  ## 说明
</div>

* Token 会自动刷新；如果刷新失败或访问被撤销，请重新运行登录命令。
* 默认基础 URL：`https://portal.qwen.ai/v1`（如果 Qwen 提供了不同的端点，可通过
  `models.providers.qwen-portal.baseUrl` 进行覆盖）。
* 参见 [模型提供方](/zh/concepts/model-providers) 了解适用于所有提供方的通用规则。