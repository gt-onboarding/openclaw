---
title: 媒体理解
summary: "入站图像/音频/视频理解（可选），支持基于提供方与 CLI 的回退方案"
read_when:
  - 设计或重构媒体理解功能时
  - 调优入站音频/视频/图像预处理流程时
---

<div id="media-understanding-inbound-2026-01-17">
  # 媒体理解（入站）— 2026-01-17
</div>

在回复流水线运行之前，OpenClaw 可以先对**入站媒体**（图片/音频/视频）生成摘要。它会自动检测本地工具或提供方密钥是否可用，并且也可以禁用或自定义。如果理解功能关闭，模型仍会像往常一样接收原始文件/URL。

<div id="goals">
  ## 目标
</div>

* 可选：对入站媒体进行预先压缩/提炼为简短文本，以加快路由并改进命令解析。
* 始终保留向模型传递的原始媒体。
* 支持 **提供方 APIs** 和 **CLI 兜底机制**。
* 允许配置多个模型并按顺序回退（错误/大小/超时）。

<div id="highlevel-behavior">
  ## 高层行为
</div>

1. 收集入站附件（`MediaPaths`、`MediaUrls`、`MediaTypes`）。
2. 针对每个已启用的能力（图像/音频/视频），按策略选择附件（默认：**第一个**）。
3. 选择第一个符合条件的模型项（尺寸 + 能力 + 认证）。
4. 如果某个模型调用失败或媒体过大，则**回退到下一个项**。
5. 成功时：
   * `Body` 将变为 `[Image]`、`[Audio]` 或 `[Video]` 块。
   * 音频会设置 `{{Transcript}}`；命令解析在存在字幕文本时优先使用字幕文本，
     否则使用转录文本。
   * 字幕会作为块内的 `User text:` 保留。

如果理解失败或被禁用，**回复流程会继续执行**，使用原始正文 + 附件。

<div id="config-overview">
  ## 配置概览
</div>

`tools.media` 支持**共享模型**以及按能力进行单独覆写：

* `tools.media.models`：共享模型列表（使用 `capabilities` 进行控制）。
* `tools.media.image` / `tools.media.audio` / `tools.media.video`：
  * 默认值（`prompt`、`maxChars`、`maxBytes`、`timeoutSeconds`、`language`）
  * 提供方级覆写配置（`baseUrl`、`headers`、`providerOptions`）
  * 通过 `tools.media.audio.providerOptions.deepgram` 配置 Deepgram 音频选项
  * 可选的**按能力划分的 `models` 列表**（优先于共享模型）
  * `attachments` 策略（`mode`、`maxAttachments`、`prefer`）
  * `scope`（可选，按 channel/chatType/session 键进行限制）
* `tools.media.concurrency`：单次可并发运行的能力数量上限（默认 **2**）。

```json5
{
  tools: {
    media: {
      models: [ /* 共享列表 */ ],
      image: { /* optional overrides */ },
      audio: { /* optional overrides */ },
      video: { /* optional overrides */ }
    }
  }
}
```

<div id="model-entries">
  ### 模型条目
</div>

每个 `models[]` 项可以是 **提供方** 或 **CLI**：

```json5
{
  type: "provider",        // default if omitted
  provider: "openai",
  model: "gpt-5.2",
  prompt: "Describe the image in <= 500 chars.",
  maxChars: 500,
  maxBytes: 10485760,
  timeoutSeconds: 60,
  capabilities: ["image"], // 可选,用于多模态条目
  profile: "vision-profile",
  preferredProfile: "vision-fallback"
}
```

```json5
{
  type: "cli",
  command: "gemini",
  args: [
    "-m",
    "gemini-3-flash",
    "--allowed-tools",
    "read_file",
    "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters."
  ],
  maxChars: 500,
  maxBytes: 52428800,
  timeoutSeconds: 120,
  capabilities: ["video", "image"]
}
```

CLI 模板也可以使用：

* `{{MediaDir}}`（包含媒体文件的目录）
* `{{OutputDir}}`（为本次运行创建的临时工作目录）
* `{{OutputBase}}`（临时文件的基础路径，不含扩展名）

