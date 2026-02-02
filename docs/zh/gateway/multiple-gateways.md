---
title: 多个 Gateway
summary: "在一台主机上运行多个 OpenClaw Gateway（隔离、端口和配置集）"
read_when:
  - 在同一台机器上运行多个 Gateway
  - 需要为每个 Gateway 隔离配置/状态/端口
---

<div id="multiple-gateways-same-host">
  # 多个 Gateway（同一主机）
</div>

大多数部署场景应只使用一个 Gateway，因为单个 Gateway 就可以处理多个消息连接和智能体。如果你需要更强的隔离性或冗余（例如一个应急机器人），可以运行多个使用独立配置文件和端口的 Gateway。

<div id="isolation-checklist-required">
  ## 隔离检查清单（必做）
</div>

- `OPENCLAW_CONFIG_PATH` — 按实例独立的配置文件
- `OPENCLAW_STATE_DIR` — 按实例独立的会话、凭据、缓存
- `agents.defaults.workspace` — 按实例独立的工作区根目录
- `gateway.port`（或 `--port`）— 每个实例唯一的端口
- 派生端口（浏览器/canvas）不得重叠

如果这些被共享，你会遇到配置竞态和端口冲突。

<div id="recommended-profiles-profile">
  ## 推荐：profiles（`--profile`）
</div>

Profiles 会自动为 `OPENCLAW_STATE_DIR` 和 `OPENCLAW_CONFIG_PATH` 设定 scope，并在服务名后追加后缀。

```bash
# main
openclaw --profile main setup
openclaw --profile main gateway --port 18789

# rescue
openclaw --profile rescue setup
openclaw --profile rescue gateway --port 19001
```

按 Profile 区分的服务：

```bash
openclaw --profile main gateway install
openclaw --profile rescue gateway install
```


<div id="rescue-bot-guide">
  ## 救援机器人指南
</div>

在同一台主机上运行第二个 Gateway 实例，使其具有独立的：

- profile/config（配置文件）
- state dir（状态目录）
- 工作区
- 基础端口（以及派生端口）

这样可以将救援机器人与主机器人隔离开来，即使主机器人宕机，它也仍然可以用于调试或应用配置更改。

端口间距：基础端口之间至少预留 20 个端口间隔，这样派生的浏览器/canvas/CDP 端口就不会发生冲突。

<div id="how-to-install-rescue-bot">
  ### 如何安装（救援 Bot）
</div>

```bash
# 主 bot(现有或全新,不带 --profile 参数)
# 运行在端口 18789 + Chrome CDC/Canvas/... 端口
openclaw onboard
openclaw gateway install

# 救援 bot(隔离的 profile + 端口)
openclaw --profile rescue onboard
# 注意事项:
# - 工作区名称默认会添加 -rescue 后缀
# - 端口应至少为 18789 + 20 个端口,
#   建议选择完全不同的基础端口,例如 19789,
# - 其余的 onboarding 流程与正常流程相同

# 安装服务(如果在 onboarding 过程中未自动完成)
openclaw --profile rescue gateway install
```


<div id="port-mapping-derived">
  ## 端口映射（派生）
</div>

基础端口 = `gateway.port`（或 `OPENCLAW_GATEWAY_PORT` / `--port`）。

- 浏览器控制服务端口 = 基础端口 + 2（仅限回环地址）
- `canvasHost.port = 基础端口 + 4`
- 浏览器 Profile 的 CDP 端口会从 `browser.controlPort + 9 .. + 108` 自动分配

如果你在配置或环境变量中覆盖了其中任意一个端口，必须确保在每个实例中它们都互不冲突（保持唯一）。

<div id="browsercdp-notes-common-footgun">
  ## Browser/CDP 说明（常见踩坑点）
</div>

- **不要**在多个实例上将 `browser.cdpUrl` 固定为相同的值。
- 每个实例都需要自己独立的浏览器控制端口和 CDP 端口范围（从各自的 Gateway 端口派生）。
- 如果需要显式指定 CDP 端口，请在每个实例上设置 `browser.profiles.<name>.cdpPort`。
- 远程 Chrome：使用 `browser.profiles.<name>.cdpUrl`（按 profile、按实例分别配置）。

<div id="manual-env-example">
  ## 手动配置环境示例
</div>

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/main.json \
OPENCLAW_STATE_DIR=~/.openclaw-main \
openclaw gateway --port 18789

OPENCLAW_CONFIG_PATH=~/.openclaw/rescue.json \
OPENCLAW_STATE_DIR=~/.openclaw-rescue \
openclaw gateway --port 19001
```


<div id="quick-checks">
  ## 快速检查
</div>

```bash
openclaw --profile main status
openclaw --profile rescue status
openclaw --profile rescue browser status
```
