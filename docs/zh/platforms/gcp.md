---
title: Gcp
summary: "在 GCP Compute Engine 虚拟机（Docker）上 24/7 运行 OpenClaw Gateway，并保持持久化状态"
read_when:
  - 你想在 GCP 上让 OpenClaw 24/7 持续运行
  - 你想在自己的虚拟机上部署一套生产级、始终在线的 Gateway
  - 你希望对持久化、二进制以及重启行为拥有完全控制权
---

<div id="openclaw-on-gcp-compute-engine-docker-production-vps-guide">
  # 在 GCP Compute Engine 上运行 OpenClaw（Docker 生产环境 VPS 指南）
</div>

<div id="goal">
  ## 目标
</div>

在 GCP Compute Engine 虚拟机上使用 Docker 运行一个长期运行的 OpenClaw Gateway，具备可持久化状态、预置的二进制文件，以及安全的重启行为。

如果你想要“每月约 5–12 美元、24/7 运行的 OpenClaw”，这是在 Google Cloud 上相当可靠的一种部署方案。
价格会随机器类型和地域而变化；选择能满足你当前工作负载的最小虚拟机，如果遇到内存不足（OOM）再向上扩容。

<div id="what-are-we-doing-simple-terms">
  ## 我们要做什么（通俗版）？
</div>

* 创建一个 GCP 项目并启用计费
* 创建一个 Compute Engine 虚拟机
* 安装 Docker（应用隔离的运行时环境）
* 在 Docker 中启动 OpenClaw Gateway
* 在宿主机上持久化 `~/.openclaw` + `~/.openclaw/workspace`（在重启/重建后仍然保留）
* 通过 SSH 隧道从你的笔记本访问 Control UI

Gateway 可以通过以下方式访问：

* 从你的笔记本进行 SSH 端口转发
* 直接暴露端口（前提是你自行管理防火墙和令牌）

本指南使用 GCP Compute Engine 上的 Debian。
Ubuntu 也同样适用；请相应调整对应的软件包。
通用 Docker 流程参见 [Docker](/zh/install/docker)。

***

<div id="quick-path-experienced-operators">
  ## 快速路径（适用于有经验的运维人员）
</div>

1. 创建 GCP 项目并启用 Compute Engine API
2. 创建 Compute Engine VM（e2-small，Debian 12，20GB）
3. 通过 SSH 登录到该 VM
4. 安装 Docker
5. 克隆 OpenClaw 仓库
6. 创建持久化宿主目录
7. 配置 `.env` 和 `docker-compose.yml`
8. 制作所需二进制文件，构建并启动

***

<div id="what-you-need">
  ## 你需要准备什么
</div>

* GCP 账号（符合免费套餐 e2-micro 资格）
* 已安装 gcloud CLI（或使用 Cloud Console）
* 能从本地电脑通过 SSH 访问实例
* 对使用 SSH 和复制/粘贴操作有基本掌握
* 约 20–30 分钟时间
* Docker 和 Docker Compose
* 模型认证凭据
* 可选的提供方凭据
  * WhatsApp 二维码
  * Telegram 机器人 token
  * Gmail OAuth

***

<div id="1-install-gcloud-cli-or-use-console">
  ## 1) 安装 gcloud CLI（或使用控制台）
</div>

**选项 A：gcloud CLI**（推荐用于自动化）

从 https://cloud.google.com/sdk/docs/install 安装

初始化并完成身份验证：

```bash
gcloud init
gcloud auth login
```

**选项 B：Cloud Console**

所有步骤都可以通过位于 https://console.cloud.google.com 的 Web UI 完成

***

<div id="2-create-a-gcp-project">
  ## 2) 创建 GCP 项目
</div>

**CLI：**

```bash
gcloud projects create my-openclaw-project --name="OpenClaw Gateway"
gcloud config set project my-openclaw-project
```

在 https://console.cloud.google.com/billing 启用结算功能（Compute Engine 必需）。

启用 Compute Engine API：

```bash
gcloud services enable compute.googleapis.com
```

**控制台：**

1. 进入 IAM &amp; Admin &gt; Create Project
2. 为项目命名并创建
3. 为该项目启用计费
4. 前往 APIs &amp; Services &gt; Enable APIs &gt; 搜索 &quot;Compute Engine API&quot; &gt; Enable

***

<div id="3-create-the-vm">
  ## 3) 创建 VM
</div>

**机器类型：**

