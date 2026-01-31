---
title: 状态
summary: "用于 `openclaw status` 的 CLI 参考（诊断、探针、使用快照）"
read_when:
  - 当你想快速诊断各消息通道的健康状况以及最近会话的接收方时
  - 当你需要一个便于复制粘贴的“all”状态输出用于调试时
---

<div id="openclaw-status">
  # `openclaw status`
</div>

用于通道和会话的诊断。

```bash
openclaw status
openclaw status --all
openclaw status --deep
openclaw status --usage
```

Notes:

* `--deep` 会运行在线探测（WhatsApp Web + Telegram + Discord + Google Chat + Slack + Signal）。
* 当配置了多个智能体时，输出会包含每个智能体的会话存储。
* 概览在可用时会包含 Gateway 和节点宿主服务的安装/运行状态。
* 概览会包含更新通道和 git SHA（用于源码检出）。
* 更新信息会显示在概览中；如果有可用更新，`status` 会输出提示，建议运行 `openclaw update`（参见 [Updating](/zh/install/updating)）。
