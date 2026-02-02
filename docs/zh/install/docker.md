---
title: Docker
summary: "基于 Docker 的可选 OpenClaw 安装与上手指南"
read_when:
  - 你希望使用容器化的 Gateway，而不是本地安装
  - 你正在验证基于 Docker 的流程
---

<div id="docker-optional">
  # Docker（可选）
</div>

Docker 是**可选的**。仅当你希望运行容器化的 Gateway 或者想验证 Docker 工作流时再使用它。

<div id="is-docker-right-for-me">
  ## Docker 适合我吗？
</div>

* **是**：你想要一个隔离的、可随时丢弃的 Gateway 环境，或者想在无需本地安装依赖的主机上运行 OpenClaw。
* **否**：你在自己的机器上运行，只是想要最快的开发循环。请改用常规安装流程。
* **沙箱说明**：智能体沙箱同样使用 Docker，但**不**要求整个 Gateway 都运行在 Docker 中。参见 [Sandboxing](/zh/gateway/sandboxing)。

本指南涵盖：

* 容器化 Gateway（在 Docker 中运行完整的 OpenClaw）
* 按会话划分的 Agent 沙箱（本机运行 Gateway + 通过 Docker 隔离的智能体工具）

沙箱详细信息：[Sandboxing](/zh/gateway/sandboxing)

<div id="requirements">
  ## 环境要求
</div>

* Docker Desktop（或 Docker Engine）+ Docker Compose v2
* 足够的磁盘空间用于存储镜像和日志

<div id="containerized-gateway-docker-compose">
  ## 使用 Docker Compose 容器化部署 Gateway
</div>

<div id="quick-start-recommended">
  ### 快速开始（推荐）
</div>

在仓库根目录下执行：

```bash
./docker-setup.sh
```

此脚本将会：

* 构建 Gateway 镜像
* 运行入门向导
* 输出可选的提供方设置提示
* 通过 Docker Compose 启动 Gateway
* 生成一个 Gateway token 并写入 `.env`

可选环境变量：

* `OPENCLAW_DOCKER_APT_PACKAGES` — 在构建期间安装额外的 apt 软件包
* `OPENCLAW_EXTRA_MOUNTS` — 添加额外的宿主机绑定挂载
* `OPENCLAW_HOME_VOLUME` — 将 `/home/node` 持久化到命名卷中

完成后：

* 在浏览器中打开 `http://127.0.0.1:18789/`。
* 将 token 粘贴到 Control UI 中（Settings → token）。

它会在宿主机上写入配置/工作区：

* `~/.openclaw/`
* `~/.openclaw/workspace`

在 VPS 上运行？请参阅 [Hetzner (Docker VPS)](/zh/platforms/hetzner)。

<div id="manual-flow-compose">
  ### 手动流程（Compose）
</div>

```bash
docker build -t openclaw:local -f Dockerfile .
docker compose run --rm openclaw-cli onboard
docker compose up -d openclaw-gateway
```

<div id="extra-mounts-optional">
  ### 额外挂载（可选）
</div>

如果你想将额外的宿主机目录挂载到容器中，请在运行 `docker-setup.sh` 之前设置
`OPENCLAW_EXTRA_MOUNTS`。该变量接受以逗号分隔的 Docker 绑定挂载列表，
并通过生成 `docker-compose.extra.yml` 将这些挂载同时应用到
`openclaw-gateway` 和 `openclaw-cli`。

示例：

```bash
export OPENCLAW_EXTRA_MOUNTS="$HOME/.codex:/home/node/.codex:ro,$HOME/github:/home/node/github:rw"
./docker-setup.sh
```

注意：

* 在 macOS/Windows 上，这些路径必须在 Docker Desktop 中共享。
* 如果你编辑了 `OPENCLAW_EXTRA_MOUNTS`，请重新运行 `docker-setup.sh` 以重新生成额外的 Compose 配置文件。
* `docker-compose.extra.yml` 是自动生成的，请不要手动编辑。

<div id="persist-the-entire-container-home-optional">
  ### 持久化整个容器的 home 目录（可选）
</div>

如果你希望 `/home/node` 在容器重建后依然保留，请通过 `OPENCLAW_HOME_VOLUME`
设置一个命名卷。这样会创建一个 Docker 卷并将其挂载到 `/home/node`，同时保留
标准的配置/工作区绑定挂载。在这里请使用命名卷（而不是绑定路径）；对于绑定挂载，
请使用 `OPENCLAW_EXTRA_MOUNTS`。

