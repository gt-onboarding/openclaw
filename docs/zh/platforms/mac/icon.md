---
title: 图标
summary: "macOS 上 OpenClaw 菜单栏图标的状态和动画"
read_when:
  - 更改菜单栏图标的行为
---

<div id="menu-bar-icon-states">
  # 菜单栏图标状态
</div>

作者：steipete · 更新日期：2025-12-06 · 适用范围：macOS app (`apps/macos`)

- **空闲（Idle）：** 正常图标动画（眨眼，偶尔轻微摆动）。
- **暂停（Paused）：** 状态项使用 `appearsDisabled`；无动画。
- **语音触发（大耳朵）：** 语音唤醒检测器在听到唤醒词时调用 `AppState.triggerVoiceEars(ttl: nil)`，在语句被捕获期间保持 `earBoostActive=true`。耳朵放大至 1.9 倍，并加上圆形耳洞以提升辨识度，然后在 1 秒静默后通过 `stopVoiceEars()` 关闭。仅由应用内语音管线触发。
- **工作中（agent 运行中）：** `AppState.isWorking=true` 驱动“尾巴/腿小跑”微动效：更快的腿部摆动和轻微偏移，表示工作正在进行中。目前只在 WebChat 智能体运行时切换；当你接好其他长任务时，也为它们加上相同的切换逻辑。

接线点（Wiring points）

- 语音唤醒：在触发时由 runtime/tester 调用 `AppState.triggerVoiceEars(ttl: nil)`，并在 1 秒静默后调用 `stopVoiceEars()`，与语音捕获窗口对齐。
- Agent 活动：在工作区间前后调用 `AppStateStore.shared.setWorking(true/false)`（WebChat 智能体调用中已实现）。保持区间尽量短，并在 `defer` 块中重置，避免动画卡死。

形状与尺寸（Shapes & sizes）

- 基础图标在 `CritterIconRenderer.makeIcon(blink:legWiggle:earWiggle:earScale:earHoles:)` 中绘制。
- 耳朵缩放默认值为 `1.0`；语音增强会设置 `earScale=1.9`，并切换 `earHoles=true`，但不会改变整体画布尺寸（18×18 pt 模板图像渲染到 36×36 px Retina 后备存储）。
- 小跑（Scurry）使用最高约 1.0 的腿部摆动，并带有小幅水平抖动；该效果会叠加在任意现有的空闲摆动之上。

行为说明（Behavioral notes）

- 不提供用于耳朵/工作状态的外部 CLI/broker 开关；保持由应用自身信号内部驱动，以避免意外抖动。
- 将 TTL 保持在较短时间（&lt;10s），这样当任务挂起时，图标也能尽快恢复到基线状态。