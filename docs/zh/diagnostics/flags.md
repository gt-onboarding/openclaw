---
title: 标志位
summary: "用于定向调试日志的诊断标志位"
read_when:
  - 你需要在不提高全局日志级别的情况下记录定向调试日志
  - 你需要捕获特定子系统的日志以便支持和排查问题
---

<div id="diagnostics-flags">
  # 诊断标志
</div>

诊断标志用于在不启用全局详细日志的前提下，开启有针对性的调试日志。标志为按需生效，只有在某个子系统显式检查它们时才会起作用。

<div id="how-it-works">
  ## 工作原理
</div>

* 标志位是字符串（不区分大小写）。
* 你可以在配置中启用标志位，或者通过环境变量覆盖配置来启用。
* 支持通配符：
  * `telegram.*` 匹配 `telegram.http`
  * `*` 启用所有标志位

<div id="enable-via-config">
  ## 通过配置启用
</div>

```json
{
  "diagnostics": {
    "flags": ["telegram.http"]
  }
}
```

多个标志位：

```json
{
  "diagnostics": {
    "flags": ["telegram.http", "gateway.*"]
  }
}
```

在修改这些标志后，请重启 Gateway。

<div id="env-override-one-off">
  ## 环境变量覆盖（单次）
</div>

```bash
OPENCLAW_DIAGNOSTICS=telegram.http,telegram.payload
```

禁用所有标志位：

```bash
OPENCLAW_DIAGNOSTICS=0
```

<div id="where-logs-go">
  ## 日志存储位置
</div>

这些标志会将日志输出到标准诊断日志文件中。默认情况下：

```
/tmp/openclaw/openclaw-YYYY-MM-DD.log
```

如果你设置了 `logging.file`，请改用该路径。日志为 JSONL 格式（每行一个 JSON 对象）。仍会根据 `logging.redactSensitive` 进行脱敏处理。

<div id="extract-logs">
  ## 导出日志
</div>

选择最新的日志文件：

```bash
ls -t /tmp/openclaw/openclaw-*.log | head -n 1
```

Telegram HTTP 诊断过滤器：

```bash
rg "telegram http error" /tmp/openclaw/openclaw-*.log
```

或者在重现问题时进行 tail：

```bash
tail -f /tmp/openclaw/openclaw-$(date +%F).log | rg "telegram http error"
```

对于远程 Gateway，你同样可以使用 `openclaw logs --follow`（参见 [/cli/logs](/zh/cli/logs)）。

<div id="notes">
  ## 注意
</div>

* 如果将 `logging.level` 设置为高于 `warn`，这些日志可能不会输出。默认值 `info` 即可。
* 保持这些标志启用是安全的；它们只会影响对应子系统的日志输出量。
* 使用 [/logging](/zh/logging) 来更改日志输出目标、级别和脱敏设置。