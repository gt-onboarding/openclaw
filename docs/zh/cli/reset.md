---
title: 重置
summary: "`openclaw reset` 的 CLI 参考（重置本地状态/配置）"
read_when:
  - 你想清除本地状态，但保留已安装的 CLI
  - 你想先以 dry-run 方式预览将会被删除的内容
---

<div id="openclaw-reset">
  # `openclaw reset`
</div>

重置本地配置和状态（保留已安装的 CLI）。

```bash
openclaw reset
openclaw reset --dry-run
openclaw reset --scope config+creds+sessions --yes --non-interactive
```
