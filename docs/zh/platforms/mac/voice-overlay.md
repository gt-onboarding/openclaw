---
title: 语音叠加
summary: "当唤醒词与按键说话（push-to-talk）重叠时的语音叠加生命周期"
read_when:
  - 调整语音叠加行为
---

<div id="voice-overlay-lifecycle-macos">
  # 语音浮层生命周期（macOS）
</div>

面向读者：macOS 应用贡献者。目标：在唤醒词与按键发言（push-to-talk）重叠时确保语音浮层行为可预测。

<div id="current-intent">
  ### 当前意图
</div>

- 如果叠加层已经通过唤醒词显示，用户又按下热键，则热键会话会*继承*现有文本，而不是重置。按住热键期间，叠加层保持显示。用户松开时：如果有去除首尾空白后的文本则发送，否则关闭叠加层。
- 仅使用唤醒词时，在检测到静音后仍会自动发送；按键说话在松开时立即发送。

<div id="implemented-dec-9-2025">
  ### 已实现（2025 年 12 月 9 日）
</div>

- 叠加层会话现在为每次捕获（唤醒词或按键说话）携带一个令牌。当令牌不匹配时，partial/final/send/dismiss/level 更新会被丢弃，从而避免使用陈旧的回调。
- 按键说话会将任何可见的叠加层文本作为前缀（因此在唤醒叠加层显示时按下热键会保留现有文本，并在其后追加新的语音内容）。它最多会等待 1.5 秒以获取最终转录结果，否则会回退到当前文本。
- 提示音 / 叠加层日志以 `info` 级别输出，类别为 `voicewake.overlay`、`voicewake.ptt` 和 `voicewake.chime`（会话开始、partial、final、send、dismiss、提示音原因）。

<div id="next-steps">
  ### 后续步骤
</div>

1. **VoiceSessionCoordinator（actor）**
   - 同一时间只拥有一个 `VoiceSession`。
   - API（基于 token）：`beginWakeCapture`, `beginPushToTalk`, `updatePartial`, `endCapture`, `cancel`, `applyCooldown`。
   - 丢弃携带过期 token 的回调（防止旧的识别器重新打开覆盖层）。
2. **VoiceSession（model）**
   - 字段：`token`，`source`（wakeWord|pushToTalk），已提交/临时文本，提示音标志，计时器（自动发送、空闲），`overlayMode`（display|editing|sending），冷却截止时间。
3. **Overlay 绑定**
   - `VoiceSessionPublisher`（`ObservableObject`）将当前活动会话镜像到 SwiftUI。
   - `VoiceWakeOverlayView` 仅通过 publisher 渲染；它绝不会直接修改全局单例。
   - 覆盖层用户操作（`sendNow`, `dismiss`, `edit`）携带会话 token 回调到 coordinator。
4. **统一发送路径**
   - 在 `endCapture` 时：如果裁剪后的文本为空 → 关闭；否则调用 `performSend(session:)`（播放一次发送提示音、转发并关闭）。
   - 按住说话：无延迟；唤醒词：可选的自动发送延迟。
   - 在按住说话结束后，对唤醒词运行时应用一个短冷却时间，以避免唤醒词立即再次触发。
5. **日志记录**
   - Coordinator 在子系统 `bot.molt` 中发出 `.info` 日志，类别为 `voicewake.overlay` 和 `voicewake.chime`。
   - 关键事件：`session_started`, `adopted_by_push_to_talk`, `partial`, `finalized`, `send`, `dismiss`, `cancel`, `cooldown`。

<div id="debugging-checklist">
  ### 调试检查清单
</div>

- 在复现卡住不消失的语音悬浮窗时实时查看日志：

  ```bash
  sudo log stream --predicate 'subsystem == "bot.molt" AND category CONTAINS "voicewake"' --level info --style compact
  ```
- 验证只有一个活动会话令牌；过期的回调应由协调器丢弃。
- 确保在按住说话松开时始终使用活动令牌调用 `endCapture`；如果文本为空，此时应触发 `dismiss`，且无提示音也不发送。

<div id="migration-steps-suggested">
  ### 迁移步骤（建议）
</div>

1. 添加 `VoiceSessionCoordinator`、`VoiceSession` 和 `VoiceSessionPublisher`。
2. 重构 `VoiceWakeRuntime`，使其负责创建/更新/结束会话，而不是直接操作 `VoiceWakeOverlayController`。
3. 重构 `VoicePushToTalk` 以接管现有会话，并在释放时调用 `endCapture`；应用运行时冷却机制。
4. 将 `VoiceWakeOverlayController` 接入 publisher；从 runtime/PTT 中移除对它的直接调用。
5. 为会话接管、冷却机制以及空文本关闭行为添加集成测试。