---
title: 设备型号
summary: "OpenClaw 如何在 macOS 应用中将 Apple 设备型号标识符映射为友好名称。"
read_when:
  - 更新设备型号标识符映射或 NOTICE/许可文件时
  - 更改 Instances UI 中设备名称显示方式时
---

<div id="device-model-database-friendly-names">
  # 设备型号数据库（友好名称）
</div>

macOS 配套应用会在 **Instances** UI 中，通过将 Apple 型号标识符（例如 `iPad16,6`、`Mac16,6`）映射为更易读的名称，来显示更友好的 Apple 设备型号名称。

该映射以内置的 JSON 形式提供，路径为：

- `apps/macos/Sources/OpenClaw/Resources/DeviceModels/`

<div id="data-source">
  ## 数据源
</div>

目前我们从以下 MIT 许可的仓库引入映射关系：

- `kyle-seongwoo-jun/apple-device-identifiers`

为保证构建具有确定性，这些 JSON 文件会锁定到特定的上游提交版本（并记录在 `apps/macos/Sources/OpenClaw/Resources/DeviceModels/NOTICE.md` 中）。

<div id="updating-the-database">
  ## 更新数据库
</div>

1. 选择要固定的上游提交（各选一个，分别用于 iOS 和 macOS）。
2. 在 `apps/macos/Sources/OpenClaw/Resources/DeviceModels/NOTICE.md` 中更新对应的提交哈希。
3. 重新下载固定到这些提交的 JSON 文件：

```bash
IOS_COMMIT="<ios-device-identifiers.json 的 commit SHA>"
MAC_COMMIT="<mac-device-identifiers.json 的 commit SHA>"

curl -fsSL "https://raw.githubusercontent.com/kyle-seongwoo-jun/apple-device-identifiers/${IOS_COMMIT}/ios-device-identifiers.json" \
  -o apps/macos/Sources/OpenClaw/Resources/DeviceModels/ios-device-identifiers.json

curl -fsSL "https://raw.githubusercontent.com/kyle-seongwoo-jun/apple-device-identifiers/${MAC_COMMIT}/mac-device-identifiers.json" \
  -o apps/macos/Sources/OpenClaw/Resources/DeviceModels/mac-device-identifiers.json
```

4. 确保 `apps/macos/Sources/OpenClaw/Resources/DeviceModels/LICENSE.apple-device-identifiers.txt` 仍然与上游版本一致（如果上游许可证有变更，则替换该文件）。
5. 确认 macOS 应用能够无警告完成构建：

```bash
swift build --package-path apps/macos
```
