---
title: 日志
summary: "日志概览：文件日志、控制台输出、CLI 实时查看，以及 Control UI"
read_when:
  - 你需要一份适合新手的日志入门概览
  - 你希望配置日志级别或格式
  - 你正在排查问题，并且需要快速定位日志
---

<div id="logging">
  # 日志
</div>

OpenClaw 会在两个位置记录日志：

* **文件日志**（JSON 行），由 Gateway 写入。
* **控制台输出**，显示在终端和 Control UI 中。

本页说明日志的存放位置、如何读取日志，以及如何配置日志级别和格式。

<div id="where-logs-live">
  ## 日志存放位置
</div>

默认情况下，Gateway 会将轮转日志文件写入：

`/tmp/openclaw/openclaw-YYYY-MM-DD.log`

日期使用 Gateway 所在主机的本地时区。

你可以在 `~/.openclaw/openclaw.json` 中覆盖此设置：

```json
{
  "logging": {
    "file": "/path/to/openclaw.log"
  }
}
```

<div id="how-to-read-logs">
  ## 如何查看日志
</div>

<div id="cli-live-tail-recommended">
  ### CLI：实时日志跟踪（推荐）
</div>

使用 CLI 通过 RPC 实时跟踪 Gateway 日志文件：

```bash
openclaw logs --follow
```

输出模式：

* **TTY 会话**：美观的、彩色且结构化的日志行。
* **非 TTY 会话**：纯文本。
* `--json`：按行分隔的 JSON（每行一个日志事件）。
* `--plain`：在 TTY 会话中强制使用纯文本。
* `--no-color`：禁用 ANSI 颜色。

在 JSON 模式下，CLI 会输出带有 `type` 标签的对象：

* `meta`：流的元数据（文件、游标、大小）
* `log`：已解析的日志条目
* `notice`：日志截断/轮转提示
* `raw`：未解析的日志行

如果 Gateway 无法访问，CLI 会打印一条简短提示，建议你运行：

```bash
openclaw doctor
```

<div id="control-ui-web">
  ### Control UI（网页端）
</div>

Control UI 的 **Logs** 选项卡通过 `logs.tail` 对同一个文件进行尾部跟踪。
关于如何打开，请参阅 [/web/control-ui](/zh/web/control-ui)。

<div id="channel-only-logs">
  ### 仅渠道日志
</div>

要筛选渠道活动（WhatsApp/Telegram 等），请使用：

```bash
openclaw channels logs --channel whatsapp
```

<div id="log-formats">
  ## 日志格式
</div>

<div id="file-logs-jsonl">
  ### 文件日志（JSONL）
</div>

日志文件的每一行都是一个 JSON 对象。CLI 和 Control UI 会解析这些条目，并将其渲染为结构化输出（时间、级别、子系统、消息）。

<div id="console-output">
  ### 控制台输出
</div>

控制台日志**具备 TTY 感知能力**，并经过格式化以提高可读性：

* 子系统前缀（例如 `gateway/channels/whatsapp`）
* 按级别着色（info/warn/error）
* 可选的紧凑模式或 JSON 模式

控制台格式由 `logging.consoleStyle` 控制。

<div id="configuring-logging">
  ## 配置日志
</div>

所有日志相关配置都位于 `~/.openclaw/openclaw.json` 文件的 `logging` 字段下。

```json
{
  "logging": {
    "level": "info",
    "file": "/tmp/openclaw/openclaw-YYYY-MM-DD.log",
    "consoleLevel": "info",
    "consoleStyle": "pretty",
    "redactSensitive": "tools",
    "redactPatterns": [
      "sk-.*"
    ]
  }
}
```

<div id="log-levels">
  ### 日志级别
</div>

* `logging.level`：**文件日志**（JSONL）的日志级别。
* `logging.consoleLevel`：**控制台输出**的日志级别（详细程度）。

`--verbose` 只影响控制台输出，不会更改文件日志级别。

<div id="console-styles">
  ### 控制台样式
</div>

`logging.consoleStyle`：

* `pretty`: 友好的可读格式，彩色输出，并带有时间戳。
* `compact`: 更紧凑的输出（适合长时间会话）。
* `json`: 每行一条 JSON 记录（用于日志处理器）。

<div id="redaction">
  ### 脱敏
</div>

工具摘要可以在写入控制台之前对敏感 token 进行脱敏处理：

* `logging.redactSensitive`: `off` | `tools`（默认：`tools`）
* `logging.redactPatterns`: 用于覆盖默认模式集合的正则表达式字符串列表

脱敏仅影响**控制台输出**，不会修改文件日志。

<div id="diagnostics-opentelemetry">
  ## 诊断 + OpenTelemetry
</div>

