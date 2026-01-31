---
title: Xpc
summary: "用于 OpenClaw 应用、Gateway 节点传输和 PeekabooBridge 的 macOS IPC 架构"
read_when:
  - 编辑 IPC 约定或菜单栏应用的 IPC
---

<div id="openclaw-macos-ipc-architecture">
  # OpenClaw macOS IPC 架构
</div>

**当前模型：** 使用本地 Unix 套接字将 **node host service** 连接到 **macOS app**，用于执行操作授权和 `system.run`。提供一个 `openclaw-mac` 调试 CLI，用于发现和连接性检查；智能体动作仍然通过 Gateway WebSocket 和 `node.invoke` 进行传递。UI 自动化使用 PeekabooBridge。

<div id="goals">
  ## 目标
</div>

* 单个 GUI 应用实例，统一负责所有面向 TCC 的工作（通知、屏幕录制、麦克风、语音、AppleScript）。
* 简洁的自动化接口：Gateway + 节点命令，加上用于 UI 自动化的 PeekabooBridge。
* 可预测的权限行为：始终使用相同的已签名 bundle ID，由 launchd 启动，从而确保 TCC 授权能够持续生效。

<div id="how-it-works">
  ## 工作原理
</div>

<div id="gateway-node-transport">
  ### Gateway + 节点通信
</div>

* 应用在本地模式下运行 Gateway，并以节点身份连接到它。
* 智能体操作通过 `node.invoke` 执行（例如 `system.run`、`system.notify`、`canvas.*`）。

<div id="node-service-app-ipc">
  ### 节点服务 + 应用 IPC
</div>

* 无界面节点宿主服务连接到 Gateway 的 WebSocket。
* `system.run` 请求通过本地 Unix 套接字转发到 macOS 应用。
* 应用在 UI 上下文中执行，按需提示用户，并返回输出。

示意图（SCI）：

```
Agent -> Gateway -> Node Service (WS)
                      |  IPC (UDS + token + HMAC + TTL)
                      v
                  Mac App (UI + TCC + system.run)
```

<div id="peekaboobridge-ui-automation">
  ### PeekabooBridge（UI 自动化）
</div>

* UI 自动化通过名为 `bridge.sock` 的独立 UNIX 套接字并使用 PeekabooBridge JSON 协议。
* 主机优先顺序（客户端侧）：Peekaboo.app → Claude.app → OpenClaw.app → 本地执行。
* 安全性：bridge 宿主必须具备被允许的 TeamID；仅限 DEBUG 的同一 UID 紧急通道由 `PEEKABOO_ALLOW_UNSIGNED_SOCKET_CLIENTS=1`（Peekaboo 约定）加以防护。
* 参见：[PeekabooBridge 使用方法](/zh/platforms/mac/peekaboo) 以了解详情。

<div id="operational-flows">
  ## 运行流程
</div>

* 重启/重建：`SIGN_IDENTITY="Apple Development: <Developer Name> (<TEAMID>)" scripts/restart-mac.sh`
  * 终止现有实例进程
  * 进行 Swift 构建和打包
  * 写入并引导/拉起 LaunchAgent
* 单实例：如果已有使用相同 bundle ID 运行的实例，应用会立即退出。

<div id="hardening-notes">
  ## 加固注意事项
</div>

* 优先要求所有特权界面都进行 TeamID 匹配。
* PeekabooBridge：`PEEKABOO_ALLOW_UNSIGNED_SOCKET_CLIENTS=1`（仅用于 DEBUG）可允许相同 UID 的调用方向本地开发套接字发起调用。
* 所有通信均仅限本地，不会暴露任何网络套接字。
* TCC 提示仅来自 GUI 应用 bundle；在重新构建时保持已签名的 bundle ID 不变。
* IPC 加固：socket 模式设为 `0600`、令牌、对端 UID 校验、HMAC 质询/响应、较短 TTL。