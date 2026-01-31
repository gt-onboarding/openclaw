---
title: 语音通话
summary: "语音通话插件：通过 Twilio/Telnyx/Plivo 实现呼出和呼入通话（插件安装、配置与 CLI）"
read_when:
  - 你希望从 OpenClaw 发起外呼语音通话
  - 你正在配置或开发 Voice Call 插件
---

<div id="voice-call-plugin">
  # 语音通话（插件）
</div>

通过插件为 OpenClaw 提供语音通话功能。支持外呼通知以及结合来电策略的多轮对话。

当前提供方：

- `twilio`（Programmable Voice + Media Streams）
- `telnyx`（Call Control v2）
- `plivo`（Voice API + XML transfer + GetInput speech）
- `mock`（开发用/无网络）

快速心智模型：

- 安装插件
- 重启 Gateway
- 在 `plugins.entries.voice-call.config` 下进行配置
- 使用 `openclaw voicecall ...` 或 `voice_call` 工具

<div id="where-it-runs-local-vs-remote">
  ## 运行位置（本地 vs 远程）
</div>

Voice Call 插件运行在 **Gateway 进程内**。

如果你使用远程 Gateway，请在 **运行 Gateway 的那台机器** 上安装并配置该插件，然后重启 Gateway 以加载该插件。

<div id="install">
  ## 安装
</div>

<div id="option-a-install-from-npm-recommended">
  ### 选项 A：使用 npm 安装（推荐）
</div>

```bash
openclaw plugins install @openclaw/voice-call
```

完成后重启 Gateway。


<div id="option-b-install-from-a-local-folder-dev-no-copying">
  ### 选项 B：从本地文件夹安装（开发用，无需复制）
</div>

```bash
openclaw plugins install ./extensions/voice-call
cd ./extensions/voice-call && pnpm install
```

然后重启 Gateway。


<div id="config">
  ## 配置
</div>

在 `plugins.entries.voice-call.config` 下进行配置：

```json5
{
  plugins: {
    entries: {
      "voice-call": {
        enabled: true,
        config: {
          provider: "twilio", // 或 "telnyx" | "plivo" | "mock"
          fromNumber: "+15550001234",
          toNumber: "+15550005678",

          twilio: {
            accountSid: "ACxxxxxxxx",
            authToken: "..."
          },

          plivo: {
            authId: "MAxxxxxxxxxxxxxxxxxxxx",
            authToken: "..."
          },

          // Webhook server
          serve: {
            port: 3334,
            path: "/voice/webhook"
          },

          // Public exposure (pick one)
          // publicUrl: "https://example.ngrok.app/voice/webhook",
          // tunnel: { provider: "ngrok" },
          // tailscale: { mode: "funnel", path: "/voice/webhook" }

          outbound: {
            defaultMode: "notify" // notify | conversation
          },

          streaming: {
            enabled: true,
            streamPath: "/voice/stream"
          }
        }
      }
    }
  }
}
```

Notes:

* Twilio/Telnyx 需要一个**可从公网访问**的 webhook URL。
* Plivo 需要一个**可从公网访问**的 webhook URL。
* `mock` 是一个本地开发提供方（无网络调用）。
* `skipSignatureVerification` 仅用于本地测试。
* 如果你使用 ngrok 免费套餐，将 `publicUrl` 设置为与实际 ngrok URL 完全一致；签名校验始终会被强制执行。
* `tunnel.allowNgrokFreeTierLoopbackBypass: true` 只在 `tunnel.provider="ngrok"` 且 `serve.bind` 为 loopback（ngrok 本地 Agent 代理）时，才会允许签名无效的 Twilio webhook。仅用于本地开发。
* ngrok 免费套餐的 URL 可能会变化或引入中间过渡页面行为；如果 `publicUrl` 与之不再匹配，Twilio 签名将会失败。生产环境中应优先使用稳定域名或 Tailscale funnel。


<div id="tts-for-calls">
  ## 通话的 TTS
</div>

Voice Call 插件在通话中进行流式语音合成时，会使用核心 `messages.tts` 配置（OpenAI 或 ElevenLabs）。你可以在插件配置中使用**相同结构**进行重写；它会与 `messages.tts` 进行深度合并。

```json5
{
  tts: {
    provider: "elevenlabs",
    elevenlabs: {
      voiceId: "pMsXgVXv3BLzUgSXRplE",
      modelId: "eleven_multilingual_v2"
    }
  }
}
```

Notes:

* **在语音通话中会忽略 Edge TTS**（电话语音需要 PCM；Edge 的输出不够稳定）。
* 启用 Twilio 媒体流时会使用核心 TTS；否则通话会回退为使用提供方的原生语音。


<div id="more-examples">
  ### 更多示例
</div>

仅使用核心 TTS（不使用任何 override 配置）：

```json5
{
  messages: {
    tts: {
      provider: "openai",
      openai: { voice: "alloy" }
    }
  }
}
```

仅在通话中改用 ElevenLabs（其他地方继续使用核心默认配置）：

```json5
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          tts: {
            provider: "elevenlabs",
            elevenlabs: {
              apiKey: "elevenlabs_key",
              voiceId: "pMsXgVXv3BLzUgSXRplE",
              modelId: "eleven_multilingual_v2"
            }
          }
        }
      }
    }
  }
}
```

仅覆盖用于呼叫的 OpenAI 模型（深度合并示例）：

```json5
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          tts: {
            openai: {
              model: "gpt-4o-mini-tts",
              voice: "marin"
            }
          }
        }
      }
    }
  }
}
```


<div id="inbound-calls">
  ## 入站呼叫
</div>

入站策略默认为 `disabled`。要启用入站呼叫，请设置：

```json5
{
  inboundPolicy: "allowlist",
  allowFrom: ["+15550001234"],
  inboundGreeting: "您好!有什么可以帮您?"
}
```

自动响应基于智能体系统。可以通过以下参数进行调优：

* `responseModel`
* `responseSystemPrompt`
* `responseTimeoutMs`


<div id="cli">
  ## CLI
</div>

```bash
openclaw voicecall call --to "+15555550123" --message "Hello from OpenClaw"
openclaw voicecall continue --call-id <id> --message "Any questions?"
openclaw voicecall speak --call-id <id> --message "One moment"
openclaw voicecall end --call-id <id>
openclaw voicecall status --call-id <id>
openclaw voicecall tail
openclaw voicecall expose --mode funnel
```


<div id="agent-tool">
  ## Agent 代理工具
</div>

工具名称：`voice_call`

操作：

- `initiate_call` (message, to?, mode?)
- `continue_call` (callId, message)
- `speak_to_user` (callId, message)
- `end_call` (callId)
- `get_status` (callId)

此仓库在 `skills/voice-call/SKILL.md` 中附带了配套的技能文档。

<div id="gateway-rpc">
  ## Gateway RPC 接口
</div>

- `voicecall.initiate` (`to?`, `message`, `mode?`)
- `voicecall.continue` (`callId`, `message`)
- `voicecall.speak` (`callId`, `message`)
- `voicecall.end` (`callId`)
- `voicecall.status` (`callId`)