示例：

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
./docker-setup.sh
```

你也可以配合额外挂载一起使用：

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
export OPENCLAW_EXTRA_MOUNTS="$HOME/.codex:/home/node/.codex:ro,$HOME/github:/home/node/github:rw"
./docker-setup.sh
```

注意：

* 如果你更改了 `OPENCLAW_HOME_VOLUME`，请重新运行 `docker-setup.sh` 以重新生成
  额外的 Compose 文件。
* 该命名卷会一直保留，直到使用 `docker volume rm &lt;name&gt;` 将其删除。

<div id="install-extra-apt-packages-optional">
  ### 安装额外的 apt 软件包（可选）
</div>

如果你需要在镜像中安装系统软件包（例如构建工具或媒体库），请在运行 `docker-setup.sh` 之前设置 `OPENCLAW_DOCKER_APT_PACKAGES`。
这些软件包会在镜像构建过程中被安装，因此即使容器被删除，这些软件包也会保留。

示例：

```bash
export OPENCLAW_DOCKER_APT_PACKAGES="ffmpeg build-essential"
./docker-setup.sh
```

Notes:

* 此处接受以空格分隔的 apt 软件包名称列表。
* 如果你更改了 `OPENCLAW_DOCKER_APT_PACKAGES`，请重新运行 `docker-setup.sh` 以重新构建镜像。

<div id="faster-rebuilds-recommended">
  ### 更快的重新构建（推荐）
</div>

为加快重新构建速度，请调整 Dockerfile 中的指令顺序，使依赖层能够被缓存。
这样可以避免在锁文件未更改时重新运行 `pnpm install`：

```dockerfile
FROM node:22-bookworm

# 安装 Bun(构建脚本必需)
RUN curl -fsSL https://bun.sh/install | bash
ENV PATH="/root/.bun/bin:${PATH}"

RUN corepack enable

WORKDIR /app

# 缓存依赖项,除非包元数据更改
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml .npmrc ./
COPY ui/package.json ./ui/package.json
COPY scripts ./scripts

RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build
RUN pnpm ui:install
RUN pnpm ui:build

ENV NODE_ENV=production

CMD ["node","dist/index.js"]
```

<div id="channel-setup-optional">
  ### 通道设置（可选）
</div>

使用 CLI 容器配置通道，然后在需要时重启 Gateway。

WhatsApp（二维码）：

```bash
docker compose run --rm openclaw-cli channels login
```

Telegram（Bot Token）：

```bash
docker compose run --rm openclaw-cli channels add --channel telegram --token "<token>"
```

Discord（Bot Token）：

```bash
docker compose run --rm openclaw-cli channels add --channel discord --token "<token>"
```

文档：[WhatsApp](/zh/channels/whatsapp)、[Telegram](/zh/channels/telegram)、[Discord](/zh/channels/discord)

<div id="health-check">
  ### 健康检查
</div>

```bash
docker compose exec openclaw-gateway node dist/index.js health --token "$OPENCLAW_GATEWAY_TOKEN"
```

<div id="e2e-smoke-test-docker">
  ### 端到端（E2E）冒烟测试（Docker）
</div>

```bash
scripts/e2e/onboard-docker.sh
```

<div id="qr-import-smoke-test-docker">
  ### 二维码导入冒烟测试（Docker）
</div>

```bash
pnpm test:docker:qr
```

<div id="notes">
  ### 注意事项
</div>

* 在容器使用场景下，Gateway 的绑定默认为 `lan`。
* Gateway 容器是会话的唯一可信来源（`~/.openclaw/agents/<agentId>/sessions/`）。

<div id="agent-sandbox-host-gateway-docker-tools">
  ## Agent 沙箱（宿主 Gateway + Docker 工具）
</div>

深入解析：[沙箱隔离](/zh/gateway/sandboxing)

<div id="what-it-does">
  ### 功能说明
</div>

当启用 `agents.defaults.sandbox` 时，**非主会话** 会在 Docker 容器内运行工具。Gateway 仍然运行在你的宿主机上，但工具执行是隔离的：

* scope：默认为 `"agent"`（每个智能体对应一个容器 + 一个工作区）
* scope：`"session"` 表示按会话级别隔离
* 每个 scope 对应的工作区目录挂载在 `/workspace`
* 可选的智能体工作区访问控制（`agents.defaults.sandbox.workspaceAccess`）
* 工具允许/拒绝策略（拒绝优先生效）
* 入站媒体会被复制到当前沙箱工作区中（`media/inbound/*`），这样工具可以对其进行 read 操作（在 `workspaceAccess: "rw"` 时，这些内容会落在智能体工作区中）

警告：`scope: "shared"` 会禁用跨会话隔离。所有会话将共享同一个容器和同一个工作区。

