---
title: Hetzner
summary: "在廉价的 Hetzner VPS（Docker）上 24/7 运行 OpenClaw Gateway，具备持久化状态和内置二进制文件"
read_when:
  - 你希望在云端 VPS（而不是在自己的笔记本上）24/7 运行 OpenClaw
  - 你希望在自己的 VPS 上拥有一套生产级、始终在线的 Gateway
  - 你希望完全控制持久化状态、二进制文件以及重启行为
  - 你在 Hetzner 或类似提供方上使用 Docker 运行 OpenClaw
---

<div id="openclaw-on-hetzner-docker-production-vps-guide">
  # 使用 Docker 在 Hetzner 上部署 OpenClaw（生产环境 VPS 指南）
</div>

<div id="goal">
  ## 目标
</div>

在 Hetzner VPS 上使用 Docker 运行一个持久运行的 OpenClaw Gateway，具备可持久化状态、内置二进制文件，以及安全的重启行为。

如果你想要“OpenClaw 24/7，费用大约 5 美元左右”，这是最简单且可靠的方案。
Hetzner 的价格可能会变动；先选择最小规格的 Debian/Ubuntu VPS，如果遇到内存耗尽（OOM）再向上扩容。

<div id="what-are-we-doing-simple-terms">
  ## 我们要做什么（通俗说）？
</div>

* 租一台小型 Linux 服务器（Hetzner VPS）
* 安装 Docker（隔离的应用运行环境）
* 在 Docker 中运行 OpenClaw Gateway
* 在宿主机上持久化存储 `~/.openclaw` 和 `~/.openclaw/workspace`（重启/重建后仍然保留）
* 通过 SSH 隧道从你的笔记本访问 Control UI

你可以通过以下方式访问 Gateway：

* 从你的笔记本做 SSH 端口转发
* 直接暴露端口（前提是你自行管理好防火墙和令牌）

本指南假设你在 Hetzner 上使用的是 Ubuntu 或 Debian。\
如果你使用的是其他 Linux VPS，请按需对应替换相关软件包。
关于通用的 Docker 流程，请参见 [Docker](/zh/install/docker)。

***

<div id="quick-path-experienced-operators">
  ## 快速流程（有经验的运维人员）
</div>

1. 申请 Hetzner VPS
2. 安装 Docker
3. 克隆 OpenClaw 仓库
4. 创建持久化宿主机目录
5. 配置 `.env` 和 `docker-compose.yml`
6. 将所需二进制文件预置到镜像中
7. 运行 `docker compose up -d`
8. 验证数据持久化和 Gateway 访问

***

<div id="what-you-need">
  ## 你需要准备什么
</div>

* 具有 root 权限的 Hetzner VPS
* 能从你的笔记本电脑通过 SSH 访问服务器
* 对 SSH 和复制/粘贴操作具备基本使用经验
* 约 20 分钟时间
* Docker 和 Docker Compose
* 模型授权凭据
* 可选的提供方凭据
  * WhatsApp 登录二维码
  * Telegram 机器人 token
  * Gmail OAuth

***

<div id="1-provision-the-vps">
  ## 1) 准备 VPS
</div>

在 Hetzner 上创建一个 Ubuntu 或 Debian VPS。

以 root 用户身份连接：

```bash
ssh root@YOUR_VPS_IP
```

本指南假定该 VPS 是有状态的。
不要把它当作可随意丢弃的临时基础设施。

***

<div id="2-install-docker-on-the-vps">
  ## 2) 在 VPS 上安装 Docker
</div>

```bash
apt-get update
apt-get install -y git curl ca-certificates
curl -fsSL https://get.docker.com | sh
```

验证：

```bash
docker --version
docker compose version
```

***

<div id="3-clone-the-openclaw-repository">
  ## 3）克隆 OpenClaw 代码仓库
</div>

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

本指南假定你将构建自定义镜像，以确保二进制持久化。

***

<div id="4-create-persistent-host-directories">
  ## 4) 创建持久化的宿主机目录
</div>

Docker 容器是短暂的。
所有需要持久化的状态都必须保存在宿主机上。

```bash
mkdir -p /root/.openclaw
mkdir -p /root/.openclaw/workspace

# 将所有权设置为容器用户(uid 1000):
chown -R 1000:1000 /root/.openclaw
chown -R 1000:1000 /root/.openclaw/workspace
```

***

<div id="5-configure-environment-variables">
  ## 5）配置环境变量
</div>

在仓库根目录创建 `.env` 文件。

```bash
OPENCLAW_IMAGE=openclaw:latest
OPENCLAW_GATEWAY_TOKEN=change-me-now
OPENCLAW_GATEWAY_BIND=lan
OPENCLAW_GATEWAY_PORT=18789

OPENCLAW_CONFIG_DIR=/root/.openclaw
OPENCLAW_WORKSPACE_DIR=/root/.openclaw/workspace

GOG_KEYRING_PASSWORD=change-me-now
XDG_CONFIG_HOME=/home/node/.openclaw
```

生成强随机密钥：

```bash
openssl rand -hex 32
```

**不要将此文件提交到版本库。**

***

<div id="6-docker-compose-configuration">
  ## 6) Docker Compose 配置
</div>

创建或更新 `docker-compose.yml` 文件。

