---
title: 出站会话镜像重构（Issue #1520）
description: 用于记录出站会话镜像重构的笔记、决策、测试以及未解决事项。
---

<div id="outbound-session-mirroring-refactor-issue-1520">
  # 出站会话镜像重构（Issue #1520）
</div>

<div id="status">
  ## 状态
</div>

- 进行中。
- 已为出站镜像更新核心与插件通道路由。
- 当省略 sessionKey 时，Gateway send 现在会自动推断目标会话。

<div id="context">
  ## 背景
</div>

出站发送被镜像到了*当前*智能体会话（工具会话键），而不是目标通道会话。入站路由使用通道/对端会话键，因此出站响应被写入到了错误的会话中，而且首次接入的目标往往没有对应的会话条目。

<div id="goals">
  ## 目标
</div>

- 将出站消息映射到目标通道的会话键下。
- 在出站时如缺少对应会话，则创建会话记录。
- 确保线程/主题作用域与入站会话键保持一致。
- 覆盖核心通道以及随附的扩展。

<div id="implementation-summary">
  ## 实现概要
</div>

- 新的出站会话路由辅助函数：
  - `src/infra/outbound/outbound-session.ts`
  - `resolveOutboundSessionRoute` 使用 `buildAgentSessionKey`（dmScope + identityLinks）构建目标 sessionKey。
  - `ensureOutboundSessionEntry` 通过 `recordSessionMetaFromInbound` 写入最小的 `MsgContext`。
- `runMessageAction`（发送）推导目标 sessionKey，并将其传递给 `executeSendAction` 用于镜像。
- `message-tool` 不再直接做镜像；它只从当前会话 key 中解析 agentId。
- 插件发送路径通过使用推导出的 sessionKey 调用 `appendAssistantMessageToSessionTranscript` 来进行镜像。
- Gateway 在未提供目标会话 key 时（默认智能体），会推导出一个目标会话 key，并确保存在对应的会话条目。

<div id="threadtopic-handling">
  ## 线程/话题处理
</div>

- Slack：replyTo/threadId -> `resolveThreadSessionKeys`（后缀）。
- Discord：threadId/replyTo -> 使用 `useSuffix=false` 的 `resolveThreadSessionKeys` 以匹配入站（线程频道 ID 已经限定了会话范围）。
- Telegram：话题 ID 通过 `buildTelegramGroupPeerId` 映射为 `chatId:topic:<id>`。

<div id="extensions-covered">
  ## 覆盖的扩展
</div>

- Matrix、MS Teams、Mattermost、BlueBubbles、Nextcloud Talk、Zalo、Zalo Personal、Nostr、Tlon。
- 注意：
  - Mattermost 目标现在在进行 DM 会话键路由时会去掉 `@`。
  - Zalo Personal 对 1:1 目标使用 DM peer 类型（仅当存在 `group:` 时才视为群组）。
  - BlueBubbles 群组目标会去掉 `chat_*` 前缀，以匹配入站会话键。
  - Slack 自动线程镜像在匹配频道 ID 时不区分大小写。
  - Gateway 的 send 会在镜像前将传入的会话键全部转换为小写。

<div id="decisions">
  ## 决策
</div>

- **Gateway send 会话推导**：如果提供了 `sessionKey`，则直接使用它。如果省略，则从目标 + 默认 Agent 代理推导出一个 `sessionKey`，并在该会话下进行镜像。
- **会话条目创建**：始终使用 `recordSessionMetaFromInbound`，并使 `Provider/From/To/ChatType/AccountId/Originating*` 与入站格式对齐。
- **目标规范化**：出站路由在可用时使用已解析目标（`resolveChannelTarget` 之后的结果）。
- **sessionKey 大小写**：在写入及迁移期间，将 `sessionKey` 规范化为小写。

<div id="tests-addedupdated">
  ## 新增／更新的测试
</div>

- `src/infra/outbound/outbound-session.test.ts`
  - Slack 线程会话 key。
  - Telegram 主题会话 key。
  - 与 Discord 的 dmScope identityLinks。
- `src/agents/tools/message-tool.test.ts`
  - 从会话 key 推导出 agentId（未传递 sessionKey）。
- `src/gateway/server-methods/send.test.ts`
  - 在未提供会话 key 时推导会话 key 并创建会话条目。

<div id="open-items-follow-ups">
  ## 未决事项 / 后续跟进
</div>

- Voice-call 插件使用自定义的 `voice:<phone>` 会话键。此处的出站映射尚未标准化；如果 message-tool 需要支持语音通话的发送，请添加显式映射。
- 确认是否有任何外部插件使用超出内置集合之外的非标准 `From/To` 格式。

<div id="files-touched">
  ## 涉及的文件
</div>

- `src/infra/outbound/outbound-session.ts`
- `src/infra/outbound/outbound-send-service.ts`
- `src/infra/outbound/message-action-runner.ts`
- `src/agents/tools/message-tool.ts`
- `src/gateway/server-methods/send.ts`
- 测试文件：
  - `src/infra/outbound/outbound-session.test.ts`
  - `src/agents/tools/message-tool.test.ts`
  - `src/gateway/server-methods/send.test.ts`