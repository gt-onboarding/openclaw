---
title: 位置
summary: "入站渠道位置解析（Telegram + WhatsApp）及上下文字段"
read_when:
  - 添加或修改渠道位置解析
  - 在 Agent 代理提示词或工具中使用位置上下文字段
---

<div id="channel-location-parsing">
  # 频道位置解析
</div>

OpenClaw 会将来自聊天频道的共享位置规范化为：

- 附加到入站消息正文的人类可读文本，及
- 自动回复上下文负载中的结构化字段。

当前支持：

- **Telegram**（位置图钉 + 地点 + 实时位置）
- **WhatsApp**（`locationMessage` + `liveLocationMessage`）
- **Matrix**（带有 `geo_uri` 的 `m.location`）

<div id="text-formatting">
  ## 文本格式
</div>

位置会被渲染为不带括号的用户友好型单行文本：

* 位置标记：
  * `📍 48.858844, 2.294351 ±12m`
* 命名地点：
  * `📍 Eiffel Tower — Champ de Mars, Paris (48.858844, 2.294351 ±12m)`
* 实时共享位置：
  * `🛰 Live location: 48.858844, 2.294351 ±12m`

如果渠道中包含说明文字/评论，则会附加在下一行：

```
📍 48.858844, 2.294351 ±12m
在此集合
```


<div id="context-fields">
  ## 上下文字段
</div>

当包含位置信息时，这些字段会被添加到 `ctx`：

- `LocationLat`（数值）
- `LocationLon`（数值）
- `LocationAccuracy`（数值，单位：米；可选）
- `LocationName`（字符串；可选）
- `LocationAddress`（字符串；可选）
- `LocationSource`（`pin | place | live`）
- `LocationIsLive`（布尔值）

<div id="channel-notes">
  ## 频道说明
</div>

- **Telegram**：地点会映射到 `LocationName/LocationAddress`；实时位置使用 `live_period`。
- **WhatsApp**：`locationMessage.comment` 和 `liveLocationMessage.caption` 会附加为说明行。
- **Matrix**：`geo_uri` 会被解析为一个标注位置；高度会被忽略，且 `LocationIsLive` 始终为 false。