---
title: 文本转语音（TTS）
summary: "用于向外发送回复的文本转语音（TTS）"
read_when:
  - 为回复启用文本转语音
  - 配置 TTS 提供方或相关限制
  - 使用 /tts 命令
---

<div id="text-to-speech-tts">
  # 文本转语音（TTS）
</div>

OpenClaw 可以使用 ElevenLabs、OpenAI 或 Edge TTS 将发出的回复转换为音频。
在任何 OpenClaw 支持发送音频的地方都可用；在 Telegram 中会显示为圆形的语音消息气泡。

<div id="supported-services">
  ## 支持的服务
</div>

* **ElevenLabs**（主要或备用提供方）
* **OpenAI**（主要或备用提供方；也用于生成摘要）
* **Edge TTS**（主要或备用提供方；使用 `node-edge-tts`，在未配置 API 密钥时为默认选项）

<div id="edge-tts-notes">
  ### Edge TTS 说明
</div>

Edge TTS 通过 `node-edge-tts` 库使用 Microsoft Edge 的在线神经网络 TTS 服务。它是托管服务（非本地），使用 Microsoft 的端点，并且不需要 API 密钥。`node-edge-tts` 提供了语音配置选项和输出格式，但并非所有选项都受 Edge 服务支持。 citeturn2search0

由于 Edge TTS 是一个没有公布 SLA 或配额的公共网络服务，应将其视为尽力而为（best-effort）服务。如果你需要有明确保障的限额和支持，请使用 OpenAI 或 ElevenLabs。Microsoft 的 Speech REST API 文档指出，每个请求的音频时长上限为 10 分钟；Edge TTS 未公布相关限制，因此应假定其限制相同或更低。 citeturn0search3

<div id="optional-keys">
  ## 可选键
</div>

如果你想使用 OpenAI 或 ElevenLabs：

* `ELEVENLABS_API_KEY`（或 `XI_API_KEY`）
* `OPENAI_API_KEY`

Edge TTS **不**需要 API 密钥。如果没有找到任何 API 密钥，OpenClaw 会默认
使用 Edge TTS（除非通过 `messages.tts.edge.enabled=false` 禁用）。

如果配置了多个提供方，会优先使用当前选定的提供方，其他提供方作为回退选项。
自动摘要使用配置的 `summaryModel`（或 `agents.defaults.model.primary`），
因此如果你启用了摘要，该提供方也必须通过鉴权。

<div id="service-links">
  ## 服务相关链接
</div>