诊断是与模型运行**以及**
消息流遥测（webhooks、队列、会话状态）相关的结构化、机器可读事件。它们**不会**
替代日志；它们用于为指标、追踪和其他导出器提供数据。

诊断事件在进程内产生，但只有在同时启用诊断功能和导出器插件时，
导出器才会被接入。

<div id="opentelemetry-vs-otlp">
  ### OpenTelemetry 与 OTLP
</div>

* **OpenTelemetry (OTel)**：用于追踪、指标和日志的数据模型与 SDK。
* **OTLP**：用于将 OTel 数据导出到收集器/后端的传输协议。
* OpenClaw 目前通过 **OTLP/HTTP（protobuf）** 进行导出。

<div id="signals-exported">
  ### 导出的信号
</div>

* **Metrics**：计数器和直方图（token 使用量、消息流、排队）。
* **Traces**：用于模型使用和 webhook/消息处理的 span。
* **Logs**：在启用 `diagnostics.otel.logs` 时通过 OTLP 导出。日志量可能较大；请注意 `logging.level` 和导出器过滤规则。

<div id="diagnostic-event-catalog">
  ### 诊断事件目录
</div>

模型使用情况：

* `model.usage`: token 数、成本、时长、上下文、提供方/模型/通道、会话 ID。

消息流：

* `webhook.received`: 各通道收到的 webhook 入口事件。
* `webhook.processed`: webhook 处理完成 + 时长。
* `webhook.error`: webhook 处理器错误。
* `message.queued`: 消息入队等待处理。
* `message.processed`: 处理结果 + 时长 + 可选错误。

队列 + 会话：

* `queue.lane.enqueue`: 命令队列通道入队 + 队列深度。
* `queue.lane.dequeue`: 命令队列通道出队 + 等待时间。
* `session.state`: 会话状态变更 + 原因。
* `session.stuck`: 会话卡住警告 + 滞留时间。
* `run.attempt`: 运行重试/尝试元数据。
* `diagnostic.heartbeat`: 汇总计数器（webhook/队列/会话）。

<div id="enable-diagnostics-no-exporter">
  ### 启用诊断（无导出器）
</div>

如果你希望诊断事件可供插件或自定义 sink 使用，请使用此选项：

```json
{
  "diagnostics": {
    "enabled": true
  }
}
```

<div id="diagnostics-flags-targeted-logs">
  ### 诊断标志（针对性日志）
</div>

使用标志在不提高 `logging.level` 的情况下开启额外的针对性调试日志。
标志不区分大小写，并支持通配符（例如 `telegram.*` 或 `*`）。

```json
{
  "diagnostics": {
    "flags": ["telegram.http"]
  }
}
```

环境变量临时覆盖（一次性）：

```
OPENCLAW_DIAGNOSTICS=telegram.http,telegram.payload
```

注意事项：

* Flag 日志会写入标准日志文件（与 `logging.file` 相同）。
* 输出内容仍会根据 `logging.redactSensitive` 进行脱敏处理。
* 完整指南：[/diagnostics/flags](/zh/diagnostics/flags)。

<div id="export-to-opentelemetry">
  ### 导出到 OpenTelemetry
</div>

可以通过 `diagnostics-otel` 插件（OTLP/HTTP）导出诊断数据。该方式
适用于任何接受 OTLP/HTTP 的 OpenTelemetry 收集器/后端。

```json
{
  "plugins": {
    "allow": ["diagnostics-otel"],
    "entries": {
      "diagnostics-otel": {
        "enabled": true
      }
    }
  },
  "diagnostics": {
    "enabled": true,
    "otel": {
      "enabled": true,
      "endpoint": "http://otel-collector:4318",
      "protocol": "http/protobuf",
      "serviceName": "openclaw-gateway",
      "traces": true,
      "metrics": true,
      "logs": true,
      "sampleRate": 0.2,
      "flushIntervalMs": 60000
    }
  }
}
```

注意：

* 你也可以使用 `openclaw plugins enable diagnostics-otel` 启用该插件。
* `protocol` 当前只支持 `http/protobuf`，`grpc` 会被忽略。
* 指标包括 token 使用量、费用、上下文大小、运行时长，以及消息流
  计数器/直方图（webhook、排队、会话状态、队列深度/等待时间）。
* 可以通过 `traces` / `metrics` 开关启用/禁用追踪与指标（默认：开启）。启用后，追踪
  将包含模型使用 span，以及 webhook/消息处理 span。
* 当你的采集器需要身份验证时，请设置 `headers`。
* 支持的环境变量：`OTEL_EXPORTER_OTLP_ENDPOINT`、
  `OTEL_SERVICE_NAME`、`OTEL_EXPORTER_OTLP_PROTOCOL`。

<div id="exported-metrics-names-types">
  ### 导出的指标（名称及类型）
