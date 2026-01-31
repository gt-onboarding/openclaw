---
title: 后台进程
summary: "后台 exec 执行与进程管理"
read_when:
  - 添加或修改后台 exec 行为
  - 调试长时间运行的 exec 任务
---

<div id="background-exec-process-tool">
  # 后台 Exec 与 Process 工具
</div>

OpenClaw 通过 `exec` 工具运行 shell 命令，并将长时间运行的任务维持在内存中。`process` 工具负责管理这些后台会话。

<div id="exec-tool">
  ## exec 工具
</div>

关键参数：

- `command`（必填）
- `yieldMs`（默认 10000）：在该延迟后自动转入后台
- `background`（布尔值）：立即在后台运行
- `timeout`（秒，默认 1800）：在达到该超时时间后终止进程
- `elevated`（布尔值）：如果已启用/允许提权模式，则在宿主机上运行
- 需要真实 TTY？设置 `pty: true`。
- `workdir`, `env`

行为：

- 前台运行会直接返回输出。
- 当在后台运行（显式指定或因超时）时，工具会返回 `status: "running"` + `sessionId`，以及末尾的一小段输出。
- 输出会保存在内存中，直到该会话被轮询或清除。
- 如果 `process` 工具被禁用，`exec` 将以同步方式运行，并忽略 `yieldMs`/`background`。

<div id="child-process-bridging">
  ## 子进程桥接
</div>

在 exec/process 工具之外启动长时间运行的子进程时（例如 CLI 重新拉起或 Gateway 辅助进程），请附加子进程桥接辅助工具，以便转发终止信号，并在退出/出错时解除监听器绑定。这样可以避免在 systemd 上遗留孤儿进程，并在各平台之间保持一致的关闭行为。

环境变量覆盖配置：

- `PI_BASH_YIELD_MS`：默认让出间隔（毫秒）
- `PI_BASH_MAX_OUTPUT_CHARS`：内存中输出上限（字符数）
- `OPENCLAW_BASH_PENDING_MAX_OUTPUT_CHARS`：每个流挂起的 stdout/stderr 上限（字符数）
- `PI_BASH_JOB_TTL_MS`：已完成会话的 TTL（存活时间，毫秒，限制在 1 分钟–3 小时）

推荐通过配置设置：

- `tools.exec.backgroundMs`（默认 10000）
- `tools.exec.timeoutSec`（默认 1800）
- `tools.exec.cleanupMs`（默认 1800000）
 - `tools.exec.notifyOnExit`（默认 true）：当后台 exec 进程退出时，将一个系统事件入队，并请求心跳。

<div id="process-tool">
  ## process 工具
</div>

操作：

- `list`：正在运行和已完成的会话
- `poll`：拉取并清空某个会话的新输出（同时报告退出状态）
- `log`：读取汇总输出（支持 `offset` 和 `limit`）
- `write`：发送 stdin（`data`，可选 `eof`）
- `kill`：终止一个后台会话
- `clear`：从内存中移除已完成的会话
- `remove`：如果正在运行则执行 kill，否则在已完成时执行 clear

注意：

- 只有在后台运行的会话才会被列出并保存在内存中。
- 进程重启后会话会丢失（不做磁盘持久化）。
- 只有当你运行 `process poll/log` 且工具结果被记录时，会话日志才会保存到聊天历史。
- `process` 按智能体进行 scope 限定；它只能看到由该智能体启动的会话。
- `process list` 会包含一个派生的 `name`（命令动词 + 目标），便于快速扫描。
- `process log` 使用基于行的 `offset`/`limit`（省略 `offset` 以获取最后的 N 行）。

<div id="examples">
  ## 示例
</div>

执行一个长时间任务，稍后再轮询：

```json
{"tool": "exec", "command": "sleep 5 && echo done", "yieldMs": 1000}
```

```json
{"tool": "process", "action": "poll", "会话 ID": "<id>"}
```

立即在后台启动：

```json
{"tool": "exec", "command": "npm run build", "background": true}
```

发送到 stdin：

```json
{"tool": "process", "action": "write", "sessionId": "<id>", "data": "y\n"}
```
