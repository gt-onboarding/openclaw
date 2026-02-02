---
title: 配置
summary: "`openclaw configure` 的 CLI 参考（交互式配置向导）"
read_when:
  - 你想通过交互方式调整凭证、设备或智能体默认配置时
---

<div id="openclaw-configure">
  # `openclaw configure`
</div>

用于交互式配置凭证、设备以及 Agent 默认设置的向导。

注意：**Model** 部分现在包含一个用于 `agents.defaults.models` 允许列表的多选控件（决定哪些模型会出现在 `/model` 和模型选择器中）。

提示：不带子命令的 `openclaw config` 会打开同一个向导。使用
`openclaw config get|set|unset` 进行非交互式编辑。

相关内容：

* Gateway 配置参考：[Configuration](/zh/gateway/configuration)
* 配置 CLI：[Config](/zh/cli/config)

注意事项：

* 选择 Gateway 运行的位置时总会更新 `gateway.mode`。如果你只需要这一项，可以在不修改其他部分的情况下直接选择 “Continue”。
* 面向频道的服务（Slack/Discord/Matrix/Microsoft Teams）会在设置期间提示配置频道/房间允许列表。你可以输入名称或 ID；向导会在可能的情况下将名称解析为 ID。

<div id="examples">
  ## 示例
</div>

```bash
openclaw configure
openclaw configure --section models --section channels
```
