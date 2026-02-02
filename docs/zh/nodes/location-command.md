---
title: 位置命令
summary: "用于节点的 location 命令（location.get）、权限模式以及后台行为"
read_when:
  - 正在添加位置节点支持或权限 UI
  - 正在设计后台定位与推送流程
---

<div id="location-command-nodes">
  # location 命令（节点）
</div>

<div id="tldr">
  ## 要点速览
</div>

- `location.get` 是一个节点命令（通过 `node.invoke` 调用）。
- 默认关闭。
- 在设置中通过选择器进行配置：关闭 / 使用期间 / 始终。
- 另有独立开关：精确位置。

<div id="why-a-selector-not-just-a-switch">
  ## 为什么要用选择器（而不只是开关）
</div>

操作系统权限是多层级的。我们可以在应用内提供一个选择器，但最终是否授予权限仍由操作系统决定。

- iOS/macOS：用户可以在系统弹窗/设置中选择 **While Using** 或 **Always**。App 可以请求提升权限，但操作系统可能要求用户前往设置中手动更改。
- Android：后台定位是单独的权限；在 Android 10+ 上通常需要通过设置页面授权。
- 精确位置是单独的授权（iOS 14+ 的 “Precise”，Android 的 “fine” vs “coarse”）。

UI 中的选择器决定我们请求的模式；实际授予的权限则由操作系统设置决定。

<div id="settings-model">
  ## 设置模型
</div>

针对每个节点设备：

- `location.enabledMode`: `off | whileUsing | always`
- `location.preciseEnabled`: bool

UI 行为：

- 选择 `whileUsing` 时会请求前台权限。
- 选择 `always` 时会先确保已获得 `whileUsing` 权限，然后请求后台权限（如有需要，则引导用户前往系统设置）。
- 如果操作系统拒绝了所请求的权限级别，则回退到已授予的最高级别，并显示状态。

<div id="permissions-mapping-nodepermissions">
  ## 权限映射表（node.permissions）
</div>

可选。macOS 节点会通过权限映射表上报 `location`；iOS/Android 可能会省略该字段。

<div id="command-locationget">
  ## 命令：`location.get`
</div>

通过 `node.invoke` 调用。

参数（建议）：

```json
{
  "timeoutMs": 10000,
  "maxAgeMs": 15000,
  "desiredAccuracy": "coarse|balanced|precise"
}
```

响应负载：

```json
{
  "lat": 48.20849,
  "lon": 16.37208,
  "accuracyMeters": 12.5,
  "altitudeMeters": 182.0,
  "speedMps": 0.0,
  "headingDeg": 270.0,
  "timestamp": "2026-01-03T12:34:56.000Z",
  "isPrecise": true,
  "source": "gps|wifi|cell|unknown"
}
```

错误（稳定错误码）：

* `LOCATION_DISABLED`: 定位功能已关闭。
* `LOCATION_PERMISSION_REQUIRED`: 缺少所请求模式所需的权限。
* `LOCATION_BACKGROUND_UNAVAILABLE`: 应用在后台运行，但只允许“使用期间”访问。
* `LOCATION_TIMEOUT`: 未能在限定时间内完成定位。
* `LOCATION_UNAVAILABLE`: 系统故障 / 无可用定位提供方。


<div id="background-behavior-future">
  ## 后台行为（未来）
</div>

目标：即使节点在后台时，模型也可以请求位置信息，但仅在以下情况下：

- 用户选择了 **Always**。
- 操作系统授予了后台定位权限。
- App 被允许为获取位置信息在后台运行（iOS 后台定位模式 / Android 前台服务或特殊许可）。

推送触发流程（未来）：

1) Gateway 向节点发送推送通知（静默推送或 FCM data）。
2) 节点被短暂唤醒并向设备请求位置信息。
3) 节点将数据负载转发给 Gateway。

注意：

- iOS：需要 Always 权限和后台定位模式。静默推送可能会被限流；应预期间歇性失败。
- Android：后台定位可能需要前台服务；否则预计会被拒绝。

<div id="modeltooling-integration">
  ## 模型 / 工具集成
</div>

- 工具接口：`nodes` 工具新增 `location_get` 操作（需要节点）。
- CLI：`openclaw nodes location get --node <id>`。
- Agent 代理使用规范：仅在用户已启用位置信息且清楚理解该操作的 scope 时才调用。

<div id="ux-copy-suggested">
  ## UX 文案（建议）
</div>

- Off: “位置共享已禁用。”
- While Using: “仅在 OpenClaw 打开时共享位置。”
- Always: “始终允许后台定位。需要系统权限。”
- Precise: “使用精确 GPS 定位。关闭此选项则仅共享大致位置。”