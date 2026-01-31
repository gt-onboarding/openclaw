---
title: Acp
summary: "运行 ACP 桥接程序以用于 IDE 集成"
read_when:
  - 设置基于 ACP 的 IDE 集成
  - 调试从 ACP 会话到 Gateway 的路由
---

<div id="acp">
  # acp
</div>

运行与 OpenClaw Gateway 通信的 ACP（Agent Client Protocol）桥接进程。

该命令通过 stdio 使用 ACP 与 IDE 通信，并通过 WebSocket 将提示转发到 Gateway。
它会保持 ACP 会话与 Gateway 会话键之间的映射关系。

<div id="usage">
  ## 使用方法
</div>

```bash
openclaw acp

# Remote Gateway
openclaw acp --url wss://gateway-host:18789 --token <token>

# Attach to an existing session key
openclaw acp --session agent:main:main

# Attach by label (must already exist)
openclaw acp --session-label "support inbox"

# 在第一个提示前重置会话键
openclaw acp --session agent:main:main --reset-session
```

<div id="acp-client-debug">
  ## ACP 客户端（调试）
</div>

使用内置的 ACP 客户端，在没有 IDE 的情况下对 ACP bridge 做基本检查。
它会启动 ACP bridge，并允许你以交互方式输入提示。

```bash
openclaw acp client

# Point the spawned bridge at a remote Gateway
openclaw acp client --server-args --url wss://gateway-host:18789 --token <token>

# 覆盖服务器命令(默认值:openclaw)
openclaw acp client --server "node" --server-args openclaw.mjs acp --url ws://127.0.0.1:19001
```

<div id="how-to-use-this">
  ## 如何使用
</div>

当 IDE（或其他客户端）支持 Agent Client Protocol 并且你希望它来驱动一个 OpenClaw Gateway 会话时，就使用 ACP。

1. 确保 Gateway 已在运行（本地或远程）。
2. 配置要连接的 Gateway 目标（通过配置文件或命令行标志）。
3. 将你的 IDE 配置为通过 stdio 运行 `openclaw acp`。

示例配置（持久化）：

```bash
openclaw config set gateway.remote.url wss://gateway-host:18789
openclaw config set gateway.remote.token <token>
```

示例：直接运行（不写入配置文件）：

```bash
openclaw acp --url wss://gateway-host:18789 --token <token>
```

<div id="selecting-agents">
  ## 选择智能体
</div>

ACP 不会直接选择智能体。它是根据 Gateway 的会话键进行路由的。

使用以智能体为 scope 的会话键来将请求路由到特定智能体：

```bash
openclaw acp --session agent:main:main
openclaw acp --session agent:design:main
openclaw acp --session agent:qa:bug-123
```

每个 ACP 会话都会映射到一个单一的 Gateway 会话 key。一个 Agent 代理可以拥有多个
会话；ACP 默认使用一个隔离的 `acp:&lt;uuid&gt;` 会话，除非你自定义该 key 或标签。

<div id="zed-editor-setup">
  ## Zed 编辑器设置
</div>

在 `~/.config/zed/settings.json` 文件中添加自定义 ACP Agent 代理（或使用 Zed 的 Settings UI）：

```json
{
  "agent_servers": {
    "OpenClaw ACP": {
      "type": "custom",
      "command": "openclaw",
      "args": ["acp"],
      "env": {}
    }
  }
}
```

要针对特定的 Gateway 或智能体：

```json
{
  "agent_servers": {
    "OpenClaw ACP": {
      "type": "custom",
      "command": "openclaw",
      "args": [
        "acp",
        "--url", "wss://gateway-host:18789",
        "--token", "<token>",
        "--session", "agent:design:main"
      ],
      "env": {}
    }
  }
}
```

在 Zed 中打开 Agent 面板，选择 “OpenClaw ACP” 来启动一个会话。

<div id="session-mapping">
  ## 会话映射
</div>

默认情况下，ACP 会话会获得一个前缀为 `acp:` 的独立 Gateway 会话键。
要复用一个已知会话，可以传入会话键或标签：

* `--session <key>`：使用指定的 Gateway 会话键。
* `--session-label <label>`：通过标签定位并使用已有会话。
* `--reset-session`：为该键生成一个新的会话 ID（相同键，新的对话记录）。

如果你的 ACP 客户端支持元数据，你可以按会话进行覆盖：

```json
{
  "_meta": {
    "sessionKey": "agent:main:main",
    "sessionLabel": "支持收件箱",
    "resetSession": true
  }
}
```

在 [/concepts/session](/zh/concepts/session) 了解更多会话键相关内容。

<div id="options">
  ## 选项
</div>

* `--url <url>`: Gateway WebSocket URL（配置时默认为 gateway.remote.url）。
* `--token <token>`: Gateway 认证令牌。
* `--password <password>`: Gateway 认证密码。
* `--session <key>`: 默认会话 key。
* `--session-label <label>`: 用于解析的默认会话标签。
* `--require-existing`: 如果会话 key/标签不存在则报错。
* `--reset-session`: 在首次使用前重置会话 key。
* `--no-prefix-cwd`: 不要在提示前加上当前工作目录前缀。
* `--verbose, -v`: 将详细日志输出到标准错误(stderr)。

<div id="acp-client-options">
  ### `acp client` 选项
</div>

* `--cwd <dir>`: ACP 会话的工作目录。
* `--server <command>`: ACP 服务器命令（默认：`openclaw`）。
* `--server-args <args...>`: 传递给 ACP 服务器的额外参数。
* `--server-verbose`: 在 ACP 服务器上启用详细日志记录。
* `--verbose, -v`: 启用客户端详细日志记录。