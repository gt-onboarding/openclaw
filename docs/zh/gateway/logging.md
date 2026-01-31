---
title: 日志
summary: "日志界面、文件日志、WS 日志样式和控制台输出格式"
read_when:
  - 更改日志输出或格式
  - 调试 CLI 或 Gateway 输出
---

<div id="logging">
  # 日志
</div>

如需面向用户的整体概览（CLI + Control UI + 配置），请参见 [/logging](/zh/logging)。

OpenClaw 提供两种日志“输出方式”：

* **控制台输出**（你在终端 / Debug UI 中看到的内容）。
* **文件日志**（JSON 行），由 Gateway 日志记录器写入。

<div id="file-based-logger">
  ## 基于文件的日志记录器
</div>

* 默认的滚动日志文件位于 `/tmp/openclaw/`（每天一个文件）：`openclaw-YYYY-MM-DD.log`
  * 日期采用 Gateway 主机的本地时区。
* 日志文件路径和日志级别可以通过 `~/.openclaw/openclaw.json` 配置：
  * `logging.file`
  * `logging.level`

文件格式为每行一个 JSON 对象。

Control UI 的 Logs 选项卡通过 Gateway（`logs.tail`）实时跟踪该文件。
CLI 也可以执行相同操作：

```bash
openclaw logs --follow
```

**详细输出 vs. 日志级别**

* **文件日志** 完全由 `logging.level` 控制。
* `--verbose` 只会影响**控制台输出的详细程度**（以及 WS 日志样式）；它 **不会**
  提升文件日志级别。
* 如果要在文件日志中捕获仅在 verbose 模式下才有的详细信息，请将 `logging.level` 设置为 `debug` 或
  `trace`。

<div id="console-capture">
  ## 控制台捕获
</div>

CLI 会捕获 `console.log/info/warn/error/debug/trace` 并将其写入日志文件，
同时仍会打印到 stdout/stderr。

你可以通过以下配置单独调整控制台输出的详细程度：

* `logging.consoleLevel`（默认值为 `info`）
* `logging.consoleStyle`（`pretty` | `compact` | `json`）

<div id="tool-summary-redaction">
  ## 工具摘要脱敏
</div>

较详细的工具摘要（例如 `🛠️ Exec: ...`）可以在写入控制台输出流之前对敏感 token 进行屏蔽。此功能仅对 **tools** 生效，不会修改文件日志。

* `logging.redactSensitive`: `off` | `tools`（默认：`tools`）
* `logging.redactPatterns`: 正则表达式字符串数组（会覆盖默认规则）
  * 使用原始正则字符串（自动加 `gi`），或者在需要自定义 flags 时使用 `/pattern/flags`。
  * 匹配内容会通过保留前 6 个和后 4 个字符进行掩码处理（长度 &gt;= 18），否则用 `***` 替代。
  * 默认规则覆盖常见的密钥赋值、CLI 参数、JSON 字段、Bearer 头部、PEM 块，以及常见的 token 前缀。

<div id="gateway-websocket-logs">
  ## Gateway WebSocket 日志
</div>

Gateway 会以两种模式打印 WebSocket 协议日志：

* **普通模式（未指定 `--verbose`）**：只打印“关键的” RPC 结果：
  * 错误（`ok=false`）
  * 慢调用（默认阈值：`>= 50ms`）
  * 解析错误
* **详细模式（`--verbose`）**：打印所有 WS 请求/响应数据。

<div id="ws-log-style">
  ### WS 日志样式
</div>

`openclaw gateway` 支持按 Gateway 单独切换日志样式：

* `--ws-log auto`（默认）：在正常模式下进行了优化；在详细模式下使用紧凑输出
* `--ws-log compact`：在详细模式下使用紧凑输出（成对的请求/响应）
* `--ws-log full`：在详细模式下对每个数据帧进行完整输出
* `--compact`：`--ws-log compact` 的别名

示例：

```bash
# optimized (only errors/slow)
openclaw gateway

# show all WS traffic (paired)
openclaw gateway --verbose --ws-log compact

# 显示所有 WS 流量(完整元数据)
openclaw gateway --verbose --ws-log full
```

<div id="console-formatting-subsystem-logging">
  ## 控制台格式化（子系统日志）
</div>

控制台格式化器能够感知 **TTY 环境**，并输出一致且带前缀的日志行。
子系统 logger 会将输出分组，便于快速扫描查看。

行为特性：

* 每一行都有**子系统前缀**（例如 `[gateway]`、`[canvas]`、`[tailscale]`）
* **子系统颜色**（每个子系统使用稳定颜色）加上级别着色
* **仅在输出为 TTY 或环境看起来像功能丰富的终端时使用颜色**（`TERM` / `COLORTERM` / `TERM_PROGRAM`），并遵守 `NO_COLOR`
* **缩短的子系统前缀**：去掉前导的 `gateway/` 和 `channels/`，保留最后 2 段（如 `whatsapp/outbound`）
* **按子系统划分的子 logger**（自动前缀 + 结构化字段 `{ subsystem }`)
* 用于 QR/UX 输出的 **`logRaw()`**（无前缀、无格式化）
* **控制台样式**（例如 `pretty | compact | json`）
* **控制台日志级别**与文件日志级别独立（当 `logging.level` 设为 `debug` / `trace` 时，文件保留完整细节）
* **WhatsApp 消息正文**在 `debug` 级别记录（使用 `--verbose` 查看）

这样既能保持现有文件日志的稳定性，又能让交互式输出更易于扫描阅读。