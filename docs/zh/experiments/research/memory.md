---
title: 记忆
summary: "研究笔记：用于 Clawd 工作区的离线记忆系统（以 Markdown 为唯一可信来源 + 派生索引）"
read_when:
  - 在日常 Markdown 日志之外设计工作区记忆系统（~/.openclaw/workspace）
  - 在独立 CLI 与深度集成 OpenClaw 之间做出选择
  - 添加离线保留、回忆与反思能力（retain/recall/reflect）
---

<div id="workspace-memory-v2-offline-research-notes">
  # 工作区记忆 v2（离线）：研究笔记
</div>

目标：Clawd 风格的工作区（`agents.defaults.workspace`，默认路径 `~/.openclaw/workspace`），其中 “memory” 以每天一个 Markdown 文件的形式存储（`memory/YYYY-MM-DD.md`），再配合一小组稳定文件（例如 `memory.md`、`SOUL.md`）。

本文档提出了一种**离线优先**的记忆架构，将 Markdown 作为规范且可审阅的权威事实来源，同时通过派生索引增加**结构化回忆**（搜索、实体摘要、置信度更新）。

<div id="why-change">
  ## 为什么要变更？
</div>

当前的方案（每天一个文件）非常适合：

- “仅追加式”的日记记录
- 人工手动编辑
- 基于 git 的持久化与可审计性
- 低阻力记录（“想到就写下来”）

但在以下方面较弱：

- 高召回的检索（“我们当时对 X 做了什么决定？”，“上次我们尝试 Y 时怎么样？”）
- 以实体为中心的回答（“跟我说说 Alice / The Castle / warelay”）而无需重读大量文件
- 观点/偏好的稳定性（以及其变化时的证据）
- 时间范围约束（“在 2025 年 11 月期间哪些是真的？”）和冲突解决

<div id="design-goals">
  ## 设计目标
</div>

- **离线**：在无网络情况下工作；可在笔记本电脑/Castle 上运行；不依赖云端。
- **可解释**：检索到的条目应当可归因（文件 + 位置），并且与推理过程解耦。
- **低门槛**：日常记录保持为 Markdown，无需繁重的 Schema 设计工作。
- **可增量演进**：仅使用 FTS 的 v1 就足够好用；语义/向量检索和图结构是可选升级。
- **对智能体友好**：便于在 token 限额内“回忆”（返回体量较小的事实集合）。

<div id="north-star-model-hindsight-letta">
  ## 北极星模型（Hindsight × Letta）
</div>

要融合的两部分：

1) **Letta/MemGPT 风格的控制循环**

- 在上下文中始终保留一个小的「核心」上下文（人格设定 + 关键用户事实）
- 其他内容都在上下文之外，通过工具检索
- 记忆写入通过显式的工具调用完成（append/replace/insert），持久化后在下一轮重新注入

2) **Hindsight 风格的底层记忆结构**

- 区分「被观察到的」「被相信的」和「被总结的」
- 支持 retain/recall/reflect
- 具有置信度的观点，可随证据演化
- 具备实体感知的检索 + 时间查询（即使没有完整的知识图谱）

<div id="proposed-architecture-markdown-source-of-truth-derived-index">
  ## 建议架构（以 Markdown 作为单一事实来源 + 派生索引）
</div>

<div id="canonical-store-git-friendly">
  ### 规范存储（适合 git）
</div>

将 `~/.openclaw/workspace` 作为规范的人类可读记忆源。

建议的工作区布局：

```
~/.openclaw/workspace/
  memory.md                    # small: durable facts + preferences (core-ish)
  memory/
    YYYY-MM-DD.md              # daily log (append; narrative)
  bank/                        # “typed” memory pages (stable, reviewable)
    world.md                   # objective facts about the world
    experience.md              # what the agent did (first-person)
    opinions.md                # 主观偏好/判断 + 置信度 + 证据指针
    entities/
      Peter.md
      The-Castle.md
      warelay.md
      ...
```

Notes:

* **每日日志就保持为每日日志**。不需要把它转换成 JSON。
* `bank/` 文件是经过**精心整理**的，由反思任务生成，但仍然可以手动编辑。
* `memory.md` 仍然保持“体量小 + 偏核心”：也就是你希望 Clawd 在每个会话中都能看到的内容。


<div id="derived-store-machine-recall">
  ### 派生存储（机器检索）
</div>

在工作区下添加一个派生索引（不一定需要纳入 git 版本控制）：

```
~/.openclaw/workspace/.memory/index.sqlite
```

底层实现依赖于：

* 用于事实 + 实体链接 + 观点元数据的 SQLite 架构（schema）
* 用于词汇级召回（快速、小巧、离线）的 SQLite **FTS5**
* 用于语义召回（同样离线）的可选 embeddings 表

该索引始终可以**从 Markdown 重新构建**。


<div id="retain-recall-reflect-operational-loop">
  ## 留存 / 召回 / 反思（运行循环）
</div>

<div id="retain-normalize-daily-logs-into-facts">
  ### 保留：将每日日志标准化为“事实”
</div>

Hindsight 在这里最重要的洞见是：存储**具备叙事性的、自洽完整的事实**，而不是细小的片段。

针对 `memory/YYYY-MM-DD.md` 的实用规则：

* 在一天结束时（或过程中），添加一个 `## Retain` 小节，其中包含 2–5 条要点，这些要点应：
  * 具备叙事性（保留跨轮次上下文）
  * 自洽完整（单独拿出来，之后看也说得通）
  * 带有类型标签 + 实体标注

示例：

```
## 保留
- W @Peter: 目前在马拉喀什（2025年11月27日至12月1日）参加Andy的生日派对。
- B @warelay: 我通过在try/catch中包装connection.update处理程序修复了Baileys WS崩溃问题（参见memory/2025-11-27.md）。
- O(c=0.95) @Peter: 在WhatsApp上偏好简洁回复（<1500字符）；长内容放入文件中。
```

