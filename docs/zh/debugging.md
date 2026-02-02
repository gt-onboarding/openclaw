---
title: 调试
summary: "调试工具：监视模式、原始模型流，以及追踪推理泄露"
read_when:
  - 你需要查看原始模型输出中的推理泄露
  - 你想在迭代开发时以监视模式运行 Gateway
  - 你需要一套可重复的调试流程
---

<div id="debugging">
  # 调试
</div>

本页介绍用于流式输出的调试辅助功能，尤其是在提供方将推理内容混入正常文本时。

<div id="runtime-debug-overrides">
  ## 运行时调试覆盖
</div>

在对话中使用 `/debug` 来设置**仅在运行时生效**的配置覆盖（仅存于内存，不会写入磁盘）。
`/debug` 默认处于禁用状态；可通过设置 `commands.debug: true` 启用。
当你需要切换一些较隐蔽的设置，又不想编辑 `openclaw.json` 时，这一功能非常有用。

示例：

```
/debug show
/debug set messages.responsePrefix="[openclaw]"
/debug unset messages.responsePrefix
/debug reset
```

`/debug reset` 会清除所有覆盖配置，并恢复为磁盘上的配置文件设置。


<div id="gateway-watch-mode">
  ## Gateway 监听模式
</div>

为了快速迭代，使用文件监听器运行 Gateway：

```bash
pnpm gateway:watch --force
```

其对应为：

```bash
tsx watch src/entry.ts gateway --force
```

在 `gateway:watch` 后添加任意 Gateway CLI 标志参数，这些参数会在每次重启时被原样传递。


<div id="dev-profile-dev-gateway-dev">
  ## Dev profile + dev gateway (--dev)
</div>

使用 dev profile 来隔离状态，并启动一个安全、可丢弃的调试环境。`--dev` 选项有**两个**：

* **全局 `--dev`（profile）：** 将状态隔离到 `~/.openclaw-dev` 下，并将 Gateway 端口默认设置为 `19001`（相关派生端口会随之偏移）。
* **`gateway --dev`：告诉 Gateway 在缺失时自动创建默认配置 +
  工作区**（并跳过 BOOTSTRAP.md）。

推荐流程（dev profile + dev bootstrap）：

```bash
pnpm gateway:dev
OPENCLAW_PROFILE=dev openclaw tui
```

如果你还没有全局安装，可以通过 `pnpm openclaw ...` 来运行 CLI。

这会执行以下操作：

1. **Profile 隔离**（全局 `--dev`）
   * `OPENCLAW_PROFILE=dev`
   * `OPENCLAW_STATE_DIR=~/.openclaw-dev`
   * `OPENCLAW_CONFIG_PATH=~/.openclaw-dev/openclaw.json`
   * `OPENCLAW_GATEWAY_PORT=19001`（浏览器/画布端口相应调整）

2. **开发引导**（`gateway --dev`）
   * 如果缺失，则写入一个最小配置（`gateway.mode=local`，绑定回环地址）。
   * 将 `agent.workspace` 设置为开发工作区。
   * 将 `agent.skipBootstrap=true`（不再使用 BOOTSTRAP.md）。
   * 如果缺失，则在工作区中初始化以下文件：
     `AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`。
   * 默认身份：**C3‑PO**（礼仪机器人，protocol droid）。
   * 在开发模式下跳过通道提供方（`OPENCLAW_SKIP_CHANNELS=1`）。

重置流程（全新开始）：

```bash
pnpm gateway:dev:reset
```

注意：`--dev` 是一个**全局** profile 标志，并且会被某些运行器忽略。
如果你需要显式指定它，请使用环境变量形式：

```bash
OPENCLAW_PROFILE=dev openclaw gateway --dev --reset
```

`--reset` 会清除配置、凭据、会话和开发工作区（使用
`trash` 而不是 `rm`），然后重新创建默认的开发环境设置。

提示：如果已有非开发环境的 Gateway 正在运行（launchd/systemd），请先将其停止：

```bash
openclaw gateway stop
```


<div id="raw-stream-logging-openclaw">
  ## 原始流日志（OpenClaw）
</div>

OpenClaw 可以在任何过滤或格式化之前记录**原始助手流**。
这是查看推理是以纯文本增量输出
（还是作为单独的思考块）返回的最佳方式。

通过 CLI 启用：

```bash
pnpm gateway:watch --force --raw-stream
```

（可选）自定义路径：

```bash
pnpm gateway:watch --force --raw-stream --raw-stream-path ~/.openclaw/logs/raw-stream.jsonl
```

等效环境变量：

```bash
OPENCLAW_RAW_STREAM=1
OPENCLAW_RAW_STREAM_PATH=~/.openclaw/logs/raw-stream.jsonl
```

默认文件：

`~/.openclaw/logs/raw-stream.jsonl`


<div id="raw-chunk-logging-pi-mono">
  ## 原始分块日志记录（pi-mono）
</div>

要在 **OpenAI 兼容的原始分块** 被解析为区块之前进行捕获，
pi-mono 提供了一个单独的日志记录器：

```bash
PI_RAW_STREAM=1
```

（可选）路径：

```bash
PI_RAW_STREAM_PATH=~/.pi-mono/logs/raw-openai-completions.jsonl
```

默认文件：

`~/.pi-mono/logs/raw-openai-completions.jsonl`

> 注意：只有使用 pi-mono 的
> `openai-completions` 提供方的进程才会输出该文件。


<div id="safety-notes">
  ## 安全注意事项
</div>

- 原始流式日志可能包含完整的提示词、工具输出和用户数据。
- 将日志保存在本地，并在调试完成后删除。
- 如果需要共享日志，先清除其中的机密信息和个人身份信息（PII）。