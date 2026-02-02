---
title: 重试
summary: "向提供方发起的出站调用的重试策略"
read_when:
  - 在更新提供方重试行为或默认配置时
  - 在调试提供方发送错误或限流问题时
---

<div id="retry-policy">
  # 重试策略
</div>

<div id="goals">
  ## 目标
</div>

- 针对每个 HTTP 请求进行重试，而不是对整个多步骤流程进行重试。
- 通过仅重试当前步骤来保持顺序性。
- 避免重复执行非幂等操作。

<div id="defaults">
  ## 默认值
</div>

- 重试次数：3
- 最大延迟上限：30000 ms
- 抖动：0.1（10%）
- 提供方默认设置：
  - Telegram 最小延迟：400 ms
  - Discord 最小延迟：500 ms

<div id="behavior">
  ## 行为
</div>

<div id="discord">
  ### Discord
</div>

- 仅在发生速率限制错误（HTTP 429）时重试。
- 如果提供了 Discord 的 `retry_after` 值则使用它，否则采用指数退避策略。

<div id="telegram">
  ### Telegram
</div>

- 遇到短暂性错误时会重试（429、超时、连接错误/重置/关闭、暂时不可用）。
- 在有 `retry_after` 时使用其指定的间隔，否则使用指数退避策略。
- 对于 Markdown 解析错误不会重试，而是回退为纯文本。

<div id="configuration">
  ## 配置
</div>

在 `~/.openclaw/openclaw.json` 中为每个提供方设置重试策略：

```json5
{
  channels: {
    telegram: {
      retry: {
        attempts: 3,
        minDelayMs: 400,
        maxDelayMs: 30000,
        jitter: 0.1
      }
    },
    discord: {
      retry: {
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1
      }
    }
  }
}
```


<div id="notes">
  ## 注意事项
</div>

- 重试是按每个请求生效的（消息发送、媒体上传、表情回应、投票、贴纸）。
- 复合流程不会重试已完成的步骤。