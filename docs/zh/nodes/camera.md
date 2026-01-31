---
title: 摄像头
summary: "摄像头采集（iOS 节点 + macOS 应用），供智能体使用：照片（jpg）和短视频片段（mp4）"
read_when:
  - 在 iOS 节点或 macOS 上添加或修改摄像头采集功能
  - 扩展智能体可访问的 MEDIA 临时文件工作流
---

<div id="camera-capture-agent">
  # 摄像头采集（智能体）
</div>

OpenClaw 支持在智能体工作流中使用**摄像头采集**功能：

- **iOS 节点**（通过 Gateway 配对）：通过 `node.invoke` 采集**照片**（`jpg`）或**短视频片段**（`mp4`，可包含音频）。
- **Android 节点**（通过 Gateway 配对）：通过 `node.invoke` 采集**照片**（`jpg`）或**短视频片段**（`mp4`，可包含音频）。
- **macOS 应用**（通过 Gateway 的节点）：通过 `node.invoke` 采集**照片**（`jpg`）或**短视频片段**（`mp4`，可包含音频）。

所有摄像头访问均受**用户可控的设置**严格控制。

<div id="ios-node">
  ## iOS 节点
</div>

<div id="user-setting-default-on">
  ### 用户设置（默认开启）
</div>

- iOS 设置选项卡 → **Camera** → **Allow Camera**（`camera.enabled`）
  - 默认值：**开启**（若缺少该键，则视为已启用）。
  - 关闭时：`camera.*` 命令返回 `CAMERA_DISABLED`。

<div id="commands-via-gateway-nodeinvoke">
  ### 命令（通过 Gateway `node.invoke`）
</div>

- `camera.list`
  - 响应载荷：
    - `devices`: 由 `{ id, name, position, deviceType }` 组成的数组

- `camera.snap`
  - 参数：
    - `facing`: `front|back`（默认：`front`）
    - `maxWidth`: 数值（可选；在 iOS 节点上的默认值为 `1600`）
    - `quality`: `0..1`（可选；默认 `0.9`）
    - `format`: 当前为 `jpg`
    - `delayMs`: 数值（可选；默认 `0`）
    - `deviceId`: 字符串（可选；来自 `camera.list`）
  - 响应载荷：
    - `format: "jpg"`
    - `base64: "<...>"`
    - `width`, `height`
  - 载荷限制：照片会被重新压缩，以确保 base64 载荷小于 5 MB。

- `camera.clip`
  - 参数：
    - `facing`: `front|back`（默认：`front`）
    - `durationMs`: 数值（默认 `3000`，上限为 `60000`）
    - `includeAudio`: 布尔值（默认 `true`）
    - `format`: 当前为 `mp4`
    - `deviceId`: 字符串（可选；来自 `camera.list`）
  - 响应载荷：
    - `format: "mp4"`
    - `base64: "<...>"`
    - `durationMs`
    - `hasAudio`

<div id="foreground-requirement">
  ### 前台使用要求
</div>

与 `canvas.*` 一样，iOS 节点仅在**前台**允许执行 `camera.*` 命令。后台调用会返回 `NODE_BACKGROUND_UNAVAILABLE`。

<div id="cli-helper-temp-files-media">
  ### CLI 辅助工具（临时文件 + MEDIA）
</div>

获取附件的最简单方式是使用 CLI 辅助工具，它会将解码后的媒体写入一个临时文件，并输出 `MEDIA:<path>`。

示例：

```bash
openclaw nodes camera snap --node <id>               # 默认：前置 + 后置（2 条 MEDIA 行）
openclaw nodes camera snap --node <id> --facing front
openclaw nodes camera clip --node <id> --duration 3000
openclaw nodes camera clip --node <id> --no-audio
```

Notes:

* `nodes camera snap` 默认会同时使用**前后**摄像头，以便让智能体获取两个视角。
* 输出文件是临时文件（位于操作系统的临时目录中），除非你编写自己的封装。


<div id="android-node">
  ## Android 节点
</div>

<div id="user-setting-default-on">
  ### 用户设置（默认开启）
</div>

- Android 设置面板 → **Camera** → **Allow Camera** (`camera.enabled`)
  - 默认：**开启**（缺少该键时视为已启用）。
  - 当关闭时：`camera.*` 命令返回 `CAMERA_DISABLED`。

<div id="permissions">
  ### 权限
</div>

- Android 需要在运行时授予以下权限：
  - `CAMERA`：`camera.snap` 和 `camera.clip` 都需要。
  - `RECORD_AUDIO`：当 `includeAudio=true` 时，`camera.clip` 需要。

如果缺少权限，应用会在可能时弹出授权提示；如果被拒绝，`camera.*` 请求将失败并返回
`*_PERMISSION_REQUIRED` 错误。

### 前台要求

与 `canvas.*` 一样，Android 节点仅在**前台**时允许执行 `camera.*` 命令。在后台调用会返回 `NODE_BACKGROUND_UNAVAILABLE`。

<div id="payload-guard">
  ### 载荷保护
</div>

照片会被重新压缩，以将 Base64 载荷控制在 5 MB 以下。

<div id="macos-app">
  ## macOS 应用
</div>

<div id="user-setting-default-off">
  ### 用户设置（默认关闭）
</div>

macOS 配套应用中提供一个复选框：

- **Settings → General → Allow Camera**（`openclaw.cameraEnabled`）
  - 默认：**off**
  - 关闭时：相机请求会返回“Camera disabled by user”。

<div id="cli-helper-node-invoke">
  ### CLI 辅助工具（节点调用）
</div>

使用主 `openclaw` CLI 工具在 macOS 节点上调用相机命令。

示例：

```bash
openclaw nodes camera list --node <id>            # list camera ids
openclaw nodes camera snap --node <id>            # prints MEDIA:<path>
openclaw nodes camera snap --node <id> --max-width 1280
openclaw nodes camera snap --node <id> --delay-ms 2000
openclaw nodes camera snap --node <id> --device-id <id>
openclaw nodes camera clip --node <id> --duration 10s          # prints MEDIA:<path>
openclaw nodes camera clip --node <id> --duration-ms 3000      # 输出 MEDIA:<path> (旧版标志)
openclaw nodes camera clip --node <id> --device-id <id>
openclaw nodes camera clip --node <id> --no-audio
```

注意：

* `openclaw nodes camera snap` 默认使用 `maxWidth=1600`，除非另有指定。
* 在 macOS 上，`camera.snap` 会在预热/曝光稳定之后等待 `delayMs`（默认 2000ms）再进行抓拍。
* 照片数据会被重新压缩，以确保 base64 大小不超过 5 MB。


<div id="safety-practical-limits">
  ## 安全性与实际限制
</div>

- 访问相机和麦克风时会触发常规的操作系统权限提示（并且需要在 Info.plist 中提供用途说明字符串）。
- 为避免节点数据负载过大（base64 开销 + 消息大小限制），视频片段的时长目前被限制为 `<= 60s`。

<div id="macos-screen-video-os-level">
  ## macOS 屏幕录制（系统级）
</div>

对于 *屏幕* 录制（非摄像头），请使用 macOS 配套应用：

```bash
openclaw nodes screen record --node <id> --duration 10s --fps 15   # 打印 MEDIA:<path>
```

Notes:

* 需要 macOS **Screen Recording** 权限（TCC）。
