---
title: 安全
summary: "用于 `openclaw security` 的 CLI 参考（审计并修复常见安全隐患）"
read_when:
  - 你想对配置/状态进行一次快速安全审计
  - 你想应用安全的修复建议（如 chmod、收紧默认配置）
---

<div id="openclaw-security">
  # `openclaw security`
</div>

安全相关工具（审计 + 可选修复）。

相关内容：

* 安全指南：[安全](/zh/gateway/security)

<div id="audit">
  ## 审计
</div>

```bash
openclaw security audit
openclaw security audit --deep
openclaw security audit --fix
```

当多个私信（DM）发送方共享主会话时，此审计会发出警告，并针对共享收件箱推荐将 `session.dmScope` 设置为 `per-channel-peer`（多账号渠道则使用 `per-account-channel-peer`）。

它还会在使用小模型（`<=300B`）、且未启用沙箱并开启了 Web/浏览器工具时发出警告。
