---
title: 群组策略加固
summary: "Telegram 允许列表加固：前缀与空白字符规范化"
read_when:
  - 查看历史 Telegram 允许列表变更
---

<div id="telegram-allowlist-hardening">
  # Telegram 允许列表强化
</div>

**日期**: 2026-01-05\
**状态**: 已完成\
**PR**: #216

<div id="summary">
  ## 摘要
</div>

Telegram 允许列表现在可以不区分大小写地接受 `telegram:` 和 `tg:` 前缀，并且容忍
意外的空白字符。这样使入站允许列表检查与出站发送规范化保持一致。

<div id="what-changed">
  ## 有哪些变更
</div>

* 前缀 `telegram:` 和 `tg:` 被视为等价（不区分大小写）。
* 允许列表中的条目会被去除首尾空白字符；空条目会被忽略。

<div id="examples">
  ## 示例
</div>

以下所有写法都会被视为同一个 ID：

* `telegram:123456`
* `TG:123456`
* `tg:123456`

<div id="why-it-matters">
  ## 为什么这很重要
</div>

从日志或聊天 ID 中复制/粘贴时，往往会带有前缀和多余的空白。进行标准化处理可以避免在判断是否要在私聊或群组中回复时出现漏判。

<div id="related-docs">
  ## 相关文档
</div>

* [群聊](/zh/concepts/groups)
* [Telegram 提供方](/zh/channels/telegram)