最简解析：

* 类型前缀：`W`（world/世界知识），`B`（experience/biographical/经历・个人背景），`O`（opinion/观点），`S`（observation/summary；通常为生成内容）
* 实体：`@Peter`、`@warelay` 等（slug 映射到 `bank/entities/*.md`）
* 观点置信度：`O(c=0.0..1.0)`（可选）

如果你不希望作者为此操心：`reflect` 任务可以从其余日志中推断出这些要点，但显式写一个 `## Retain` 小节是最简单的“质量调节手段”。


<div id="recall-queries-over-the-derived-index">
  ### 回忆：基于派生索引的查询
</div>

回忆应支持：

- **词汇**： “查找精确术语 / 名称 / 命令”（FTS5）
- **实体**： “告诉我关于 X 的信息”（实体页面 + 与实体关联的事实）
- **时间**： “11 月 27 日前后发生了什么” / “自上周以来”
- **意见/偏好**： “Peter 更偏好什么？”（附置信度 + 证据）

返回格式应便于智能体使用，并标注来源：

- `kind`（`world|experience|opinion|observation`）
- `timestamp`（来源日期，或在存在时间信息时为抽取出的时间范围）
- `entities`（`["Peter","warelay"]`）
- `content`（叙述性事实）
- `source`（`memory/2025-11-27.md#L12` 等）

<div id="reflect-produce-stable-pages-update-beliefs">
  ### 反思：生成稳定页面并更新信念
</div>

反思是一个定时任务（每日或心跳 `ultrathink`），它会：

- 根据最近的事实更新 `bank/entities/*.md`（实体摘要）
- 基于强化/反驳更新 `bank/opinions.md` 中的置信度
- 可选地为 `memory.md`（接近“核心”的持久事实）提出修改建议

观点演化（简单、可解释）：

- 每个观点包含：
  - 陈述
  - 置信度 `c ∈ [0,1]`
  - last_updated
  - 证据链接（支持和反驳的事实 ID）
- 当有新事实出现时：
  - 根据实体重叠和相似度查找候选观点（先用 FTS，之后用 embeddings）
  - 通过小幅增量更新置信度；大的置信度跃迁需要强烈的反驳和反复出现的证据

<div id="cli-integration-standalone-vs-deep-integration">
  ## CLI 集成：独立模式 vs 深度集成
</div>

建议：**与 OpenClaw 深度集成**，但保留可分离的核心库。

<div id="why-integrate-into-openclaw">
  ### 为什么要集成进 OpenClaw？
</div>

- OpenClaw 已经知道：
  - 工作区路径（`agents.defaults.workspace`）
  - 会话模型 + 心跳机制
  - 日志记录 + 故障排查模式
- 你希望由智能体本身来调用工具：
  - `openclaw memory recall "…" --k 25 --since 30d`
  - `openclaw memory reflect --since 7d`

<div id="why-still-split-a-library">
  ### 为什么还要单独拆分成一个库？
</div>

- 让 memory 逻辑在没有 Gateway/运行时的情况下也能进行测试
- 便于在其他场景中复用（本地脚本、未来的桌面应用等）

形态：
memory 工具计划做成一个小型的 CLI + 库层，目前只是探索性设计。

<div id="s-collide-suco-when-to-use-it-research">
  ## “S-Collide” / SuCo：何时使用（研究）
</div>

如果 “S-Collide” 指的是 **SuCo (Subspace Collision)**：这是一种近似最近邻（ANN）检索方法，通过在子空间中使用学习 / 结构化碰撞来优化召回率 / 延迟之间的权衡（论文：arXiv 2411.14754, 2024）。

针对 `~/.openclaw/workspace` 的务实建议：

- **不要一上来** 就用 SuCo。
- 先用 SQLite FTS +（可选）简单 embeddings；你会立刻拿到大部分体验提升。
- 只在以下情况才考虑 SuCo/HNSW/ScaNN 这类方案：
  - 语料库规模很大（数万 / 数十万 chunks）
  - 暴力 embedding 搜索变得太慢
  - 召回质量明显被纯词法搜索卡住

离线友好型替代方案（按复杂度升序）：

- SQLite FTS5 + 元数据过滤（零 ML）
- Embeddings + 暴力搜索（如果 chunk 数量较少，效果出乎意料地好）
- HNSW 索引（常见、稳健；需要库绑定）
- SuCo（研究级；如果有可以嵌入的可靠实现，就很有吸引力）

开放问题：

- 在你的机器上（笔记本 + 台式机），用于“个人助手记忆”的**最佳**离线 embedding 模型是什么？
  - 如果你已经有 Ollama：用本地模型做 embed；否则就在工具链里附带一个小型 embedding 模型。

<div id="smallest-useful-pilot">
  ## 最小有用试点
</div>

如果你想要一个最小但仍然实用的版本：

- 在 `bank/` 中添加实体页面，并在每日日志中添加一个 `## Retain` 小节。
- 使用 SQLite FTS 做带引用的检索（路径 + 行号）。
- 只有在召回质量或规模有要求时才添加向量嵌入（embeddings）。

<div id="references">
  ## 参考资料
</div>

- Letta / MemGPT 概念：“core memory blocks（核心记忆块）” + “archival memory（档案式记忆）” + 工具驱动的自编辑记忆。
- Hindsight 技术报告：“retain / recall / reflect（保留 / 回忆 / 反思）”、四网络记忆架构、叙事事实抽取、观点置信度演变。
- SuCo：arXiv 2411.14754 (2024)：“Subspace Collision（子空间碰撞）”近似最近邻检索。