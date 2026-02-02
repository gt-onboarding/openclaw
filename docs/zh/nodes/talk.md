---
title: Talk
summary: "Talk 模式：使用 ElevenLabs TTS 实现连续语音对话"
read_when:
  - 在 macOS/iOS/Android 上实现 Talk 模式
  - 调整语音/TTS/中断行为
---

<div id="talk-mode">
  # 对话模式
</div>

对话模式是一个持续进行的语音会话循环：

1) 监听语音
2) 将语音转写文本发送给模型（主会话，chat.send）
3) 等待响应
4) 通过 ElevenLabs 语音朗读响应（流式播放）

<div id="behavior-macos">
  ## 行为（macOS）
</div>

- 当启用 Talk 模式时，**始终显示悬浮层**。
- **Listening → Thinking → Speaking** 阶段切换。
- 在出现**短暂停顿**（静音窗口）时，会发送当前转写文本。
- 回复会**写入 WebChat**（效果等同于键入）。
- **语音打断**（默认开启）：如果用户在助手说话时开始说话，我们会停止播放，并记录中断时间戳，以用于下一次提示。

<div id="voice-directives-in-replies">
  ## 回复中的语音指令
</div>

assistant 可以在回复内容前添加一行 **JSON 单行** 来控制语音：

```json
{"voice":"<voice-id>","once":true}
```

规则：

* 仅处理首个非空行。
* 会忽略未知键。
* `once: true` 只对当前回复生效。
* 未设置 `once` 时，该语音会成为 Talk 模式的新默认值。
* 在 TTS 播放前会去除该 JSON 行。

支持的键：

* `voice` / `voice_id` / `voiceId`
* `model` / `model_id` / `modelId`
* `speed`、`rate`（WPM）、`stability`、`similarity`、`style`、`speakerBoost`
* `seed`、`normalize`、`lang`、`output_format`、`latency_tier`
* `once`


<div id="config-openclawopenclawjson">
  ## 配置（`~/.openclaw/openclaw.json`）
</div>

```json5
{
  "talk": {
    "voiceId": "elevenlabs_voice_id",
    "modelId": "eleven_v3",
    "outputFormat": "mp3_44100_128",
    "apiKey": "elevenlabs_api_key",
    "interruptOnSpeech": true
  }
}
```

Defaults:

* `interruptOnSpeech`: true
* `voiceId`: 如果未设置，则回退到 `ELEVENLABS_VOICE_ID` / `SAG_VOICE_ID`（或在 API key 可用时使用第一个 ElevenLabs 语音）
* `modelId`: 未设置时默认为 `eleven_v3`
* `apiKey`: 如果未设置，则回退到 `ELEVENLABS_API_KEY`（或在可用时使用 Gateway shell 配置文件）
* `outputFormat`: 在 macOS/iOS 上默认为 `pcm_44100`，在 Android 上默认为 `pcm_24000`（将其设置为 `mp3_*` 以强制使用 MP3 流式传输）


<div id="macos-ui">
  ## macOS UI
</div>

- 菜单栏开关：**Talk**
- 配置标签页：**Talk 模式**分组（语音 ID + 打断开关）
- 悬浮面板：
  - **Listening**：云朵随麦克风音量脉动
  - **Thinking**：下沉动画
  - **Speaking**：辐射状波纹
  - 点击云朵：停止发声
  - 点击 X：退出 Talk 模式

<div id="notes">
  ## 说明
</div>

- 需要语音与麦克风权限。
- 使用 `chat.send` 方法，针对会话键 `main`。
- TTS 使用带有 `ELEVENLABS_API_KEY` 的 ElevenLabs 流式 API，并在 macOS/iOS/Android 上进行增量播放以降低延迟。
- `eleven_v3` 的 `stability` 仅允许取值 `0.0`、`0.5` 或 `1.0`；其他模型接受 `0..1`。
- 当设置时，`latency_tier` 仅允许取值 `0..4`。
- Android 支持 `pcm_16000`、`pcm_22050`、`pcm_24000` 和 `pcm_44100` 输出格式，用于低延迟的 AudioTrack 流式传输。