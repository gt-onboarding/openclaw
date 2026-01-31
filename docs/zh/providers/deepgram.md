---
title: Deepgram
summary: "用于传入语音消息的 Deepgram 转写"
read_when:
  - 你希望为音频附件使用 Deepgram 语音转文本
  - 你需要一个快速上手的 Deepgram 配置示例
---

<div id="deepgram-audio-transcription">
  # Deepgram（音频转写）
</div>

Deepgram 是一个语音转文本的 api。在 OpenClaw 中，它通过 `tools.media.audio` 用于**传入音频 / 语音留言的转写**。

启用后，OpenClaw 会将音频文件上传到 Deepgram，并将转写结果注入到回复处理流水线中（`{{Transcript}}` + `[Audio]` 区块）。这**不是流式处理**；它使用的是预录音频的转写接口。

Website: https://deepgram.com  
Docs: https://developers.deepgram.com

<div id="quick-start">
  ## 快速开始
</div>

1. 设置 API 密钥：

```
DEEPGRAM_API_KEY=dg_...
```

2. 启用此提供方：

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


<div id="options">
  ## 选项
</div>

* `model`: Deepgram 模型 ID（默认：`nova-3`）
* `language`: 语言提示（可选）
* `tools.media.audio.providerOptions.deepgram.detect_language`: 启用语言自动检测（可选）
* `tools.media.audio.providerOptions.deepgram.punctuate`: 启用自动加标点（可选）
* `tools.media.audio.providerOptions.deepgram.smart_format`: 启用智能格式化（可选）

包含 language 的示例：

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [
          { provider: "deepgram", model: "nova-3", language: "en" }
        ]
      }
    }
  }
}
```

Deepgram 选项示例：

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        providerOptions: {
          deepgram: {
            detect_language: true,
            punctuate: true,
            smart_format: true
          }
        },
        models: [{ provider: "deepgram", model: "nova-3" }]
      }
    }
  }
}
```


<div id="notes">
  ## 注意事项
</div>

- 身份验证遵循标准的提供方验证顺序；`DEEPGRAM_API_KEY` 是最简单的方式。
- 使用代理时，可以通过 `tools.media.audio.baseUrl` 和 `tools.media.audio.headers` 自定义端点或请求头。
- 输出遵循与其他提供方相同的音频规则（大小上限、超时、转录内容注入）。