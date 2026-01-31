---
title: Gateway 锁
summary: "使用 WebSocket 监听端口绑定实现 Gateway 单实例保护"
read_when:
  - 运行或调试 Gateway 进程时
  - 排查或分析单实例强制机制时
---

<div id="gateway-lock">
  # Gateway 锁定
</div>

最后更新：2025-12-11

<div id="why">
  ## 为什么
</div>

- 确保在同一主机上，每个基础端口只运行一个 Gateway 实例；额外的 Gateway 必须使用隔离的运行配置（profile）和唯一的端口。
- 在崩溃或收到 SIGKILL 时，不会留下陈旧的锁文件。
- 当控制端口已被占用时，能够快速失败并给出清晰的错误信息。

<div id="mechanism">
  ## 机制
</div>

- Gateway 在启动时会立即使用专用 TCP 监听器绑定 WebSocket 监听端口（默认 `ws://127.0.0.1:18789`）。
- 如果绑定失败并返回 `EADDRINUSE`，启动过程会抛出 `GatewayLockError("another gateway instance is already listening on ws://127.0.0.1:<port>")`。
- 操作系统会在任意进程退出时（包括崩溃或被 SIGKILL 杀死时）自动释放监听器——不需要单独的锁文件或额外的清理步骤。
- 在关闭时，Gateway 会关闭 WebSocket 服务器及其下层的 HTTP 服务器，以便及时释放端口。

<div id="error-surface">
  ## 错误暴露
</div>

- 如果端口已被其他进程占用，启动时会抛出 `GatewayLockError("another gateway instance is already listening on ws://127.0.0.1:<port>")`。
- 其他绑定失败会同样抛出 `GatewayLockError("failed to bind gateway socket on ws://127.0.0.1:<port>: …")`。

<div id="operational-notes">
  ## 运行注意事项
</div>

- 如果端口被*其他*进程占用，会出现同样的错误；释放该端口，或使用 `openclaw gateway --port <port>` 选择其他端口。
- 在启动 Gateway 之前，macOS 应用仍然会维护自己轻量级的 PID 保护机制；运行时锁则由 WebSocket 绑定来强制执行。