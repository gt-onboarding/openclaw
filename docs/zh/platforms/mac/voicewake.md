---
title: 语音唤醒
summary: "Mac 应用中的语音唤醒和按键说话（PTT）模式及路由细节"
read_when:
  - 在处理语音唤醒或 PTT 流程时
---

<div id="voice-wake-push-to-talk">
  # 语音唤醒与按住说话
</div>

<div id="modes">
  ## 模式
</div>

- **唤醒词模式**（默认）：始终开启的语音识别器会等待触发词（`swabbleTriggerWords`）。匹配到后开始录音，显示带有中间识别结果的浮层，并在检测到静音后自动发送。
- **按键说话（按住右 Option 键）**：按住右侧 Option 键即可立即开始录音——不需要唤醒词。按住期间会显示浮层；松开后会在短暂延迟后完成最终识别并发送文本，方便你在此之前对内容进行微调。

<div id="runtime-behavior-wake-word">
  ## 运行时行为（唤醒词）
</div>

- 语音识别器运行在 `VoiceWakeRuntime` 中。
- 只有当唤醒词和下一个词之间存在**足够明显的停顿**（约 0.55 秒间隔）时才会触发。即使指令尚未开始，覆盖层/提示音也可以在这段停顿期间启动。
- 静音检测窗口：语音持续输入时为 2.0 秒；如果只听到唤醒触发词则为 5.0 秒。
- 强制停止时间：120 秒，用于防止会话失控。
- 会话之间的去抖间隔：350ms。
- 覆盖层由 `VoiceWakeOverlayController` 驱动，并使用“已提交/暂态”两种状态配色。
- 发送之后，识别器会被完整重启，以监听下一个触发词。

<div id="lifecycle-invariants">
  ## 生命周期不变式
</div>

- 如果启用了 Voice Wake 且已授予相关权限，唤醒词识别器应始终处于监听状态（除非正在进行显式的按键说话捕获）。
- 覆盖层是否可见（包括通过 X 按钮手动关闭）都不得阻止识别器恢复监听。

<div id="sticky-overlay-failure-mode-previous">
  ## 粘滞覆盖层失效模式（先前行为）
</div>

此前，如果覆盖层卡在可见状态，而你手动将其关闭，Voice Wake 可能会表现得像“失灵”了一样，因为运行时尝试重启可能会被覆盖层可见性阻塞，且不会再安排后续的重启尝试。

加固措施：

- Voice Wake 运行时的重启不再会被覆盖层可见性阻塞。
- 覆盖层关闭完成后会通过 `VoiceSessionCoordinator` 触发一次 `VoiceWakeRuntime.refresh(...)`，因此手动点击 X 关闭后总会恢复监听。

<div id="push-to-talk-specifics">
  ## 按住说话细节
</div>

- 热键检测使用全局 `.flagsChanged` 监听 **右 Option**（`keyCode 61` + `.option`）。我们只观察事件（不拦截/不吞事件）。
- 采集流水线位于 `VoicePushToTalk`：立即启动 Speech，将中间结果流式发送到叠加层，并在按键松开时调用 `VoiceWakeForwarder`。
- 按住说话开始时，我们会暂停唤醒词运行时，以避免出现音频 tap 互相抢占；松开后会自动重新启动。
- 权限：需要麦克风 + Speech；要监听键盘事件则需要辅助功能/输入监控授权。
- 外接键盘：有些可能不会按预期暴露右 Option——如果用户反馈经常按了没反应，应提供备用快捷键。

<div id="user-facing-settings">
  ## 面向用户的设置
</div>

- **Voice Wake** 开关：启用唤醒词运行时功能。
- **按住 Cmd+Fn 说话**：启用按键说话监控。在 macOS < 26 上不可用。
- 语言与麦克风选择器、实时电平表、触发词列表、测试器（仅本地；不会转发）。
- 当设备断开时，麦克风选择器会保留上次选择，显示已断开提示，并在设备恢复前临时回退到系统默认设备。
- **Sounds**：在检测到触发和发送时播放提示音；默认使用 macOS 的 “Glass” 系统音效。你可以为每个事件选择任意 `NSSound` 可加载文件（例如 MP3/WAV/AIFF），或者选择 **No Sound**。

<div id="forwarding-behavior">
  ## 转发行为
</div>

- 当启用 Voice Wake 时，转录内容会被转发到当前激活的 Gateway/智能体（沿用 mac 应用其余部分使用的本地/远程模式）。
- 回复会发送到**最近一次使用的主提供方**（WhatsApp/Telegram/Discord/WebChat）。如果发送失败，错误会被记录，该次运行仍可通过 WebChat/会话日志查看。

<div id="forwarding-payload">
  ## 转发载荷
</div>

- `VoiceWakeForwarder.prefixedTranscript(_:)` 会在发送前预置机器提示文本。由唤醒词和按键说话两条路径共用。

<div id="quick-verification">
  ## 快速验证
</div>

- 开启按键说话功能，按住 Cmd+Fn，说话后松开：浮层应先显示部分识别结果，然后发送。
- 按住期间，菜单栏耳朵应保持放大状态（使用 `triggerVoiceEars(ttl:nil)`）；松开后会缩回。