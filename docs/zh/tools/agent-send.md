---
title: Agent 代理发送
summary: "直接通过 `openclaw agent` CLI 运行（可选消息发送）"
read_when:
  - 在添加或修改智能体 CLI 入口点时阅读
---

<div id="openclaw-agent-direct-agent-runs">
  # `openclaw agent`（直接智能体运行）
</div>

`openclaw agent` 在无需传入聊天消息的情况下运行智能体的单次对话轮次。
默认情况下它会**通过 Gateway** 运行；添加 `--local` 可强制使用当前机器上的嵌入式运行时。

<div id="behavior">
  ## 行为
</div>

- 必需：`--message <text>`
- 会话选择：
  - `--to <dest>` 推导会话 key（群组/频道目标保持隔离；直接聊天会折叠到 `main`），**或者**
  - `--session-id <id>` 通过 id 复用已有会话，**或者**
  - `--agent <id>` 直接指定一个已配置的智能体（使用该智能体的 `main` 会话 key）
- 运行的嵌入式智能体运行时与正常入站消息回复相同。
- 思考 / verbose 标志位会持久化到会话存储中。
- 输出：
  - 默认：打印回复文本（以及 `MEDIA:<url>` 行）
  - `--json`：打印结构化数据负载和元数据
- 可选地通过 `--deliver` + `--channel` 将结果回送到某个频道（目标格式与 `openclaw message --target` 一致）。
- 使用 `--reply-channel` / `--reply-to` / `--reply-account` 在不更改会话的情况下覆盖投递目标。

如果 Gateway 不可达，CLI **会回退** 到本地嵌入式运行模式。

<div id="examples">
  ## 示例
</div>

```bash
openclaw agent --to +15555550123 --message "status update"
openclaw agent --agent ops --message "Summarize logs"
openclaw agent --session-id 1234 --message "Summarize inbox" --thinking medium
openclaw agent --to +15555550123 --message "Trace logs" --verbose on --json
openclaw agent --to +15555550123 --message "Summon reply" --deliver
openclaw agent --agent ops --message "Generate report" --deliver --reply-channel slack --reply-to "#reports"
```


<div id="flags">
  ## 参数
</div>

- `--local`: 在本地运行（需要在你的 shell 环境中配置模型提供方的 API 密钥）
- `--deliver`: 将回复发送到所选渠道
- `--channel`: 发送渠道（`whatsapp|telegram|discord|googlechat|slack|signal|imessage`，默认为 `whatsapp`）
- `--reply-to`: 覆盖默认发送目标
- `--reply-channel`: 覆盖默认发送渠道
- `--reply-account`: 覆盖默认发送账号 ID
- `--thinking <off|minimal|low|medium|high|xhigh>`: 持久化“思考”级别（仅适用于 GPT-5.2 和 Codex 模型）
- `--verbose <on|full|off>`: 持久化详细输出级别
- `--timeout <seconds>`: 覆盖智能体超时时间
- `--json`: 以结构化 JSON 输出