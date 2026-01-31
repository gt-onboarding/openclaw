---
title: Googlechat
summary: "Google Chat 应用支持状态、功能与配置"
read_when:
  - 在处理 Google Chat 渠道功能
---

<div id="google-chat-chat-api">
  # Google Chat（Chat API）
</div>

状态：已支持通过 Google Chat API webhooks（仅限 HTTP）处理私信（DMs）和空间（spaces）。

<div id="quick-setup-beginner">
  ## 快速设置（初学者）
</div>

1. 创建一个 Google Cloud 项目并启用 **Google Chat API**。
   * 前往：[Google Chat API Credentials](https://console.cloud.google.com/apis/api/chat.googleapis.com/credentials)
   * 如果尚未启用该 API，请先启用。
2. 创建一个 **Service Account（服务账号）**：
   * 点击 **Create Credentials** &gt; **Service Account**。
   * 可随意命名（例如：`openclaw-chat`）。
   * 权限留空（点击 **Continue**）。
   * 具有访问权限的 principals 留空（点击 **Done**）。
3. 创建并下载 **JSON Key**：
   * 在服务账号列表中，点击刚刚创建的服务账号。
   * 切换到 **Keys** 标签页。
   * 点击 **Add Key** &gt; **Create new key**。
   * 选择 **JSON** 并点击 **Create**。
4. 将下载的 JSON 文件存放在你的 Gateway 主机上（例如：`~/.openclaw/googlechat-service-account.json`）。
5. 在 [Google Cloud Console Chat Configuration](https://console.cloud.google.com/apis/api/chat.googleapis.com/hangouts-chat) 中创建一个 Google Chat 应用：
   * 填写 **Application info**：
     * **App name**：例如 `OpenClaw`
     * **Avatar URL**：例如 `https://openclaw.ai/logo.png`
     * **Description**：例如 `Personal AI Assistant`
   * 启用 **Interactive features**。
   * 在 **Functionality** 下勾选 **Join spaces and group conversations**。
   * 在 **Connection settings** 下选择 **HTTP endpoint URL**。
   * 在 **Triggers** 下选择 **Use a common HTTP endpoint URL for all triggers**，并将其设置为你的 Gateway 公网可访问的 URL，后面加上 `/googlechat`。
     * *提示：运行 `openclaw status` 以查看你的 Gateway 公网 URL。*
   * 在 **Visibility** 下勾选 **Make this Chat app available to specific people and groups in &lt;Your Domain&gt;**。
   * 在文本框中输入你的邮箱地址（例如 `user@example.com`）。
   * 点击底部的 **Save**。
6. **启用应用状态**：
   * 保存后，**刷新页面**。
   * 找到 **App status** 区域（通常位于保存后页面的顶部或底部附近）。
   * 将状态更改为 **Live - available to users**。
   * 再次点击 **Save**。
7. 使用服务账号路径和 webhook audience 配置 OpenClaw：
   * 环境变量：`GOOGLE_CHAT_SERVICE_ACCOUNT_FILE=/path/to/service-account.json`
   * 或在配置中：`channels.googlechat.serviceAccountFile: "/path/to/service-account.json"`。
8. 设置 webhook audience 的类型和取值（需与 Chat 应用配置保持一致）。
9. 启动 Gateway。Google Chat 会向你的 webhook 路径发送 POST 请求。

<div id="add-to-google-chat">
  ## 添加到 Google Chat
</div>

当 Gateway 已运行且你的邮箱已被加入可见列表后：

1. 前往 [Google Chat](https://chat.google.com/)。
2. 点击 **Direct Messages** 旁边的 **+**（加号）图标。
3. 在搜索栏（你平时用来添加联系人的位置）中，输入你在 Google Cloud Console 中配置的 **App name**。
   * **注意**：该机器人*不会*出现在 “Marketplace” 浏览列表中，因为它是一个私有应用。你必须通过名称来搜索它。
4. 在搜索结果中选择你的机器人。
5. 点击 **Add** 或 **Chat** 以开始 1:1 对话。
6. 发送 “Hello” 以触发助理！

<div id="public-url-webhook-only">
  ## 公共 URL（仅用于 Webhook）
</div>

Google Chat webhook 需要一个公共 HTTPS 端点。出于安全考虑，**只将 `/googlechat` 路径暴露到互联网**。请将 OpenClaw 控制面板和其他敏感端点保留在你的私有网络中。

<div id="option-a-tailscale-funnel-recommended">
  ### 选项 A：Tailscale Funnel（推荐）
</div>

对私有仪表盘使用 Tailscale Serve，对公共 webhook 路径使用 Funnel。这样可以保持 `/` 私有，仅暴露 `/googlechat`。

1. **检查你的 Gateway 绑定到哪个地址：**
   ```bash
   ss -tlnp | grep 18789
   ```
   记下 IP 地址（例如 `127.0.0.1`、`0.0.0.0`，或者你的 Tailscale IP，如 `100.x.x.x`）。

2. **仅向 tailnet 暴露仪表盘（端口 8443）：**
   ```bash
   # 如果绑定到 localhost（127.0.0.1 或 0.0.0.0）：
   tailscale serve --bg --https 8443 http://127.0.0.1:18789

   # 如果仅绑定到 Tailscale IP（例如 100.106.161.80）：
   tailscale serve --bg --https 8443 http://100.106.161.80:18789
   ```

3. **仅将 webhook 路径公开到公网：**
   ```bash
   # 如果绑定到 localhost（127.0.0.1 或 0.0.0.0）：
   tailscale funnel --bg --set-path /googlechat http://127.0.0.1:18789/googlechat

   # 如果仅绑定到 Tailscale IP（例如 100.106.161.80）：
   tailscale funnel --bg --set-path /googlechat http://100.106.161.80:18789/googlechat
   ```

4. **为 Funnel 访问授权此节点：**
   如果出现提示，请访问输出中显示的授权 URL，在你的 tailnet 策略中为此节点启用 Funnel。

5. **验证配置：**
   ```bash
   tailscale serve status
   tailscale funnel status
   ```

你的公共 webhook URL 将是：
`https://<node-name>.<tailnet>.ts.net/googlechat`

你的私有仪表盘将保持仅限 tailnet 访问：
`https://<node-name>.<tailnet>.ts.net:8443/`

在 Google Chat 应用配置中使用公共 URL（不带 `:8443`）。

> 注意：此配置在重启后仍然有效。若要之后移除它，请运行 `tailscale funnel reset` 和 `tailscale serve reset`。

<div id="option-b-reverse-proxy-caddy">
  ### 方案 B：反向代理（Caddy）
</div>

如果你使用 Caddy 等反向代理，请仅代理该特定路径：

```caddy
your-domain.com {
    reverse_proxy /googlechat* localhost:18789
}
```

通过此配置，对 `your-domain.com/` 的任何请求都会被忽略或返回 404，而 `your-domain.com/googlechat` 将被安全地路由到 OpenClaw。

<div id="option-c-cloudflare-tunnel">
  ### 选项 C：Cloudflare Tunnel
</div>

将你的隧道 ingress 规则配置为只路由 webhook 路径：

* **路径**：`/googlechat` -&gt; `http://localhost:18789/googlechat`
* **默认规则**：HTTP 404（未找到）

<div id="how-it-works">
  ## 工作原理
</div>

1. Google Chat 向 Gateway 发送 webhook POST 请求。每个请求都包含 `Authorization: Bearer <token>` 请求头。
2. OpenClaw 根据配置的 `audienceType` + `audience` 验证该 token：
   * `audienceType: "app-url"` → audience 是你的 HTTPS webhook URL。
   * `audienceType: "project-number"` → audience 是 Cloud 项目编号。
3. 消息按 space（会话空间）进行路由：
   * 私信（DM）使用会话键 `agent:<agentId>:googlechat:dm:<spaceId>`。
   * 群组 space 使用会话键 `agent:<agentId>:googlechat:group:<spaceId>`。
4. 私信访问默认通过配对控制。未知发送者会收到配对码；使用以下命令批准：
   * `openclaw pairing approve googlechat <code>`
5. 群组 space 默认需要 @ 提及。若提及检测需要使用应用的用户名，请配置 `botUser`。

<div id="targets">
  ## 目标
</div>

在消息发送和允许列表中使用以下标识符：

* 私聊：`users/<userId>` 或 `users/<email>`（支持电子邮件地址）。
* 空间：`spaces/<spaceId>`。

<div id="config-highlights">
  ## 关键配置
</div>

```json5
{
  channels: {
    "googlechat": {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      audienceType: "app-url",
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890", // 可选;辅助提及检测
      dm: {
        policy: "pairing",
        allowFrom: ["users/1234567890", "name@example.com"]
      },
      groupPolicy: "allowlist",
      groups: {
        "spaces/AAAA": {
          allow: true,
          requireMention: true,
          users: ["users/1234567890"],
          systemPrompt: "Short answers only."
        }
      },
      actions: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20
    }
  }
}
```

Notes:

* 服务账号凭据也可以通过 `serviceAccount`（JSON 字符串）以内联方式传入。
* 如果未设置 `webhookPath`，默认的 webhook 路径为 `/googlechat`。
* 当启用 `actions.reactions` 时，可以通过 `reactions` 工具和 `channels action` 使用表情回应（reactions）。
* `typingIndicator` 支持 `none`、`message`（默认）和 `reaction`（使用 reaction 需要用户 OAuth）。
* 附件通过 Chat API 下载，并存储在媒体管道中（大小受 `mediaMaxMb` 限制）。

<div id="troubleshooting">
  ## 故障排除
</div>

<div id="405-method-not-allowed">
  ### 405 方法不允许
</div>

如果 Google Cloud Logs Explorer 显示类似如下错误：

```
status code: 405, reason phrase: HTTP error response: HTTP/1.1 405 Method Not Allowed
```

这意味着 webhook 处理程序尚未注册。常见原因如下：

1. **通道未配置**：你的配置中缺少 `channels.googlechat` 部分。使用以下命令确认：
   ```bash
   openclaw config get channels.googlechat
   ```
   如果返回 “Config path not found”，请添加相应配置（参见 [配置要点](#config-highlights)）。

2. **插件未启用**：检查插件状态：
   ```bash
   openclaw plugins list | grep googlechat
   ```
   如果显示 “disabled”，请在配置中添加 `plugins.entries.googlechat.enabled: true`。

3. **Gateway 未重启**：添加配置后，重启 Gateway：
   ```bash
   openclaw gateway restart
   ```

验证通道是否正在运行：

```bash
openclaw channels status
# 应显示:Google Chat default: enabled, configured, ...
```

<div id="other-issues">
  ### 其他问题
</div>

* 运行 `openclaw channels status --probe` 检查是否存在身份验证错误或遗漏的 audience 配置。
* 如果没有收到消息，确认 Chat 应用的 webhook URL 和事件订阅是否正确。
* 如果因为提及门控导致回复被拦截，将 `botUser` 设置为应用的用户资源名称，并核实 `requireMention` 的配置。
* 在发送测试消息时使用 `openclaw logs --follow`，查看请求是否到达 Gateway。

相关文档：

* [Gateway 配置](/zh/gateway/configuration)
* [安全](/zh/gateway/security)
* [Reactions](/zh/tools/reactions)