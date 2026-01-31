---
title: 运行状况
summary: "`openclaw health` 的 CLI 参考（通过 RPC 访问 Gateway 运行状况检查端点）"
read_when:
  - 你想要快速检查正在运行的 Gateway 的运行状况
---

<div id="openclaw-health">
  # `openclaw health`
</div>

获取正在运行的 Gateway 的健康状态。

```bash
openclaw health
openclaw health --json
openclaw health --verbose
```

Notes:

* 使用 `--verbose` 时会运行实时探测，并在配置了多个账户时显示每个账户的耗时。
* 在配置了多个智能体时，输出会包含每个智能体的会话存储。
