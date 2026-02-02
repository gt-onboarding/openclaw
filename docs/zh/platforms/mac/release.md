---
title: 发布
summary: "OpenClaw macOS 发布检查清单（Sparkle 提要、打包、签名）"
read_when:
  - 制作或验证 OpenClaw macOS 发布版本时
  - 更新 Sparkle appcast 或提要资源时
---

<div id="openclaw-macos-release-sparkle">
  # OpenClaw macOS 发布（Sparkle）
</div>

此应用现已通过 Sparkle 支持自动更新。发布构建必须使用 Developer ID 签名、打包为 zip 压缩包，并附带已签名的 appcast 条目进行发布。

<div id="prereqs">
  ## 前置条件
</div>

- 已安装 Developer ID Application 证书（示例：`Developer ID Application: <Developer Name> (<TEAMID>)`）。
- 在环境中将 Sparkle 私钥路径设置为 `SPARKLE_PRIVATE_KEY_FILE`（指向你的 Sparkle ed25519 私钥；公钥已写入 Info.plist）。如果未设置，请检查 `~/.profile`。
- 已准备好用于 `xcrun notarytool` 的公证凭据（Keychain 配置文件或 API key），以便生成通过 Gatekeeper 校验的安全 DMG/zip 分发包。
  - 我们使用名为 `openclaw-notary` 的 Keychain 配置文件，它由你在 shell 配置文件中定义的 App Store Connect API key 环境变量创建：
    - `APP_STORE_CONNECT_API_KEY_P8`, `APP_STORE_CONNECT_KEY_ID`, `APP_STORE_CONNECT_ISSUER_ID`
    - `echo "$APP_STORE_CONNECT_API_KEY_P8" | sed 's/\\n/\n/g' > /tmp/openclaw-notary.p8`
    - `xcrun notarytool store-credentials "openclaw-notary" --key /tmp/openclaw-notary.p8 --key-id "$APP_STORE_CONNECT_KEY_ID" --issuer "$APP_STORE_CONNECT_ISSUER_ID"`
- 已安装 `pnpm` 依赖（`pnpm install --config.node-linker=hoisted`）。
- Sparkle 工具会通过 SwiftPM 自动获取，路径为 `apps/macos/.build/artifacts/sparkle/Sparkle/bin/`（`sign_update`、`generate_appcast` 等）。

<div id="build-package">
  ## 构建与打包
</div>

注意：

* `APP_BUILD` 映射到 `CFBundleVersion`/`sparkle:version`；保持其为纯数字且严格递增（不要使用 `-beta`），否则 Sparkle 会将其视为相等。
* 默认为当前架构（`$(uname -m)`）。对于发布版/通用构建，设置 `BUILD_ARCHS="arm64 x86_64"`（或 `BUILD_ARCHS=all`）。
* 使用 `scripts/package-mac-dist.sh` 生成发布产物（zip + DMG + 公证）。本地/开发环境打包使用 `scripts/package-mac-app.sh`。

```bash
# 从仓库根目录执行;设置发布 ID 以启用 Sparkle 更新源。
# APP_BUILD 必须是数字且单调递增,供 Sparkle 进行版本比较。
BUNDLE_ID=bot.molt.mac \
APP_VERSION=2026.1.27-beta.1 \
APP_BUILD="$(git rev-list --count HEAD)" \
BUILD_CONFIG=release \
SIGN_IDENTITY="Developer ID Application: <Developer Name> (<TEAMID>)" \
scripts/package-mac-app.sh

# 打包为 zip 用于分发(包含资源分支以支持 Sparkle 增量更新)
ditto -c -k --sequesterRsrc --keepParent dist/OpenClaw.app dist/OpenClaw-2026.1.27-beta.1.zip

# 可选:同时构建样式化的 DMG 供用户使用(拖拽到 /Applications)
scripts/create-dmg.sh dist/OpenClaw.app dist/OpenClaw-2026.1.27-beta.1.dmg

# 推荐:构建 + 公证/装订 zip + DMG
# 首先,创建一次钥匙串配置文件:
#   xcrun notarytool store-credentials "openclaw-notary" \
#     --apple-id "<apple-id>" --team-id "<team-id>" --password "<app-specific-password>"
NOTARIZE=1 NOTARYTOOL_PROFILE=openclaw-notary \
BUNDLE_ID=bot.molt.mac \
APP_VERSION=2026.1.27-beta.1 \
APP_BUILD="$(git rev-list --count HEAD)" \
BUILD_CONFIG=release \
SIGN_IDENTITY="Developer ID Application: <Developer Name> (<TEAMID>)" \
scripts/package-mac-dist.sh

# 可选:随发布版本一起提供 dSYM
ditto -c -k --keepParent apps/macos/.build/release/OpenClaw.app.dSYM dist/OpenClaw-2026.1.27-beta.1.dSYM.zip
```


<div id="appcast-entry">
  ## Appcast 条目
</div>

使用发行说明生成器，以便 Sparkle 能渲染带格式的 HTML 说明内容：

```bash
SPARKLE_PRIVATE_KEY_FILE=/path/to/ed25519-private-key scripts/make_appcast.sh dist/OpenClaw-2026.1.27-beta.1.zip https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml
```

从 `CHANGELOG.md` 生成 HTML 格式的发行说明（通过 [`scripts/changelog-to-html.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/changelog-to-html.sh)），并将其嵌入到 appcast 条目中。
在发布时，将更新后的 `appcast.xml` 与发布资产（zip 文件和 dSYM）一同提交到仓库中。


<div id="publish-verify">
  ## 发布与验证
</div>

- 将 `OpenClaw-2026.1.27-beta.1.zip`（以及 `OpenClaw-2026.1.27-beta.1.dSYM.zip`）上传到标签 `v2026.1.27-beta.1` 对应的 GitHub Release。
- 确认原始 appcast URL 与内置 feed 一致：`https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml`。
- 基本检查（sanity checks）：
  - `curl -I https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml` 返回 200。
  - 资源上传完成后，`curl -I <enclosure url>` 返回 200。
  - 在之前的公开构建版本上，从 About 选项卡中运行 “Check for Updates…”（检查更新…），并验证 Sparkle 能干净地安装新的构建版本。

完成标准（Definition of done）：已签名的应用和 appcast 均已发布，从已安装的旧版本触发的更新流程工作正常，并且发布资源已附加到对应的 GitHub Release。