---
title: 模型配置
summary: "探索：模型配置、认证配置文件以及回退机制"
read_when:
  - 探索未来的模型选择和认证配置文件方案时
---

<div id="model-config-exploration">
  # 模型配置（探索）
</div>

本文档记录的是未来模型配置的一些**设想**。它不是一份
正式的发布规范。关于当前行为，请参阅：

* [模型](/zh/concepts/models)
* [模型故障切换](/zh/concepts/model-failover)
* [OAuth 与配置文件](/zh/concepts/oauth)

<div id="motivation">
  ## 动机
</div>

运维人员希望：

* 每个提供方支持多个认证配置文件（个人和工作）。
* 简单的 `/model` 选择方式，并具有可预测的回退行为。
* 在纯文本模型与支持图像的模型之间有清晰的区分。

<div id="possible-direction-high-level">
  ## 可能的高层方向
</div>

* 保持模型选择简单：使用 `provider/model`，可选别名。
* 让提供方支持多个认证配置，并具有明确的优先顺序。
* 使用全局回退列表，使所有会话的故障切换行为保持一致。
* 仅在显式配置时才覆盖图像路由。

<div id="open-questions">
  ## 未决问题
</div>

* 配置档轮换应该按提供方还是按模型进行？
* UI 应该如何呈现会话的配置档选择？
* 从遗留配置键迁移的最安全路径是什么？