<div id="defaults-and-limits">
  ## 默认值和限制
</div>

推荐默认值：

* `maxChars`：图像/视频为 **500**（简短、便于命令行使用）
* `maxChars`：音频为 **未设置**（除非你手动设置上限，否则返回完整转录）
* `maxBytes`：
  * 图像：**10MB**
  * 音频：**20MB**
  * 视频：**50MB**

规则：

* 如果媒体大小超过 `maxBytes`，将跳过该模型并**尝试下一个模型**。
* 如果模型返回的内容超过 `maxChars`，输出会被截断。
* `prompt` 默认为简单的 “Describe the {media}.”，再加上关于 `maxChars` 的说明（仅限图像/视频）。
* 如果 `<capability>.enabled: true` 但没有配置任何模型，当其提供方支持该能力时，OpenClaw 会尝试使用**当前启用的回复模型**。

<div id="auto-detect-media-understanding-default">
  ### 自动检测媒体理解（默认）
</div>

如果 `tools.media.<capability>.enabled` **没有**被设置为 `false`，并且你尚未
配置模型，OpenClaw 会按如下顺序自动检测，并在**找到第一个可用选项时停止**：

1. **本地 CLI**（仅音频；如果已安装）
   * `sherpa-onnx-offline`（需要包含 encoder/decoder/joiner/tokens 的 `SHERPA_ONNX_MODEL_DIR`）
   * `whisper-cli`（`whisper-cpp`；使用 `WHISPER_CPP_MODEL` 或内置的 tiny 模型）
   * `whisper`（Python CLI；会自动下载模型）
2. **Gemini CLI**（`gemini`），使用 `read_many_files`
3. **提供方密钥**
   * 音频：OpenAI → Groq → Deepgram → Google
   * 图像：OpenAI → Anthropic → Google → MiniMax
   * 视频：Google

要禁用自动检测，请设置：

```json5
{
  tools: {
    media: {
      audio: {
        enabled: false
      }
    }
  }
}
```

注意：在 macOS/Linux/Windows 上的二进制检测只能做到尽力而为；请确保 CLI 已在 `PATH` 中（我们会展开 `~`），或者使用包含完整命令路径的方式显式配置 CLI 模型。

<div id="capabilities-optional">
  ## 功能（可选）
</div>

如果你设置了 `capabilities`，该条目只会用于这些媒体类型。对于共享
列表，OpenClaw 可以推断默认值：

* `openai`, `anthropic`, `minimax`: **图像**
* `google` (Gemini API): **图像 + 音频 + 视频**
* `groq`: **音频**
* `deepgram`: **音频**

对于 CLI 条目，**务必显式设置 `capabilities`**，以避免出现意外匹配。
如果你省略 `capabilities`，该条目会被视为适用于其所在的列表。

<div id="provider-support-matrix-openclaw-integrations">
  ## 提供方支持矩阵（OpenClaw 集成）
</div>

| 功能 | 提供方集成 | 说明 |
|------------|----------------------|-------|
| 图像 | OpenAI / Anthropic / Google / 其他通过 `pi-ai` | 注册表中任何支持图像的模型都可使用。 |
| 音频 | OpenAI, Groq, Deepgram, Google | 由提供方进行音频转写（Whisper/Deepgram/Gemini）。 |
| 视频 | Google（Gemini API） | 由提供方进行视频理解。 |

<div id="recommended-providers">
  ## 推荐提供方
</div>

**图像**

* 如果你当前使用的模型支持图像，优先使用该模型。
* 推荐默认选项：`openai/gpt-5.2`、`anthropic/claude-opus-4-5`、`google/gemini-3-pro-preview`。

**音频**

* `openai/gpt-4o-mini-transcribe`、`groq/whisper-large-v3-turbo` 或 `deepgram/nova-3`。
* CLI 兜底方案：`whisper-cli`（whisper-cpp）或 `whisper`。
* Deepgram 配置：[Deepgram（音频转写）](/zh/providers/deepgram)。

**视频**

