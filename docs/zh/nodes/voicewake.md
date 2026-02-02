---
title: Voicewake
summary: "全局语音唤醒词（由 Gateway 统一管理）及其在各节点之间的同步机制"
read_when:
  - 更改语音唤醒词行为或默认值时
  - 新增需要唤醒词同步的节点平台时
---

<div id="voice-wake-global-wake-words">
  # 语音唤醒（全局唤醒词）
</div>

OpenClaw 将**唤醒词视为由 Gateway 统一管理的单一全局列表**。

- **不支持按节点自定义唤醒词**。
- **任何节点/应用 UI 都可以编辑**该列表；更改由 Gateway 负责持久化并广播给所有节点/应用。
- 每台设备仍然保留自己的 **语音唤醒启用/禁用** 开关（本地交互体验和权限设置可能不同）。

<div id="storage-gateway-host">
  ## 存储（Gateway 所在主机）
</div>

唤醒词会存储在运行 Gateway 的机器上：

* `~/.openclaw/settings/voicewake.json`

数据结构：

```json
{ "triggers": ["openclaw", "claude", "computer"], "updatedAtMs": 1730000000000 }
```


<div id="protocol">
  ## 协议
</div>

<div id="methods">
  ### 方法
</div>

- `voicewake.get` → `{ triggers: string[] }`
- `voicewake.set` 带参数 `{ triggers: string[] }` → `{ triggers: string[] }`

注意：

- 触发词（triggers）会被标准化处理（去除首尾空白并丢弃空字符串）。空列表将回退为使用默认值。
- 为安全起见会施加限制（数量/长度上限）。

<div id="events">
  ### 事件
</div>

- `voicewake.changed` 负载 `{ triggers: string[] }`

接收方：

- 所有 WebSocket 客户端（macOS 应用、WebChat 等）
- 所有已连接的节点（iOS/Android），并且在节点建立连接时，会作为初始“当前状态”推送一次。

<div id="client-behavior">
  ## 客户端行为
</div>

<div id="macos-app">
  ### macOS 应用
</div>

- 使用全局列表来控制 `VoiceWakeRuntime` 的触发。
- 在 Voice Wake 设置中编辑 “Trigger words”（触发词）时，会调用 `voicewake.set`，并依赖广播机制让其他客户端保持同步。

<div id="ios-node">
  ### iOS 节点
</div>

- 使用全局列表进行 `VoiceWakeManager` 触发检测。
- 在「设置」中编辑唤醒词会通过 Gateway 的 WS 调用 `voicewake.set`，同时也会保持本地唤醒词检测的灵敏响应。

<div id="android-node">
  ### Android 节点
</div>

- 在设置中提供唤醒词编辑器。
- 通过 Gateway 的 WS 调用 `voicewake.set`，使这些修改在各处保持同步。