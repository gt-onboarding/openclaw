---
title: 认证
summary: "模型认证：OAuth、API 密钥和 setup-token"
read_when:
  - 排查模型认证或 OAuth 过期问题
  - 编写关于认证或凭据存储的文档
---

<div id="authentication">
  # 身份验证
</div>

OpenClaw 支持使用 OAuth 和 API key 对模型提供方进行身份验证。对于 Anthropic
账户，我们推荐使用 **API key**。对于 Claude 订阅服务访问，
请使用通过 `claude setup-token` 创建的长期令牌。

请参阅 [/concepts/oauth](/zh/concepts/oauth)，了解完整的 OAuth 流程和存储布局。

<div id="recommended-anthropic-setup-api-key">
  ## 推荐的 Anthropic 配置（API 密钥）
</div>

如果你直接使用 Anthropic，请使用 API 密钥。

1. 在 Anthropic Console 中创建一个 API 密钥。
2. 将它配置在 **Gateway 主机**上（运行 `openclaw gateway` 的那台机器）。

```bash
export ANTHROPIC_API_KEY="..."
openclaw models status
```

3. 如果 Gateway 在 systemd/launchd 下运行，建议将密钥放在
   `~/.openclaw/.env` 中，以便守护进程可以读取该密钥：

```bash
cat >> ~/.openclaw/.env <<'EOF'
ANTHROPIC_API_KEY=...
EOF
```

然后重启守护进程（或重启你的 Gateway 进程），然后再次检查：

```bash
openclaw models status
openclaw doctor
```

如果你不想手动管理环境变量，可以使用入门向导来为守护进程存储 API 密钥：`openclaw onboard`。

有关环境变量继承（`env.shellEnv`、`~/.openclaw/.env`、systemd/launchd）的详细信息，请参见 [帮助](/zh/help)。

<div id="anthropic-setup-token-subscription-auth">
  ## Anthropic：setup-token（订阅认证）
</div>

对于 Anthropic，推荐的首选方式是使用 **API key**。如果你在使用 Claude
订阅，也支持 setup-token 流程。在 **Gateway 所在主机** 上运行：

```bash
claude setup-token
```

然后将其粘贴到 OpenClaw：

```bash
openclaw models auth setup-token --provider anthropic
```

如果该 token 是在另一台机器上生成的，请手动粘贴：

```bash
openclaw models auth paste-token --provider anthropic
```

如果你遇到类似如下的 Anthropic 错误：

```
此凭证仅授权用于 Claude Code,无法用于其他 API 请求。
```

…改为使用 Anthropic API 密钥。

手动输入令牌（适用于任意提供方；会写入 `auth-profiles.json` 并更新配置）：

```bash
openclaw models auth paste-token --provider anthropic
openclaw models auth paste-token --provider openrouter
```

便于自动化的检查（过期或缺失时退出码为 `1`，临近过期时退出码为 `2`）：

```bash
openclaw models status --check
```

可选的运维脚本（systemd/Termux）在此处有说明：
[/automation/auth-monitoring](/zh/automation/auth-monitoring)

> `claude setup-token` 需要一个交互式 TTY 终端。

<div id="checking-model-auth-status">
  ## 检查模型鉴权状态
</div>

```bash
openclaw models status
openclaw doctor
```

<div id="controlling-which-credential-is-used">
  ## 控制使用哪种凭证
</div>

<div id="per-session-chat-command">
  ### 按会话（聊天命令）
</div>

使用 `/model <alias-or-id>@<profileId>` 为当前会话绑定一个特定的提供方凭据（示例 profile id：`anthropic:default`、`anthropic:work`）。

使用 `/model`（或 `/model list`）打开一个简洁的选择器；使用 `/model status` 查看完整视图（候选项 + 下一个认证配置文件，以及在已配置时的提供方端点详情）。

<div id="per-agent-cli-override">
  ### 按智能体（CLI 覆写）
</div>

为某个智能体设置显式的认证配置文件顺序覆写（保存在该智能体的 `auth-profiles.json` 中）：

```bash
openclaw models auth order get --provider anthropic
openclaw models auth order set --provider anthropic anthropic:default
openclaw models auth order clear --provider anthropic
```

使用 `--agent &lt;id&gt;` 指定要操作的特定智能体；省略该参数时将使用已配置的默认智能体。

<div id="troubleshooting">
  ## 故障排除
</div>

<div id="no-credentials-found">
  ### “未找到凭据”
</div>

如果缺少 Anthropic token 配置文件，请在 **Gateway 主机** 上运行 `claude setup-token`，然后再次检查。

```bash
openclaw models status
```

<div id="token-expiringexpired">
  ### 令牌即将过期/已过期
</div>

运行 `openclaw models status` 以确认哪个配置文件的令牌即将过期。\
如果该配置文件不存在，请重新运行 `claude setup-token` 并再次粘贴令牌。

<div id="requirements">
  ## 前提条件
</div>

* Claude Max 或 Pro 订阅计划（用于执行 `claude setup-token`）
* 已安装 Claude Code CLI（可在命令行中使用 `claude` 命令）