| 类型       | 规格                 | 费用      | 备注                  |
| -------- | ------------------ | ------- | ------------------- |
| e2-small | 2 vCPU，2GB RAM     | 约 $12/月 | 推荐                  |
| e2-micro | 2 vCPU（共享），1GB RAM | 符合免费层资格 | 在高负载下可能发生 OOM（内存耗尽） |

**CLI：**

```bash
gcloud compute instances create openclaw-gateway \
  --zone=us-central1-a \
  --machine-type=e2-small \
  --boot-disk-size=20GB \
  --image-family=debian-12 \
  --image-project=debian-cloud
```

**控制台：**

1. 前往 Compute Engine &gt; VM instances &gt; Create instance
2. 名称（Name）：`openclaw-gateway`
3. 区域（Region）：`us-central1`，可用区（Zone）：`us-central1-a`
4. 机器类型（Machine type）：`e2-small`
5. 启动磁盘（Boot disk）：Debian 12，20GB
6. 点击 Create（创建）

***

<div id="4-ssh-into-the-vm">
  ## 4) 通过 SSH 登录到 VM
</div>

**CLI：**

```bash
gcloud compute ssh openclaw-gateway --zone=us-central1-a
```

**控制台：**

在 Compute Engine 控制台中，点击你的 VM 旁边的“SSH”按钮。

注意：VM 创建后，SSH 密钥的生效可能需要 1–2 分钟。如果连接被拒绝，请稍等片刻后重试。

***

<div id="5-install-docker-on-the-vm">
  ## 5) 安装 Docker（在虚拟机上）
</div>

```bash
sudo apt-get update
sudo apt-get install -y git curl ca-certificates
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker $USER
```

注销并重新登录以使组更改生效：

```bash
exit
```

然后再次通过 SSH 登录：

```bash
gcloud compute ssh openclaw-gateway --zone=us-central1-a
```

验证：

```bash
docker --version
docker compose version
```

***

<div id="6-clone-the-openclaw-repository">
  ## 6）克隆 OpenClaw 代码仓库
</div>

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

本指南假定你会构建自定义镜像，以保证二进制持久化。

***

<div id="7-create-persistent-host-directories">
  ## 7) 创建持久化的主机目录
</div>

Docker 容器是临时性的。
所有需要长期保留的状态都必须存放在主机上。

```bash
mkdir -p ~/.openclaw
mkdir -p ~/.openclaw/workspace
```

***

<div id="8-configure-environment-variables">
  ## 8) 配置环境变量
</div>

在仓库根目录下创建一个 `.env` 文件。

```bash
OPENCLAW_IMAGE=openclaw:latest
OPENCLAW_GATEWAY_TOKEN=change-me-now
OPENCLAW_GATEWAY_BIND=lan
OPENCLAW_GATEWAY_PORT=18789

OPENCLAW_CONFIG_DIR=/home/$USER/.openclaw
OPENCLAW_WORKSPACE_DIR=/home/$USER/.openclaw/workspace

GOG_KEYRING_PASSWORD=change-me-now
XDG_CONFIG_HOME=/home/node/.openclaw
```

生成高强度机密字符串：

```bash
openssl rand -hex 32
```

**不要提交此文件。**

***

<div id="9-docker-compose-configuration">
  ## 9) Docker Compose 配置
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
      # Recommended: keep the Gateway loopback-only on the VM; access via SSH tunnel.
      # To expose it publicly, remove the `127.0.0.1:` prefix and firewall accordingly.
      - "127.0.0.1:${OPENCLAW_GATEWAY_PORT}:18789"

      # 可选:仅当您需要在此虚拟机上运行 iOS/Android 节点并需要 Canvas 主机时。
      # 如需公开暴露此端口,请阅读 /gateway/security 并相应配置防火墙。
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

<div id="10-bake-required-binaries-into-the-image-critical">
  ## 10) 将所需二进制程序烘焙进镜像（关键）
</div>

在运行中的容器内安装二进制程序是个陷阱。
任何在运行时安装的内容，在重启后都会丢失。

所有技能所需的外部二进制程序都必须在镜像构建时安装。

下面的示例只展示了三种常见的二进制程序：

* 用于访问 Gmail 的 `gog`
* 用于 Google Places 的 `goplaces`
* 用于 WhatsApp 的 `wacli`

这些只是示例，并不是完整列表。
你可以按照相同的模式安装任意数量的二进制程序。

如果你之后新增了依赖额外二进制程序的技能，你必须：

1. 更新 Dockerfile
2. 重新构建镜像
3. 重启容器

**示例 Dockerfile**

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

# 按相同模式在下方添加更多二进制文件

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

<div id="11-build-and-launch">
  ## 11) 构建并运行
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