</div>

模型使用情况：

* `openclaw.tokens`（计数器，属性：`openclaw.token`、`openclaw.channel`、
  `openclaw.provider`、`openclaw.model`）
* `openclaw.cost.usd`（计数器，属性：`openclaw.channel`、`openclaw.provider`、
  `openclaw.model`）
* `openclaw.run.duration_ms`（直方图，属性：`openclaw.channel`、
  `openclaw.provider`、`openclaw.model`）
* `openclaw.context.tokens`（直方图，属性：`openclaw.context`、
  `openclaw.channel`、`openclaw.provider`、`openclaw.model`）

消息流转：

* `openclaw.webhook.received`（计数器，属性：`openclaw.channel`、
  `openclaw.webhook`）
* `openclaw.webhook.error`（计数器，属性：`openclaw.channel`、
  `openclaw.webhook`）
* `openclaw.webhook.duration_ms`（直方图，属性：`openclaw.channel`、
  `openclaw.webhook`）
* `openclaw.message.queued`（计数器，属性：`openclaw.channel`、
  `openclaw.source`）
* `openclaw.message.processed`（计数器，属性：`openclaw.channel`、
  `openclaw.outcome`）
* `openclaw.message.duration_ms`（直方图，属性：`openclaw.channel`、
  `openclaw.outcome`）

队列和会话：

* `openclaw.queue.lane.enqueue`（计数器，属性：`openclaw.lane`）
* `openclaw.queue.lane.dequeue`（计数器，属性：`openclaw.lane`）
* `openclaw.queue.depth`（直方图，属性：`openclaw.lane` 或
  `openclaw.channel=heartbeat`）
* `openclaw.queue.wait_ms`（直方图，属性：`openclaw.lane`）
* `openclaw.session.state`（计数器，属性：`openclaw.state`、`openclaw.reason`）
* `openclaw.session.stuck`（计数器，属性：`openclaw.state`）
* `openclaw.session.stuck_age_ms`（直方图，属性：`openclaw.state`）
* `openclaw.run.attempt`（计数器，属性：`openclaw.attempt`）

<div id="exported-spans-names-key-attributes">
  ### 导出的 span（名称与关键属性）
</div>

* `openclaw.model.usage`
  * `openclaw.channel`, `openclaw.provider`, `openclaw.model`
  * `openclaw.sessionKey`, `openclaw.sessionId`
  * `openclaw.tokens.*`（输入/输出/cache&#95;read/cache&#95;write/总计）
* `openclaw.webhook.processed`
  * `openclaw.channel`, `openclaw.webhook`, `openclaw.chatId`
* `openclaw.webhook.error`
  * `openclaw.channel`, `openclaw.webhook`, `openclaw.chatId`,
    `openclaw.error`
* `openclaw.message.processed`
  * `openclaw.channel`, `openclaw.outcome`, `openclaw.chatId`,
    `openclaw.messageId`, `openclaw.sessionKey`, `openclaw.sessionId`,
    `openclaw.reason`
* `openclaw.session.stuck`
  * `openclaw.state`, `openclaw.ageMs`, `openclaw.queueDepth`,
    `openclaw.sessionKey`, `openclaw.sessionId`

<div id="sampling-flushing">
  ### 采样与刷新
</div>

* 跟踪采样：`diagnostics.otel.sampleRate`（0.0–1.0，仅根 span）。
* 指标导出间隔：`diagnostics.otel.flushIntervalMs`（最小值 1000ms）。

<div id="protocol-notes">
  ### 协议说明
</div>

* OTLP/HTTP 端点可以通过 `diagnostics.otel.endpoint` 或
  `OTEL_EXPORTER_OTLP_ENDPOINT` 设置。
* 如果端点已包含 `/v1/traces` 或 `/v1/metrics`，则原样使用。
* 如果端点已包含 `/v1/logs`，则在日志中原样使用。
* `diagnostics.otel.logs` 会为主日志记录器的输出启用 OTLP 日志导出。

<div id="log-export-behavior">
  ### 日志导出行为
</div>

* OTLP 日志使用与写入 `logging.file` 相同的结构化记录。
* 遵循 `logging.level`（文件日志级别）。控制台输出脱敏 **不** 适用于 OTLP 日志。
* 高负载部署应优先通过 OTLP 收集器进行采样/过滤。

<div id="troubleshooting-tips">
  ## 故障排查提示
</div>

* **Gateway 无法访问？** 先运行 `openclaw doctor`。
* **日志为空？** 检查 Gateway 是否在运行，并且是否正在写入 `logging.file` 中配置的文件路径所指向的文件。
* **需要更多细节？** 将 `logging.level` 设置为 `debug` 或 `trace` 后重试。