<div id="per-agent-sandbox-profiles-multi-agent">
  ### 按智能体划分的沙箱配置（多智能体）
</div>

如果你使用多智能体路由，每个智能体都可以单独覆写沙箱和工具设置：
`agents.list[].sandbox` 和 `agents.list[].tools`（以及 `agents.list[].tools.sandbox.tools`）。这使你可以在同一个 Gateway 中运行
具有不同访问级别的智能体：

* 完全访问权限（个人智能体）
* 只读工具 + 只读工作区（家庭/工作智能体）
* 无文件系统/命令行工具（公共智能体）

示例、优先级规则和故障排查请参见 [多智能体沙箱与工具](/zh/multi-agent-sandbox-tools)。

<div id="default-behavior">
  ### 默认行为
</div>

* 镜像：`openclaw-sandbox:bookworm-slim`
* 每个智能体一个容器
* Agent 代理的工作区访问：`workspaceAccess: "none"`（默认）使用 `~/.openclaw/sandboxes`
  * `"ro"` 将沙箱工作区保持在 `/workspace`，并将智能体工作区以只读方式挂载在 `/agent`（禁用 `write`/`edit`/`apply_patch`）
  * `"rw"` 将智能体工作区以读写方式挂载在 `/workspace`
* 自动清理：空闲时间 &gt; 24 小时或存活时间 &gt; 7 天
* 网络：默认 `none`（如需出站访问必须显式开启）
* 默认允许：`exec`、`process`、`read`、`write`、`edit`、`sessions_list`、`sessions_history`、`sessions_send`、`sessions_spawn`、`session_status`
* 默认拒绝：`browser`、`canvas`、`nodes`、`cron`、`discord`、`gateway`

<div id="enable-sandboxing">
  ### 启用沙箱
</div>

如果你打算在 `setupCommand` 中安装软件包，请注意：

