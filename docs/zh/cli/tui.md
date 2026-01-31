---
title: Tui 终端界面
summary: "`openclaw tui` 的 CLI 参考（连接 Gateway 的终端 UI）"
read_when:
  - 你希望使用适合远程访问的 Gateway 终端 UI
  - 你希望在脚本中传递 url/token/session 参数
---

<div id="openclaw-tui">
  # `openclaw tui`
</div>

打开与 Gateway 相连的终端 UI。

相关内容：

* TUI 指南：[TUI](/zh/tui)

<div id="examples">
  ## 示例用法
</div>

```bash
openclaw tui
openclaw tui --url ws://127.0.0.1:18789 --token <token>
openclaw tui --session main --deliver
```