```yaml
services:
  openclaw-gateway:
    image: ${OPENCLAW_IMAGE}
    build: .
    restart: unless-stopped
    env_file:
      - .env
    environment:
      - HOME=/home/node
      - NODE_ENV=production
      - TERM=xterm-256color
      - OPENCLAW_GATEWAY_BIND=${OPENCLAW_GATEWAY_BIND}
      - OPENCLAW_GATEWAY_PORT=${OPENCLAW_GATEWAY_PORT}
      - OPENCLAW_GATEWAY_TOKEN=${OPENCLAW_GATEWAY_TOKEN}
      - GOG_KEYRING_PASSWORD=${GOG_KEYRING_PASSWORD}
      - XDG_CONFIG_HOME=${XDG_CONFIG_HOME}
      - PATH=/home/linuxbrew/.linuxbrew/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    volumes:
      - ${OPENCLAW_CONFIG_DIR}:/home/node/.openclaw
      - ${OPENCLAW_WORKSPACE_DIR}:/home/node/.openclaw/workspace
    ports:
      # Recommended: keep the Gateway loopback-only on the VPS; access via SSH tunnel.
      # To expose it publicly, remove the `127.0.0.1:` prefix and firewall accordingly.
      - "127.0.0.1:${OPENCLAW_GATEWAY_PORT}:18789"

      # 可选:仅当您在此 VPS 上运行 iOS/Android 节点且需要 Canvas 主机时启用。
      # 如需公开暴露,请阅读 /gateway/security 并相应配置防火墙。
      # - "18793:18793"
    command:
      [
        "node",
        "dist/index.js",
        "gateway",
        "--bind",
        "${OPENCLAW_GATEWAY_BIND}",
        "--port",
        "${OPENCLAW_GATEWAY_PORT}"
      ]
```

***

<div id="7-bake-required-binaries-into-the-image-critical">
  ## 7) 将所需二进制文件预置到镜像中（关键）
</div>

在运行中的容器内安装二进制文件是个陷阱。
任何在运行时安装的内容在重启后都会丢失。

所有技能所依赖的外部二进制文件都必须在镜像构建阶段安装。

下面的示例仅展示三种常见的二进制文件：

* 用于访问 Gmail 的 `gog`
* 用于 Google Places 的 `goplaces`
* 用于 WhatsApp 的 `wacli`

这些只是示例，而不是完整列表。
你可以按照相同方式安装任意数量的二进制文件。

如果你之后添加了依赖额外二进制文件的新技能，你必须：

1. 更新 Dockerfile
2. 重新构建镜像
3. 重启容器

**Dockerfile 示例**

```dockerfile
FROM node:22-bookworm

RUN apt-get update && apt-get install -y socat && rm -rf /var/lib/apt/lists/*

# 示例二进制文件 1：Gmail CLI
RUN curl -L https://github.com/steipete/gog/releases/latest/download/gog_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/gog

# 示例二进制文件 2：Google Places CLI
RUN curl -L https://github.com/steipete/goplaces/releases/latest/download/goplaces_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/goplaces

# 示例二进制文件 3：WhatsApp CLI
RUN curl -L https://github.com/steipete/wacli/releases/latest/download/wacli_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/wacli

# 按照相同模式在下方添加更多二进制文件

WORKDIR /app
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml .npmrc ./
COPY ui/package.json ./ui/package.json
COPY scripts ./scripts

RUN corepack enable
RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build
RUN pnpm ui:install
RUN pnpm ui:build

ENV NODE_ENV=production

CMD ["node","dist/index.js"]
```

***

<div id="8-build-and-launch">
  ## 8) 构建并启动
</div>

```bash
docker compose build
docker compose up -d openclaw-gateway
```

校验二进制文件：

```bash
docker compose exec openclaw-gateway which gog
docker compose exec openclaw-gateway which goplaces
docker compose exec openclaw-gateway which wacli
```

预期输出：

```
/usr/local/bin/gog
/usr/local/bin/goplaces
/usr/local/bin/wacli
```

***

<div id="9-verify-gateway">
  ## 9）验证 Gateway
</div>

```bash
docker compose logs -f openclaw-gateway
```

成功：

```
[gateway] listening on ws://0.0.0.0:18789
```

在你的本地电脑上：

```bash
ssh -N -L 18789:127.0.0.1:18789 root@YOUR_VPS_IP
```

在浏览器中打开：

`http://127.0.0.1:18789/`

粘贴你的 Gateway 令牌。

***

<div id="what-persists-where-source-of-truth">
  ## 哪些内容持久化在哪里（权威数据源）
</div>

OpenClaw 运行在 Docker 中，但 Docker 本身不是权威数据源（source of truth）。
所有需要长期保存的状态都必须在服务重启、镜像重建和系统重启后依然保留。

| 组件 | 位置 | 持久化机制 | 说明 |
|---|---|---|---|
| Gateway 配置 | `/home/node/.openclaw/` | 挂载宿主机卷 | 包含 `openclaw.json`、令牌（tokens） |
| 模型认证配置 | `/home/node/.openclaw/` | 挂载宿主机卷 | OAuth 令牌、API key |
| 技能配置 | `/home/node/.openclaw/skills/` | 挂载宿主机卷 | 技能级状态 |
| Agent 工作区 | `/home/node/.openclaw/workspace/` | 挂载宿主机卷 | 代码和 Agent 工件 |
| WhatsApp 会话 | `/home/node/.openclaw/` | 挂载宿主机卷 | 保留二维码登录状态 |
| Gmail 密钥环 | `/home/node/.openclaw/` | 宿主机卷 + 密码 | 需要 `GOG_KEYRING_PASSWORD` |
| 外部二进制文件 | `/usr/local/bin/` | Docker 镜像 | 必须在构建阶段打包进镜像 |
| Node 运行时环境 | 容器文件系统 | Docker 镜像 | 每次构建镜像都会重建 |
| 操作系统软件包 | 容器文件系统 | Docker 镜像 | 不要在运行时安装 |
| Docker 容器 | 临时 | 可重启 | 可安全销毁 |