* 默认的 `docker.network` 是 `"none"`（无出站网络访问）。
* `readOnlyRoot: true` 会阻止安装软件包。
* `user` 必须是 root 才能使用 `apt-get`（省略 `user` 或设置为 `user: "0:0"`）。
  当 `setupCommand`（或 docker 配置）发生变化时，除非容器在**最近使用过**（约 5 分钟内），否则 OpenClaw 会自动重新创建容器。处于热状态的容器
  会在日志中输出一条包含精确 `openclaw sandbox recreate ...` 命令的警告。

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // off | non-main | all
        scope: "agent", // session | agent | shared (agent 为默认值)
        workspaceAccess: "none", // none | ro | rw
        workspaceRoot: "~/.openclaw/sandboxes",
        docker: {
          image: "openclaw-sandbox:bookworm-slim",
          workdir: "/workspace",
          readOnlyRoot: true,
          tmpfs: ["/tmp", "/var/tmp", "/run"],
          network: "none",
          user: "1000:1000",
          capDrop: ["ALL"],
          env: { LANG: "C.UTF-8" },
          setupCommand: "apt-get update && apt-get install -y git curl jq",
          pidsLimit: 256,
          memory: "1g",
          memorySwap: "2g",
          cpus: 1,
          ulimits: {
            nofile: { soft: 1024, hard: 2048 },
            nproc: 256
          },
          seccompProfile: "/path/to/seccomp.json",
          apparmorProfile: "openclaw-sandbox",
          dns: ["1.1.1.1", "8.8.8.8"],
          extraHosts: ["internal.service:10.0.0.5"]
        },
        prune: {
          idleHours: 24, // 0 disables idle pruning
          maxAgeDays: 7  // 0 disables max-age pruning
        }
      }
    }
  },
  tools: {
    sandbox: {
      tools: {
        allow: ["exec", "process", "read", "write", "edit", "sessions_list", "sessions_history", "sessions_send", "sessions_spawn", "session_status"],
        deny: ["browser", "canvas", "nodes", "cron", "discord", "gateway"]
      }
    }
  }
}
```

安全加固相关的配置项位于 `agents.defaults.sandbox.docker`：
`network`, `user`, `pidsLimit`, `memory`, `memorySwap`, `cpus`, `ulimits`,
`seccompProfile`, `apparmorProfile`, `dns`, `extraHosts`。

多智能体场景：可以通过 `agents.list[].sandbox.{docker,browser,prune}.*` 为每个智能体单独覆盖 `agents.defaults.sandbox.{docker,browser,prune}.*`
（当 `agents.defaults.sandbox.scope` / `agents.list[].sandbox.scope` 为 `"shared"` 时，这些覆盖会被忽略）。

<div id="build-the-default-sandbox-image">
  ### 构建默认的沙箱镜像
</div>

```bash
scripts/sandbox-setup.sh
```

这将使用 `Dockerfile.sandbox` 来构建 `openclaw-sandbox:bookworm-slim` 镜像。

<div id="sandbox-common-image-optional">
  ### 通用沙箱镜像（可选）
</div>

如果你需要一个包含常用构建工具（Node、Go、Rust 等）的沙箱镜像，请构建该通用镜像：

```bash
scripts/sandbox-common-setup.sh
```

这将构建 `openclaw-sandbox-common:bookworm-slim`。要使用该镜像：

```json5
{
  agents: { defaults: { sandbox: { docker: { image: "openclaw-sandbox-common:bookworm-slim" } } } }
}
```

<div id="sandbox-browser-image">
  ### 沙箱中的浏览器镜像
</div>

要在沙箱中运行浏览器工具，请先构建浏览器镜像：

```bash
scripts/sandbox-browser-setup.sh
```

这会使用 `Dockerfile.sandbox-browser` 构建镜像 `openclaw-sandbox-browser:bookworm-slim`。该容器运行启用 CDP 的 Chromium，并提供可选的 noVNC 观察界面（通过 Xvfb 实现有头模式）。

注意：

* 有头模式（Xvfb）相比无头模式更不容易被识别为机器人并被封禁。
* 仍可通过设置 `agents.defaults.sandbox.browser.headless=true` 来使用无头模式。
* 不需要完整桌面环境（GNOME）；Xvfb 即可提供显示能力。

使用如下配置：

```json5
{
  agents: {
    defaults: {
      sandbox: {
        browser: { enabled: true }
      }
    }
  }
}
```

自定义浏览器镜像：

```json5
{
  agents: {
    defaults: {
      sandbox: { browser: { image: "my-openclaw-browser" } }
    }
  }
}
```

启用后，智能体会收到：

* 一个沙箱浏览器控制 URL（用于 `browser` 工具）
* 一个 noVNC URL（如果启用且 headless=false）

请注意：如果你对工具使用了允许列表，需要将 `browser` 添加到允许列表中（并从 `deny` 中移除），否则该工具会继续被阻止。
清理规则（`agents.defaults.sandbox.prune`）同样适用于浏览器容器。

<div id="custom-sandbox-image">
  ### 自定义沙箱镜像
</div>

构建你自己的镜像，并在配置中引用它：

```bash
docker build -t my-openclaw-sbx -f Dockerfile.sandbox .
```

```json5
{
  agents: {
    defaults: {
      sandbox: { docker: { image: "my-openclaw-sbx" } }
    }
  }
}
```

<div id="tool-policy-allowdeny">
  ### 工具策略（允许/禁止）
</div>

* `deny` 优先于 `allow`。
* 如果 `allow` 为空：所有工具（`deny` 中的除外）都可用。
* 如果 `allow` 非空：只有 `allow` 中的工具可用（不包括 `deny` 中的）。

<div id="pruning-strategy">
  ### 清理策略
</div>

两个配置项：

* `prune.idleHours`: 移除在最近 X 小时内未使用的容器（0 = 禁用）
* `prune.maxAgeDays`: 移除创建时间早于 X 天之前的容器（0 = 禁用）

示例：

* 保留活跃会话但设置生命周期上限：
  `idleHours: 24`, `maxAgeDays: 7`
* 从不清理：
  `idleHours: 0`, `maxAgeDays: 0`

<div id="security-notes">
  ### 安全说明
</div>

* 硬边界仅适用于 **tools**（exec/read/write/edit/apply&#95;patch）。
* 像 browser/camera/canvas 这样的仅限宿主机的工具默认会被禁用。
* 在沙箱中允许 `browser` 会**破坏隔离**（browser 实际在宿主机上运行）。

<div id="troubleshooting">
  ## 故障排查
</div>

* 镜像缺失：使用 [`scripts/sandbox-setup.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/sandbox-setup.sh) 构建，或设置 `agents.defaults.sandbox.docker.image`。
* 容器未运行：它会在每个会话中按需自动创建。
* 沙箱中的权限错误：将 `docker.user` 设置为与你挂载的工作区所有者匹配的 UID:GID（或对工作区目录执行 chown）。
* 找不到自定义工具：OpenClaw 使用 `sh -lc`（登录 shell）运行命令，它会加载 `/etc/profile`，并可能重置 PATH。将 `docker.env.PATH` 设置为在前面前置你的自定义工具路径（例如 `/custom/bin:/usr/local/share/npm-global/bin`），或者在 Dockerfile 中在 `/etc/profile.d/` 下添加脚本。