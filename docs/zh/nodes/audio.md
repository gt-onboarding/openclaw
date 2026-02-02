---
title: 音频
summary: "收到的音频 / 语音消息如何被下载、转录并注入到回复中"
read_when:
  - 修改音频转录或媒体处理方式
---

<div id="audio-voice-notes-2026-01-17">
  # 音频/语音笔记 — 2026-01-17
</div>

<div id="what-works">
  ## 当前可用功能
</div>

* **媒体理解（音频）**：如果启用了（或自动检测到）音频理解，OpenClaw 会：
  1. 定位第一个音频附件（本地路径或 URL），必要时先下载。
  2. 在发送到每个模型项之前强制执行 `maxBytes` 限制。
  3. 按顺序运行第一个符合条件的模型项（提供方或 CLI）。
  4. 如果失败或被跳过（因大小/超时），则尝试下一个项。
  5. 成功后，用一个 `[Audio]` 块替换 `Body`，并设置 `{{Transcript}}`。
* **命令解析**：当转录成功时，会将 `CommandBody`/`RawBody` 设置为该转录文本，以便斜杠命令仍然可用。
* **详细日志**：在 `--verbose` 模式下，会记录转录何时运行以及何时替换消息正文。

<div id="auto-detection-default">
  ## 自动检测（默认）
</div>

如果你**不配置模型**，并且 `tools.media.audio.enabled` **没有**被设置为 `false`，
OpenClaw 会按如下顺序自动检测，并在找到第一个可用选项时停止：

1. **本地 CLI**（如果已安装）
   * `sherpa-onnx-offline`（需要设置 `SHERPA_ONNX_MODEL_DIR`，其中包含 encoder/decoder/joiner/tokens）
   * `whisper-cli`（来自 `whisper-cpp`；使用 `WHISPER_CPP_MODEL` 或自带的 tiny 模型）
   * `whisper`（Python CLI；会自动下载模型）
2. 使用 `read_many_files` 的 **Gemini CLI**（`gemini`）
3. **提供方密钥**（OpenAI → Groq → Deepgram → Google）

要禁用自动检测，将 `tools.media.audio.enabled` 设置为 `false`。
要进行自定义，将 `tools.media.audio.models` 设置为相应值。
注意：可执行文件检测在 macOS/Linux/Windows 上是尽力而为；请确保 CLI 在 `PATH` 中（我们会展开 `~`），或者通过完整命令路径显式指定一个 CLI 模型。

<div id="config-examples">
  ## 配置示例
</div>

<div id="provider-cli-fallback-openai-whisper-cli">
  ### 提供方 + CLI 回退方案（OpenAI + Whisper CLI）
</div>

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        maxBytes: 20971520,
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          {
            type: "cli",
            command: "whisper",
            args: ["--model", "base", "{{MediaPath}}"],
            timeoutSeconds: 45
          }
        ]
      }
    }
  }
}
```

<div id="provider-only-with-scope-gating">
  ### 仅限提供方（使用 scope 限制）
</div>

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        scope: {
          default: "allow",
          rules: [
            { action: "deny", match: { chatType: "group" } }
          ]
        },
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" }
        ]
      }
    }
  }
}
```

<div id="provider-only-deepgram">
  ### 仅限提供方（Deepgram）
</div>

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "deepgram", model: "nova-3" }]
      }
    }
  }
}
```

<div id="notes-limits">
  ## 注意事项与限制
</div>

* 提供方认证遵循标准的模型认证顺序（认证配置、环境变量、`models.providers.*.apiKey`）。
* 使用 `provider: "deepgram"` 时，Deepgram 会自动读取 `DEEPGRAM_API_KEY`。
* Deepgram 配置详情：[Deepgram（音频转写）](/zh/providers/deepgram)。
* 音频提供方可以通过 `tools.media.audio` 覆盖 `baseUrl`、`headers` 和 `providerOptions`。
* 默认大小上限为 20MB（`tools.media.audio.maxBytes`）。超出大小的音频会在该模型下被跳过，然后尝试下一个条目。
* 音频的默认 `maxChars` 为**未设置**（完整转写）。可设置全局 `tools.media.audio.maxChars` 或按条目设置 `maxChars` 来裁剪输出。
* OpenAI 默认自动使用 `gpt-4o-mini-transcribe`；将 `model` 设置为 `"gpt-4o-transcribe"` 以获得更高精度。
* 使用 `tools.media.audio.attachments` 处理多条语音（`mode: "all"` + `maxAttachments`）。
* 转写文本在模板中可通过 `{{Transcript}}` 获取。
* CLI 标准输出上限为 5MB；请保持 CLI 输出简洁。

<div id="gotchas">
  ## 注意事项
</div>

* scope 规则采用“先匹配即生效”原则。`chatType` 会被规范化为 `direct`、`group` 或 `room`。
* 确保你的 CLI 以退出码 0 结束并输出纯文本；JSON 需要通过 `jq -r .text` 做处理。
* 将超时时间设置为合理值（`timeoutSeconds`，默认 60s），以避免阻塞回复队列。