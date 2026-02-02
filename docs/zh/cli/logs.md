---
title: 日志
summary: "`openclaw logs` 的 CLI 参考（通过 RPC 实时跟踪 Gateway 日志）"
read_when:
  - 你需要在无需 SSH 的情况下远程实时跟踪 Gateway 日志
  - 你希望以 JSON 日志行的形式获取数据供工具使用
---

<div id="openclaw-logs">
  # `openclaw logs`
</div>

通过 RPC 实时跟踪 Gateway 文件日志（在远程模式下也可用）。

相关内容：

* 日志概览：[Logging](/zh/logging)

<div id="examples">
  ## 示例用法
</div>

```bash
openclaw logs
openclaw logs --follow
openclaw logs --json
openclaw logs --limit 500
```
