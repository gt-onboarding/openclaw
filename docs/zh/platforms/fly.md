---
title: Fly.io
description: 将 OpenClaw 部署到 Fly.io
---

<div id="flyio-deployment">
  # Fly.io 部署
</div>

**目标：**在具有持久存储、自动 HTTPS 和 Discord/频道访问的 [Fly.io](https://fly.io) 机器上运行 OpenClaw Gateway。

<div id="what-you-need">
  ## 你需要准备什么
</div>

- 已安装的 [flyctl CLI](https://fly.io/docs/hands-on/install-flyctl/)
- Fly.io 账号（免费套餐即可）
- 模型认证信息：Anthropic API 密钥（或其他提供方密钥）
- 渠道凭据：Discord 机器人令牌、Telegram 令牌等

<div id="beginner-quick-path">
  ## 新手快速上手
</div>

1. 克隆仓库 → 修改 `fly.toml`
2. 创建应用和卷 → 设置 Secrets
3. 使用 `fly deploy` 部署
4. 通过 SSH 登录后创建配置，或使用 Control UI

<div id="1-create-the-fly-app">
  ## 1) 创建 Fly 应用程序
</div>

```bash
# 克隆代码仓库
git clone https://github.com/openclaw/openclaw.git
cd openclaw

# 创建新的 Fly 应用(选择你自己的名称)
fly apps create my-openclaw

# 创建持久化卷(通常 1GB 就足够了)
fly volumes create openclaw_data --size 1 --region iad
```

**提示：** 选择一个靠近你所在位置的区域。常见选项：`lhr`（伦敦）、`iad`（弗吉尼亚）、`sjc`（圣何塞）。


<div id="2-configure-flytoml">
  ## 2) 配置 fly.toml
</div>

编辑 `fly.toml` 以匹配你的应用名称和需求。

**安全提示：** 默认配置会公开一个公网 URL。要进行无公共 IP 的加固部署，请参见 [私有部署](#private-deployment-hardened) 或使用 `fly.private.toml`。

```toml
app = "my-openclaw"  # 你的应用名称
primary_region = "iad"

[build]
  dockerfile = "Dockerfile"

[env]
  NODE_ENV = "production"
  OPENCLAW_PREFER_PNPM = "1"
  OPENCLAW_STATE_DIR = "/data"
  NODE_OPTIONS = "--max-old-space-size=1536"

[processes]
  app = "node dist/index.js gateway --allow-unconfigured --port 3000 --bind lan"

[http_service]
  internal_port = 3000
  force_https = true
  auto_stop_machines = false
  auto_start_machines = true
  min_machines_running = 1
  processes = ["app"]

[[vm]]
  size = "shared-cpu-2x"
  memory = "2048mb"

[mounts]
  source = "openclaw_data"
  destination = "/data"
```

**关键设置：**

| Setting                        | Why                                                          |
| ------------------------------ | ------------------------------------------------------------ |
| `--bind lan`                   | 绑定到 `0.0.0.0`，以便 Fly 的代理可以访问 Gateway                         |
| `--allow-unconfigured`         | 允许在没有配置文件的情况下启动（你之后再创建配置文件）                                  |
| `internal_port = 3000`         | 必须与 `--port 3000`（或 `OPENCLAW_GATEWAY_PORT`）一致，供 Fly 的健康检查使用 |
| `memory = "2048mb"`            | 512MB 太小，推荐设置为 2GB                                           |
| `OPENCLAW_STATE_DIR = "/data"` | 将状态持久化存储在该数据卷上                                               |


<div id="3-set-secrets">
  ## 3) 设置机密信息
</div>

```bash
# 必需:Gateway 令牌(用于非回环绑定)
fly secrets set OPENCLAW_GATEWAY_TOKEN=$(openssl rand -hex 32)

# Model provider API keys
fly secrets set ANTHROPIC_API_KEY=sk-ant-...

# Optional: Other providers
fly secrets set OPENAI_API_KEY=sk-...
fly secrets set GOOGLE_API_KEY=...

# Channel tokens
fly secrets set DISCORD_BOT_TOKEN=MTQ...
```

**注意：**

* 非回环地址绑定（`--bind lan`）出于安全考虑需要提供 `OPENCLAW_GATEWAY_TOKEN`。
* 把这些 token 当成密码对待。
* 对所有 API 密钥和 token，**优先使用环境变量而不是配置文件**。这样可以避免将机密信息放进 `openclaw.json`，以免被意外暴露或记录到日志中。


<div id="4-deploy">
  ## 4）部署
</div>

```bash
fly deploy
```

首次部署时会构建 Docker 镜像（大约 2–3 分钟）。后续部署会更快。

部署完成后，请检查：

```bash
fly status
fly logs
```

你应该会看到：

```
[gateway] listening on ws://0.0.0.0:3000 (PID xxx)
[discord] logged in to discord as xxx
```


<div id="5-create-config-file">
  ## 5) 创建配置文件
</div>

通过 SSH 登录到该机器，创建配置文件：

```bash
fly ssh console
```

创建配置目录和文件：

```bash
mkdir -p /data
cat > /data/openclaw.json << 'EOF'
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-opus-4-5",
        "fallbacks": ["anthropic/claude-sonnet-4-5", "openai/gpt-4o"]
      },
      "maxConcurrent": 4
    },
    "list": [
      {
        "id": "main",
        "default": true
      }
    ]
  },
  "auth": {
    "profiles": {
      "anthropic:default": { "mode": "token", "provider": "anthropic" },
      "openai:default": { "mode": "token", "provider": "openai" }
    }
  },
  "bindings": [
    {
      "agentId": "main",
      "match": { "channel": "discord" }
    }
  ],
  "channels": {
    "discord": {
      "enabled": true,
      "groupPolicy": "allowlist",
      "guilds": {
        "YOUR_GUILD_ID": {
          "channels": { "general": { "allow": true } },
          "requireMention": false
        }
      }
    }
  },
  "gateway": {
    "mode": "local",
    "bind": "auto"
  },
  "meta": {
    "lastTouchedVersion": "2026.1.29"
  }
}
EOF
```

**注意：** 当 `OPENCLAW_STATE_DIR=/data` 时，配置文件路径为 `/data/openclaw.json`。

**注意：** Discord token 可以来自以下任一来源：

* 环境变量：`DISCORD_BOT_TOKEN`（推荐用于存放机密信息）
* 配置文件：`channels.discord.token`

如果使用环境变量，则无需在配置中添加 token。Gateway 会自动读取 `DISCORD_BOT_TOKEN`。

重启以生效：

```bash
exit
fly machine restart <machine-id>
```


<div id="6-access-the-gateway">
  ## 6）访问 Gateway
</div>

<div id="control-ui">
  ### Control UI
</div>

在浏览器中打开：

```bash
fly open
```

或者访问 `https://my-openclaw.fly.dev/`

将你的 Gateway 令牌（即 `OPENCLAW_GATEWAY_TOKEN` 中的值）粘贴进去以完成身份验证。


<div id="logs">
  ### 日志
</div>

```bash
fly logs              # 实时日志
fly logs --no-tail    # 近期日志
```


<div id="ssh-console">
  ### SSH 控制台
</div>

```bash
fly ssh console
```


<div id="troubleshooting">
  ## 故障排查
</div>

<div id="app-is-not-listening-on-expected-address">
  ### “应用未在预期地址监听”
</div>

Gateway 正在绑定到 `127.0.0.1`，而不是 `0.0.0.0`。

**解决方法：** 在 `fly.toml` 中的进程启动命令中添加 `--bind lan`。

<div id="health-checks-failing-connection-refused">
  ### 健康检查失败 / 连接被拒绝
</div>

Fly 无法通过配置的端口连接到 Gateway。

**解决方法：** 确保 `internal_port` 与 Gateway 实际端口一致（设置 `--port 3000` 或 `OPENCLAW_GATEWAY_PORT=3000`）。

<div id="oom-memory-issues">
  ### OOM / 内存不足问题
</div>

容器反复重启或被终止。迹象包括：`SIGABRT`、`v8::internal::Runtime_AllocateInYoungGeneration`，或静默重启。

**解决方法：** 在 `fly.toml` 中增加内存：

```toml
[[vm]]
  memory = "2048mb"
```

或者更新一台现有的机器：

```bash
fly machine update <machine-id> --vm-memory 2048 -y
```

**注意：**512MB 内存太小。1GB 可能勉强可用，但在高负载或启用详细日志时可能会触发内存不足（OOM）。**推荐使用 2GB。**


<div id="gateway-lock-issues">
  ### Gateway 锁文件问题
</div>

Gateway 无法启动，并报 &quot;already running&quot; 错误。

当容器重启但 PID 锁文件仍残留在卷上时，就会出现这种情况。

**解决方法：** 删除该锁文件：

```bash
fly ssh console --command "rm -f /data/gateway.*.lock"
fly machine restart <machine-id>
```

锁文件位于 `/data/gateway.*.lock`（不在任何子目录下）。


<div id="config-not-being-read">
  ### 配置未被加载
</div>

如果使用 `--allow-unconfigured`，Gateway 会创建一个最小配置。重启时应加载你在 `/data/openclaw.json` 中的自定义配置。

确认该配置文件已存在：

```bash
fly ssh console --command "cat /data/openclaw.json"
```


<div id="writing-config-via-ssh">
  ### 通过 SSH 写配置
</div>

`fly ssh console -C` 命令不支持使用 shell 重定向。若要写配置文件：

```bash
# Use echo + tee (pipe from local to remote)
echo '{"your":"config"}' | fly ssh console -C "tee /data/openclaw.json"

# Or use sftp
fly sftp shell
> put /local/path/config.json /data/openclaw.json
```

**注意：** 如果文件已存在，`fly sftp` 可能会失败。请先将其删除：

```bash
fly ssh console --command "rm /data/openclaw.json"
```


<div id="state-not-persisting">
  ### 状态未持久化
</div>

如果在重启后丢失凭据或会话，说明状态目录被写到了容器文件系统中。

**修复方法：** 确保在 `fly.toml` 中设置 `OPENCLAW_STATE_DIR=/data`，然后重新部署。

<div id="updates">
  ## 更新
</div>

```bash
# 拉取最新变更
git pull

# 重新部署
fly deploy

# 检查运行状态
fly status
fly logs
```


<div id="updating-machine-command">
  ### 更新 Machine 启动命令
</div>

如果你需要在不进行完整重新部署的情况下修改启动命令：

```bash
# 获取机器 ID
fly machines list

# 更新命令
fly machine update <machine-id> --command "node dist/index.js gateway --port 3000 --bind lan" -y

# 或增加内存
fly machine update <machine-id> --vm-memory 2048 --command "node dist/index.js gateway --port 3000 --bind lan" -y
```

**注意：** 执行 `fly deploy` 之后，`machine` 的 command 配置可能会被重置为 `fly.toml` 中的值。如果你做过手动修改，请在部署后重新应用这些更改。


<div id="private-deployment-hardened">
  ## 私有部署（加固）
</div>

默认情况下，Fly 会分配公网 IP 地址，使你的 Gateway 可以通过 `https://your-app.fly.dev` 访问。这样虽然方便，但也意味着你的部署可能会被互联网扫描服务（如 Shodan、Censys 等）发现。

如果需要**完全不对公网暴露**的加固部署，请使用私有模板。

<div id="when-to-use-private-deployment">
  ### 何时使用私有部署
</div>

- 你只发起**出站**调用/消息（不接收入站 webhook 请求）
- 你通过 **ngrok 或 Tailscale** 隧道处理任何 webhook 回调
- 你通过 **SSH、代理或 WireGuard** 访问 Gateway，而不是通过浏览器
- 你希望部署对**互联网扫描工具/扫描服务保持隐藏**

<div id="setup">
  ### 配置
</div>

使用 `fly.private.toml`，而不是标准配置：

```bash
# 使用私有配置部署
fly deploy -c fly.private.toml
```

或者转换现有的部署：

```bash
# List current IPs
fly ips list -a my-openclaw

# Release public IPs
fly ips release <public-ipv4> -a my-openclaw
fly ips release <public-ipv6> -a my-openclaw

# 切换到私有配置,使后续部署不再重新分配公网 IP
# (移除 [http_service] 或使用私有模板部署)
fly deploy -c fly.private.toml

# Allocate private-only IPv6
fly ips allocate-v6 --private -a my-openclaw
```

完成上述步骤后，`fly ips list` 应该只会显示一个 `private` 类型的 IP 地址：

```
VERSION  IP                   TYPE             REGION
v6       fdaa:x:x:x:x::x      private          global
```


<div id="accessing-a-private-deployment">
  ### 访问私有部署
</div>

由于没有公开 URL，请选择以下任一方法：

**方案 1：本地代理（最简单）**

```bash
# 将本地端口 3000 转发到应用程序
fly proxy 3000:3000 -a my-openclaw

# 然后在浏览器中打开 http://localhost:3000
```

**方案二：WireGuard VPN**

```bash
# 创建 WireGuard 配置（一次性）
fly wireguard create

# 导入到 WireGuard 客户端，然后通过内部 IPv6 访问
# 示例：http://[fdaa:x:x:x:x::x]:3000
```

**选项 3：仅使用 SSH**

```bash
fly ssh console -a my-openclaw
```


<div id="webhooks-with-private-deployment">
  ### 私有部署场景下的 Webhook
</div>

如果你需要在无需对外暴露服务的情况下处理 webhook 回调（Twilio、Telnyx 等）：

1. **ngrok 隧道** - 在容器内部或以 sidecar 方式运行 ngrok
2. **Tailscale Funnel** - 通过 Tailscale 暴露特定路径
3. **仅出站（Outbound-only）** - 某些提供方（如 Twilio）在不使用 webhook 的情况下，仅进行出站呼叫也可以正常工作

使用 ngrok 的语音呼叫配置示例：

```json
{
  "plugins": {
    "entries": {
      "voice-call": {
        "enabled": true,
        "config": {
          "provider": "twilio",
          "tunnel": { "provider": "ngrok" }
        }
      }
    }
  }
}
```

ngrok 隧道在容器内运行，在不直接暴露 Fly 应用本身的情况下提供一个公开的 webhook URL。


<div id="security-benefits">
  ### 安全优势
</div>

| 方面 | 公共部署 | 私有部署 |
|--------|--------|---------|
| 互联网扫描器 | 易被发现 | 对外隐藏 |
| 直接攻击 | 可能发生 | 被阻断 |
| Control UI 访问 | 直接通过浏览器 | 需通过代理/VPN |
| Webhook 投递 | 直接连接 | 通过隧道转发 |

<div id="notes">
  ## 注意事项
</div>

- Fly.io 使用的是 **x86 架构**（非 ARM）
- 该 Dockerfile 同时兼容这两种架构
- 对于 WhatsApp/Telegram 的接入/初始配置，请使用 `fly ssh console`
- 持久化数据位于挂载在 `/data` 的卷中
- Signal 需要 Java 和 signal-cli；请使用自定义镜像，并将内存设置为至少 2GB。

<div id="cost">
  ## 成本
</div>

使用推荐配置（`shared-cpu-2x`，2GB RAM）：

- 每月约 $10–15，具体取决于实际使用情况
- 免费层包含一定的免费额度

详情请参见 [Fly.io 定价](https://fly.io/docs/about/pricing/)。