* [OpenAI 文本转语音指南](https://platform.openai.com/docs/guides/text-to-speech)
* [OpenAI Audio API 参考文档](https://platform.openai.com/docs/api-reference/audio)
* [ElevenLabs 文本转语音](https://elevenlabs.io/docs/api-reference/text-to-speech)
* [ElevenLabs 认证](https://elevenlabs.io/docs/api-reference/authentication)
* [node-edge-tts](https://github.com/SchneeHertz/node-edge-tts)
* [Microsoft 语音输出格式](https://learn.microsoft.com/azure/ai-services/speech-service/rest-text-to-speech#audio-outputs)

<div id="is-it-enabled-by-default">
  ## 默认是启用的吗？
</div>

不是。自动 TTS 默认是**关闭**的。你可以在配置中通过
`messages.tts.auto` 将其启用，或者在单个会话中使用 `/tts always`（别名：`/tts on`）启用。

一旦开启了 TTS，Edge TTS **默认会被启用**，并会在没有可用的 OpenAI 或 ElevenLabs API 密钥时自动使用。

<div id="config">
  ## 配置
</div>

TTS 配置位于 `openclaw.json` 文件中的 `messages.tts` 字段下。
完整的配置 schema 见 [Gateway configuration](/zh/gateway/configuration)。

<div id="minimal-config-enable-provider">
  ### 最小配置（启用 + 提供方）
</div>

```json5
{
  messages: {
    tts: {
      auto: "always",
      provider: "elevenlabs"
    }
  }
}
```

<div id="openai-primary-with-elevenlabs-fallback">
  ### 以 OpenAI 为主，ElevenLabs 为备用
</div>

```json5
{
  messages: {
    tts: {
      auto: "always",
      provider: "openai",
      summaryModel: "openai/gpt-4.1-mini",
      modelOverrides: {
        enabled: true
      },
      openai: {
        apiKey: "openai_api_key",
        model: "gpt-4o-mini-tts",
        voice: "alloy"
      },
      elevenlabs: {
        apiKey: "elevenlabs_api_key",
        baseUrl: "https://api.elevenlabs.io",
        voiceId: "voice_id",
        modelId: "eleven_multilingual_v2",
        seed: 42,
        applyTextNormalization: "auto",
        languageCode: "en",
        voiceSettings: {
          stability: 0.5,
          similarityBoost: 0.75,
          style: 0.0,
          useSpeakerBoost: true,
          speed: 1.0
        }
      }
    }
  }
}
```

<div id="edge-tts-primary-no-api-key">
  ### Edge TTS 主通道（无需 API 密钥）
</div>

```json5
{
  messages: {
    tts: {
      auto: "always",
      provider: "edge",
      edge: {
        enabled: true,
        voice: "en-US-MichelleNeural",
        lang: "en-US",
        outputFormat: "audio-24khz-48kbitrate-mono-mp3",
        rate: "+10%",
        pitch: "-5%"
      }
    }
  }
}
```

<div id="disable-edge-tts">
  ### 禁用 Edge 语音合成
</div>

```json5
{
  messages: {
    tts: {
      edge: {
        enabled: false
      }
    }
  }
}
```

<div id="custom-limits-prefs-path">
  ### 自定义限制和偏好设置路径
</div>

```json5
{
  messages: {
    tts: {
      auto: "always",
      maxTextLength: 4000,
      timeoutMs: 30000,
      prefsPath: "~/.openclaw/settings/tts.json"
    }
  }
}
```

<div id="only-reply-with-audio-after-an-inbound-voice-note">
  ### 仅在收到语音消息后才以音频回复
</div>

```json5
{
  messages: {
    tts: {
      auto: "inbound"
    }
  }
}
```

<div id="disable-auto-summary-for-long-replies">
  ### 对长回复禁用自动摘要
</div>

```json5
{
  messages: {
    tts: {
      auto: "always"
    }
  }
}
```

然后执行：

```
/tts summary off
```

<div id="notes-on-fields">
  ### 字段说明
</div>

* `auto`: 自动 TTS 模式（`off`、`always`、`inbound`、`tagged`）。
  * `inbound` 只会在收到语音消息后发送音频。
  * `tagged` 只会在回复中包含 `[[tts]]` 标签时发送音频。
* `enabled`: 旧版开关（`doctor` 会将其迁移为 `auto`）。
* `mode`: `"final"`（默认）或 `"all"`（包含工具/块级回复）。
* `provider`: `"elevenlabs"`、`"openai"` 或 `"edge"`（回退自动进行）。
* 如果 `provider` **未设置**，OpenClaw 会优先选择 `openai`（若存在密钥），然后是 `elevenlabs`（若存在密钥），
  否则为 `edge`。
* `summaryModel`: 用于自动摘要的可选低成本模型；默认为 `agents.defaults.model.primary`。
  * 接受 `provider/model` 或已配置的模型别名。
* `modelOverrides`: 允许模型发出 TTS 指令（默认开启）。
* `maxTextLength`: TTS 输入的硬上限（字符数）。如果超出，`/tts audio` 会失败。
* `timeoutMs`: 请求超时时间（毫秒）。
* `prefsPath`: 覆盖本地偏好设置 JSON 路径（provider/limit/summary）。
* `apiKey` 的值会回退到环境变量（`ELEVENLABS_API_KEY`/`XI_API_KEY`、`OPENAI_API_KEY`）。
* `elevenlabs.baseUrl`: 覆盖 ElevenLabs API 基础 URL。
* `elevenlabs.voiceSettings`：
  * `stability`、`similarityBoost`、`style`: `0..1`
  * `useSpeakerBoost`: `true|false`
  * `speed`: `0.5..2.0`（1.0 = 正常）
* `elevenlabs.applyTextNormalization`: `auto|on|off`
* `elevenlabs.languageCode`: 2 位 ISO 639-1 语言代码（例如 `en`、`de`）
* `elevenlabs.seed`: 整数 `0..4294967295`（尽量保证结果可复现）
* `edge.enabled`: 允许使用 Edge TTS（默认 `true`；无需 API key）。
* `edge.voice`: Edge 神经网络语音名称（例如 `en-US-MichelleNeural`）。
* `edge.lang`: 语言代码（例如 `en-US`）。
* `edge.outputFormat`: Edge 输出格式（例如 `audio-24khz-48kbitrate-mono-mp3`）。
  * 有效取值见 Microsoft Speech 输出格式；并非所有格式都受 Edge 支持。
* `edge.rate` / `edge.pitch` / `edge.volume`: 百分比字符串（例如 `+10%`、`-5%`）。
* `edge.saveSubtitles`: 将 JSON 字幕文件与音频文件一并写出。
* `edge.proxy`: Edge TTS 请求使用的代理 URL。
* `edge.timeoutMs`: 请求超时覆盖配置（毫秒）。

<div id="model-driven-overrides-default-on">
  ## 模型驱动的覆盖（默认开启）
</div>

默认情况下，模型**可以**为单条回复输出 TTS 指令。
当 `messages.tts.auto` 为 `tagged` 时，这些指令是触发音频所必需的。

启用后，模型可以输出 `[[tts:...]]` 指令来覆盖单条回复所使用的语音，
并可选使用一个 `[[tts:text]]...[[/tts:text]]` 区块来提供表达性标签
（如笑声、歌唱提示等），这些标签只应出现在音频中。

示例回复载荷：

```
Here you go.

[[tts:provider=elevenlabs voiceId=pMsXgVXv3BLzUgSXRplE model=eleven_v3 speed=1.1]]
[[tts:text]](laughs) Read the song once more.[[/tts:text]]
```

可用的指令键（启用后）：

* `provider`（`openai` | `elevenlabs` | `edge`）
* `voice`（OpenAI 语音）或 `voiceId`（ElevenLabs）
* `model`（OpenAI TTS 模型或 ElevenLabs 模型 ID）
* `stability`、`similarityBoost`、`style`、`speed`、`useSpeakerBoost`
* `applyTextNormalization`（`auto|on|off`）
* `languageCode`（ISO 639-1）
* `seed`

禁用所有模型覆盖：

```json5
{
  messages: {
    tts: {
      modelOverrides: {
        enabled: false
      }
    }
  }
}
```

可选的允许列表（在保持标签启用的同时禁用特定覆盖配置）：

```json5
{
  messages: {
    tts: {
      modelOverrides: {
        enabled: true,
        allowProvider: false,
        allowSeed: false
      }
    }
  }
}
```

<div id="per-user-preferences">
  ## 用户级偏好设置
</div>

斜杠命令会将本地覆盖配置写入 `prefsPath`（默认：
`~/.openclaw/settings/tts.json`，可通过 `OPENCLAW_TTS_PREFS` 或
`messages.tts.prefsPath` 覆盖）。

存储的字段包括：

* `enabled`
* `provider`
* `maxLength`（摘要阈值；默认 1500 个字符）
* `summarize`（默认 `true`）

这些会覆盖该主机上的 `messages.tts.*` 设置。

<div id="output-formats-fixed">
  ## 输出格式（固定）
</div>

* **Telegram**：Opus 语音消息（ElevenLabs 使用 `opus_48000_64`，OpenAI 使用 `opus`）。
  * 48kHz / 64kbps 是语音消息的较好折中，也是圆形语音气泡所要求的格式。
* **其他渠道**：MP3（ElevenLabs 使用 `mp3_44100_128`，OpenAI 使用 `mp3`）。
  * 44.1kHz / 128kbps 是语音清晰度的默认折中设置。
* **Edge TTS**：使用 `edge.outputFormat`（默认值为 `audio-24khz-48kbitrate-mono-mp3`）。
  * `node-edge-tts` 接受一个 `outputFormat`，但 Edge 服务并不支持所有格式。 citeturn2search0
  * 输出格式的取值遵循 Microsoft Speech 的输出格式规范（包括 Ogg/WebM Opus）。 citeturn1search0
  * Telegram 的 `sendVoice` 接受 OGG/MP3/M4A；如果你需要确保使用 Opus 格式的语音消息，请使用 OpenAI/ElevenLabs。 citeturn1search1
  * 如果配置的 Edge 输出格式失败，OpenClaw 会回退为 MP3 并重试。

OpenAI/ElevenLabs 的格式是固定的；Telegram 为了语音消息的用户体验期望使用 Opus。

<div id="auto-tts-behavior">
  ## 自动 TTS 行为
</div>

启用后，OpenClaw 会：

* 如果回复已经包含媒体或 `MEDIA:` 指令，则跳过 TTS。
* 跳过非常短的回复（&lt; 10 个字符）。
* 在启用摘要时，使用 `agents.defaults.model.primary`（或 `summaryModel`）对较长回复进行摘要。
* 将生成的音频附加到回复中。

如果回复长度超过 `maxLength`，且未启用摘要（或没有摘要模型的 api key），则会跳过音频，仅发送普通文本回复。

<div id="flow-diagram">
  ## 流程图
</div>

```
Reply -> TTS enabled?
  no  -> send text
  yes -> has media / MEDIA: / short?
          yes -> send text
          no  -> length > limit?
                   no  -> TTS -> attach audio
                   yes -> summary enabled?
                            no  -> send text
                            yes -> summarize (summaryModel or agents.defaults.model.primary)
                                      -> TTS -> attach audio
```

<div id="slash-command-usage">
  ## 斜杠命令用法
</div>

当前只有一个命令：`/tts`。
启用方式的详细说明请参见[斜杠命令](/zh/tools/slash-commands)。

Discord 说明：`/tts` 是 Discord 的内置命令，因此 OpenClaw 会在 Discord 中注册
`/voice` 作为对应的原生命令。`/tts ...` 作为文本命令仍然可用。

```
/tts off
/tts always
/tts inbound
/tts tagged
/tts status
/tts provider openai
/tts limit 2000
/tts summary off
/tts audio Hello from OpenClaw
```

说明：

* 命令要求由已授权的发送者执行（允许列表/owner 规则仍然生效）。
* 必须启用 `commands.text` 或原生命令注册。
* `off|always|inbound|tagged` 是按会话生效的开关选项（`/tts on` 是 `/tts always` 的别名）。
* `limit` 和 `summary` 存储在本地偏好设置中，而不是主配置文件中。
* `/tts audio` 会生成一次性音频回复（不会切换 TTS 开关）。

<div id="agent-tool">
  ## Agent 工具
</div>

`tts` 工具会将文本转换为语音，并返回一个 `MEDIA:` 路径。当结果与 Telegram 兼容时，该工具会包含 `[[audio_as_voice]]`，从而让 Telegram 发送语音消息气泡。

<div id="gateway-rpc">
  ## Gateway RPC
</div>

Gateway 的 RPC 方法：

* `tts.status`
* `tts.enable`
* `tts.disable`
* `tts.convert`
* `tts.setProvider`
* `tts.providers`