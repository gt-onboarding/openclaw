---
title: 图像
summary: "针对发送操作、Gateway 和智能体回复的图像与媒体处理规则"
read_when:
  - 修改媒体流水线或附件
---

<div id="image-media-support-2025-12-05">
  # 图像和媒体支持 — 2025-12-05
</div>

WhatsApp 通道通过 **Baileys Web** 运行。本文档记录了当前针对 send、Gateway 和智能体回复的媒体处理规则。

<div id="goals">
  ## 目标
</div>

- 通过 `openclaw message send --media` 发送带可选说明文字的媒体。
- 允许从 Web 收件箱发送的自动回复在文本之外附带媒体内容。
- 保持按类型划分的限制合理且可预测。

<div id="cli-surface">
  ## CLI 命令概览
</div>

- `openclaw message send --media <path-or-url> [--message <caption>]`
  - `--media` 为可选参数；说明文字可以为空，以便仅发送媒体内容。
  - `--dry-run` 打印解析后的请求负载；`--json` 输出 `{ channel, to, messageId, mediaUrl, caption }`。

<div id="whatsapp-web-channel-behavior">
  ## WhatsApp Web 渠道行为
</div>

- 输入：本地文件路径 **或** HTTP(S) URL。
- 流程：加载为 Buffer，检测媒体类型，并构建正确的消息载荷：
  - **图片：** 调整尺寸并重新压缩为 JPEG（最长边 2048px），目标大小为 `agents.defaults.mediaMaxMb`（默认 5 MB），硬上限为 6 MB。
  - **音频/语音/视频：** 直接透传，最大 16 MB；音频将作为语音消息发送（`ptt: true`）。
  - **文档：** 其他任何类型，最大 100 MB，如有文件名则保留。
- WhatsApp GIF 式播放：发送一个带有 `gifPlayback: true` 的 MP4（CLI：`--gif-playback`），以便移动客户端内联循环播放。
- MIME 检测优先使用文件魔数，其次是文件头，最后是文件扩展名。
- 说明文字来源于 `--message` 或 `reply.text`；允许为空说明文字。
- 日志：在非 verbose 模式下只显示 `↩️`/`✅`；在 verbose 模式下会包含大小和源路径/URL。

<div id="auto-reply-pipeline">
  ## 自动回复处理流水线
</div>

- `getReplyFromConfig` 返回 `{ text?, mediaUrl?, mediaUrls? }`。
- 当存在媒体内容时，Web 发送器会使用与 `openclaw message send` 相同的处理流水线来解析本地路径或 URL。
- 如果提供了多个媒体项，则会按顺序依次发送。

<div id="inbound-media-to-commands-pi">
  ## 传入媒体到命令（Pi）
</div>

- 当入站 Web 消息包含媒体时，OpenClaw 会将其下载到一个临时文件，并暴露以下模板变量：
  - `{{MediaUrl}}`：指向传入媒体的伪 URL。
  - `{{MediaPath}}`：在运行命令之前写入的本地临时路径。
- 当启用了每会话 Docker 沙箱时，传入媒体会被复制到沙箱工作区中，并且 `MediaPath` / `MediaUrl` 会被重写为类似 `media/inbound/<filename>` 的相对路径。
- 媒体理解（如果通过 `tools.media.*` 或共享的 `tools.media.models` 进行了配置）会在模板渲染之前运行，并可以在 `Body` 中插入 `[Image]`、`[Audio]` 和 `[Video]` 块。
  - 音频会设置 `{{Transcript}}`，并使用该转录文本进行命令解析，因此斜杠命令仍然可以正常工作。
  - 视频和图像描述会保留任何字幕文本，用于命令解析。
- 默认情况下，只处理第一个匹配的图像 / 音频 / 视频附件；设置 `tools.media.<cap>.attachments` 以处理多个附件。

<div id="limits-errors">
  ## 限制与错误
</div>

**出站发送上限（WhatsApp Web 发送）**

- 图片：重新压缩后约 6 MB 上限。
- 音频/语音/视频：16 MB 上限；文档：100 MB 上限。
- 过大或无法读取的媒体 → 在日志中记录明确错误，并跳过该条回复。

**媒体理解上限（转录/描述）**

- 图片默认：10 MB（`tools.media.image.maxBytes`）。
- 音频默认：20 MB（`tools.media.audio.maxBytes`）。
- 视频默认：50 MB（`tools.media.video.maxBytes`）。
- 过大的媒体会跳过理解，但回复仍会使用原始正文发送。

<div id="notes-for-tests">
  ## 测试注意事项
</div>

- 覆盖图片 / 音频 / 文档场景下的发送与回复流程。
- 验证图片的重新压缩（大小限制）以及音频的语音消息标志。
- 确保多媒体回复被拆分为按顺序进行的发送操作。