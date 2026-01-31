---
title: Dns
summary: "`openclaw dns` 的 CLI 参考（广域发现辅助命令）"
read_when:
  - 你希望通过 Tailscale + CoreDNS 实现广域发现（DNS-SD）
  - 你正在为自定义发现域（例如：openclaw.internal）配置 Split DNS（拆分 DNS）
---

<div id="openclaw-dns">
  # `openclaw dns`
</div>

用于广域发现的 DNS 辅助工具（Tailscale + CoreDNS）。目前主要针对 macOS + Homebrew CoreDNS。

相关内容：

* Gateway 发现：[发现](/zh/gateway/discovery)
* 广域发现配置：[配置](/zh/gateway/configuration)

<div id="setup">
  ## 配置
</div>

```bash
openclaw dns setup
openclaw dns setup --apply
```