<div id="12-verify-gateway">
  ## 12) 确认 Gateway
</div>

```bash
docker compose logs -f openclaw-gateway
```

成功：

```
[gateway] listening on ws://0.0.0.0:18789
```

***

<div id="13-access-from-your-laptop">
  ## 13) 从笔记本电脑访问
</div>

创建一个 SSH 隧道，将 Gateway 端口转发过来：

```bash
gcloud compute ssh openclaw-gateway --zone=us-central1-a -- -L 18789:127.0.0.1:18789
```

在浏览器中打开：

`http://127.0.0.1:18789/`

粘贴你的 Gateway 令牌。

***

<div id="what-persists-where-source-of-truth">
  ## 什么会持久化到哪里（权威数据源）
</div>

OpenClaw 运行在 Docker 中，但 Docker 本身不是权威数据源。
所有需要长期保留的状态都必须能在服务重启、镜像重建和主机重启后继续存在。

| 组件 | 位置 | 持久化机制 | 备注 |
|---|---|---|---|
| Gateway 配置 | `/home/node/.openclaw/` | 宿主机卷挂载 | 包含 `openclaw.json`、令牌 |
| 模型认证配置文件 | `/home/node/.openclaw/` | 宿主机卷挂载 | OAuth 令牌、API keys |
| 技能配置 | `/home/node/.openclaw/skills/` | 宿主机卷挂载 | 技能级别状态 |
| Agent 工作区 | `/home/node/.openclaw/workspace/` | 宿主机卷挂载 | 代码和 Agent 工件 |
| WhatsApp 会话 | `/home/node/.openclaw/` | 宿主机卷挂载 | 保留二维码登录状态 |
| Gmail 密钥环 | `/home/node/.openclaw/` | 宿主机卷 + 密码 | 需要 `GOG_KEYRING_PASSWORD` |
| 外部二进制文件 | `/usr/local/bin/` | Docker 镜像 | 必须在镜像构建时打包进入 |
| Node.js 运行时 | 容器文件系统 | Docker 镜像 | 每次构建镜像都会重建 |
| 操作系统软件包 | 容器文件系统 | Docker 镜像 | 不要在运行时安装 |
| Docker 容器 | 临时 | 可重启 | 可安全销毁 |

***

<div id="updates">
  ## 更新
</div>

要在此虚拟机上更新 OpenClaw：

```bash
cd ~/openclaw
git pull
docker compose build
docker compose up -d
```

***

<div id="troubleshooting">
  ## 故障排查
</div>

**SSH 连接被拒绝**

在创建 VM 后，SSH 密钥可能需要 1–2 分钟才能生效。请等待片刻后再重试。

**OS Login 问题**

检查你的 OS Login 配置：

```bash
gcloud compute os-login describe-profile
```

确保你的账号拥有所需的 IAM 权限（Compute OS Login 或 Compute OS Admin Login）。

**内存不足（OOM）**

如果你使用的是 e2-micro 实例并遇到 OOM 问题，请升级到 e2-small 或 e2-medium：

```bash
# 首先停止 VM
gcloud compute instances stop openclaw-gateway --zone=us-central1-a

# 更改机器类型
gcloud compute instances set-machine-type openclaw-gateway \
  --zone=us-central1-a \
  --machine-type=e2-small

# 启动 VM
gcloud compute instances start openclaw-gateway --zone=us-central1-a
```

***

<div id="service-accounts-security-best-practice">
  ## 服务账户（安全最佳实践）
</div>

对于个人使用场景，你的默认用户账户就足够了。

对于自动化或 CI/CD 流水线，请创建一个具有最小权限的专用服务账户：

1. 创建服务账户：
   ```bash
   gcloud iam service-accounts create openclaw-deploy \
     --display-name="OpenClaw Deployment"
   ```

2. 授予 Compute Instance Admin 角色（或权限范围更小的自定义角色）：
   ```bash
   gcloud projects add-iam-policy-binding my-openclaw-project \
     --member="serviceAccount:openclaw-deploy@my-openclaw-project.iam.gserviceaccount.com" \
     --role="roles/compute.instanceAdmin.v1"
   ```

避免在自动化中使用 Owner 角色。请遵循最小权限原则。

有关 IAM 角色的详细信息，请参阅 https://cloud.google.com/iam/docs/understanding-roles。

***

<div id="next-steps">
  ## 后续步骤
</div>

* 设置消息通道：[Channels](/zh/channels)
* 将本地设备配对为节点：[Nodes](/zh/nodes)
* 配置 Gateway：[Gateway configuration](/zh/gateway/configuration)