* `google/gemini-3-flash-preview`（更快）、`google/gemini-3-pro-preview`（能力更强）。
* CLI 兜底方案：`gemini` CLI（在视频/音频上支持 `read_file`）。

<div id="attachment-policy">
  ## 附件策略
</div>

按能力维度配置的 `attachments` 用于控制要处理哪些附件：

* `mode`：`first`（默认）或 `all`
* `maxAttachments`：设定要处理的附件数量上限（默认 **1**）
* `prefer`：`first`、`last`、`path`、`url`

当 `mode: "all"` 时，输出会被标记为 `[Image 1/2]`、`[Audio 2/2]` 等。

<div id="config-examples">
  ## 配置示例
</div>

<div id="1-shared-models-list-overrides">
  ### 1) 共享模型列表 + 覆盖配置
</div>

```json5
{
  tools: {
    media: {
      models: [
        { provider: "openai", model: "gpt-5.2", capabilities: ["image"] },
        { provider: "google", model: "gemini-3-flash-preview", capabilities: ["image", "audio", "video"] },
        {
          type: "cli",
          command: "gemini",
          args: [
            "-m",
            "gemini-3-flash",
            "--allowed-tools",
            "read_file",
            "读取 {{MediaPath}} 处的媒体并在 <= {{MaxChars}} 个字符内描述它。"
          ],
          capabilities: ["image", "video"]
        }
      ],
      audio: {
        attachments: { mode: "all", maxAttachments: 2 }
      },
      video: {
        maxChars: 500
      }
    }
  }
}
```

<div id="2-audio-video-only-image-off">
  ### 2) 仅音频和视频（关闭图像理解）
</div>

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          {
            type: "cli",
            command: "whisper",
            args: ["--model", "base", "{{MediaPath}}"]
          }
        ]
      },
      video: {
        enabled: true,
        maxChars: 500,
        models: [
          { provider: "google", model: "gemini-3-flash-preview" },
          {
            type: "cli",
            command: "gemini",
            args: [
              "-m",
              "gemini-3-flash",
              "--allowed-tools",
              "read_file",
              "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters."
            ]
          }
        ]
      }
    }
  }
}
```

<div id="3-optional-image-understanding">
  ### 3) 可选的图像理解
</div>

```json5
{
  tools: {
    media: {
      image: {
        enabled: true,
        maxBytes: 10485760,
        maxChars: 500,
        models: [
          { provider: "openai", model: "gpt-5.2" },
          { provider: "anthropic", model: "claude-opus-4-5" },
          {
            type: "cli",
            command: "gemini",
            args: [
              "-m",
              "gemini-3-flash",
              "--allowed-tools",
              "read_file",
              "读取 {{MediaPath}} 处的媒体并在 <= {{MaxChars}} 个字符内描述它。"
            ]
          }
        ]
      }
    }
  }
}
```

<div id="4-multimodal-single-entry-explicit-capabilities">
  ### 4) 多模态单一入口（显式能力）
</div>

```json5
{
  tools: {
    media: {
      image: { models: [{ provider: "google", model: "gemini-3-pro-preview", capabilities: ["image", "video", "audio"] }] },
      audio: { models: [{ provider: "google", model: "gemini-3-pro-preview", capabilities: ["image", "video", "audio"] }] },
      video: { models: [{ provider: "google", model: "gemini-3-pro-preview", capabilities: ["image", "video", "audio"] }] }
    }
  }
}
```

<div id="status-output">
  ## 状态输出
</div>

当媒体理解功能在运行时，`/status` 会包含一行简短的摘要：

```
📎 Media: image ok (openai/gpt-5.2) · audio skipped (maxBytes)
```

这里将按能力展示各项结果，以及在适用时所选择的提供方/模型。

<div id="notes">
  ## 说明
</div>

* 理解是按**尽力而为**原则进行的。出错不会阻止回复。
* 即使关闭理解功能，附件仍会传递给模型。
* 使用 `scope` 来限制理解运行的范围（例如仅在私信 DM 中）。

<div id="related-docs">
  ## 相关文档
</div>

* [配置](/zh/gateway/configuration)
* [图像和媒体支持](/zh/nodes/images)