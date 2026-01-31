---
title: 上手引导
summary: "用于 `openclaw onboard` 的 CLI 参考（交互式上手引导向导）"
read_when:
  - 当你希望通过引导式流程完成 Gateway、工作区、身份验证、渠道和技能的配置时
---

<div id="openclaw-onboard">
  # `openclaw onboard`
</div>

交互式入门向导（用于本地或远程 Gateway 的设置）。

相关内容：

* 向导指南：[Onboarding](/zh/start/onboarding)

<div id="examples">
  ## 示例
</div>

```bash
openclaw onboard
openclaw onboard --flow quickstart
openclaw onboard --flow manual
openclaw onboard --mode remote --remote-url ws://gateway-host:18789
```

流程说明：

* `quickstart`：最少交互步骤，自动生成 Gateway 令牌。
* `manual`：提供完整交互步骤以配置端口 / 绑定地址 / 认证（`advanced` 的别名）。
* 最快开始首次对话的方式：运行 `openclaw dashboard`（使用 Control UI，无需配置通道）。
