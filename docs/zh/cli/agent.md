---
title: Agent 代理
summary: "`openclaw agent` 的 CLI 参考（通过 Gateway 发送智能体的一轮对话）"
read_when:
  - 你希望在脚本中运行智能体的一轮对话（可选择发送回复）
---

<div id="openclaw-agent">
  # `openclaw agent`
</div>

通过 Gateway 运行一次智能体会话轮次（嵌入模式请使用 `--local`）。
使用 `--agent <id>` 直接指定一个已配置的智能体作为目标。

相关内容：

* Agent 发送工具：[Agent send](/zh/tools/agent-send)

<div id="examples">
  ## 示例
</div>

```bash
openclaw agent --to +15555550123 --message "status update" --deliver
openclaw agent --agent ops --message "Summarize logs"
openclaw agent --session-id 1234 --message "Summarize inbox" --thinking medium
openclaw agent --agent ops --message "Generate report" --deliver --reply-channel slack --reply-to "#reports"
```
