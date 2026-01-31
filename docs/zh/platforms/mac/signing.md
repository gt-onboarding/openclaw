---
title: 签名
summary: "为打包脚本生成的 macOS 调试版本进行签名的步骤"
read_when:
  - 构建或签名 macOS 调试版本时
---

<div id="mac-signing-debug-builds">
  # mac 签名（调试构建）
</div>

此应用通常通过 [`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) 构建，该脚本现在会：

* 设置一个稳定的调试 bundle 标识符：`ai.openclaw.mac.debug`
* 使用该 bundle id 写入 Info.plist（可通过 `BUNDLE_ID=...` 覆盖）
* 调用 [`scripts/codesign-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/codesign-mac-app.sh) 对主二进制文件和 app bundle 进行签名，使 macOS 将每次重建视为同一个已签名 bundle，并保留 TCC 权限（通知、辅助功能、屏幕录制、麦克风、语音）。如需稳定的权限行为，请使用真实的签名身份；ad-hoc 为自选且脆弱（参见 [macOS permissions](/zh/platforms/mac/permissions)）。
* 默认使用 `CODESIGN_TIMESTAMP=auto`，为 Developer ID 签名启用受信任时间戳。将 `CODESIGN_TIMESTAMP=off` 以关闭时间戳，从而跳过时间戳步骤（适用于离线调试构建）。
* 向 Info.plist 注入构建元数据：`OpenClawBuildTimestamp`（UTC）和 `OpenClawGitCommit`（短哈希），以便 “About” 面板可以显示构建信息、git 信息以及调试/发布通道。
* **打包需要 Node 22+**：脚本会运行 TypeScript 构建和 Control UI 构建。
* 从环境中读取 `SIGN_IDENTITY`。在你的 shell rc 中添加 `export SIGN_IDENTITY="Apple Development: Your Name (TEAMID)"`（或你的 Developer ID Application 证书）以始终使用你的证书进行签名。ad-hoc 签名需要通过 `ALLOW_ADHOC_SIGNING=1` 或 `SIGN_IDENTITY="-"` 显式启用（不推荐用于权限测试）。
* 在签名后运行 Team ID 审核，如果 app bundle 内任意 Mach-O 使用不同的 Team ID 签名，则会失败。将环境变量 `SKIP_TEAM_ID_CHECK` 设为 1 以跳过该检查。

<div id="usage">
  ## 使用方法
</div>

```bash
# 从仓库根目录执行
scripts/package-mac-app.sh               # 自动选择身份；未找到时报错
SIGN_IDENTITY="Developer ID Application: Your Name" scripts/package-mac-app.sh   # 使用真实证书
ALLOW_ADHOC_SIGNING=1 scripts/package-mac-app.sh    # 临时签名（权限不会持久化）
SIGN_IDENTITY="-" scripts/package-mac-app.sh        # 显式临时签名（同样限制）
DISABLE_LIBRARY_VALIDATION=1 scripts/package-mac-app.sh   # 仅开发环境使用的 Sparkle Team ID 不匹配规避方案
```

<div id="ad-hoc-signing-note">
  ### 临时签名说明
</div>

使用 `SIGN_IDENTITY="-"`（ad-hoc 临时签名）进行签名时，脚本会自动禁用 **Hardened Runtime**（`--options runtime`）。这是为了避免应用在尝试加载未使用相同 Team ID 的内嵌框架（例如 Sparkle）时发生崩溃。临时签名还会破坏 TCC 权限的持久化；恢复步骤参见 [macOS permissions](/zh/platforms/mac/permissions)。

<div id="build-metadata-for-about">
  ## About 页的构建元数据
</div>

`package-mac-app.sh` 会将以下元数据写入应用包：

* `OpenClawBuildTimestamp`：打包时的 ISO8601 UTC 时间戳
* `OpenClawGitCommit`：简短的 git 提交哈希（如不可用则为 `unknown`）

About 标签页会读取这些键，以显示版本号、构建日期、git 提交信息，以及当前是否为调试构建（通过 `#if DEBUG` 判断）。代码变更后请重新运行打包脚本以刷新这些值。

<div id="why">
  ## 原因
</div>

TCC 权限是同时绑定到 bundle identifier *和* 代码签名上的。未签名的调试构建在每次构建时 UUID 都会变化，导致 macOS 在每次重新构建后都忘记之前授予的权限。对二进制文件进行签名（默认使用临时 ad‑hoc 签名）并保持固定的 bundle id 和路径（`dist/OpenClaw.app`），可以在不同构建之间保留这些授权，与 VibeTunnel 的做法一致。