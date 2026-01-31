---
title: 权限
summary: "macOS 权限持久化（TCC）与签名要求"
read_when:
  - 调试缺失或卡住的 macOS 权限提示
  - 打包或签名 macOS 应用程序
  - 修改 Bundle ID 或应用安装路径
---

<div id="macos-permissions-tcc">
  # macOS 权限（TCC）
</div>

macOS 的权限授予机制非常脆弱。TCC 会将某项权限授予绑定到应用的代码签名、bundle identifier，以及在磁盘上的路径。如果其中任意一项发生变化，macOS 就会将该应用视为一个新应用，并可能丢弃或隐藏权限请求提示。

<div id="requirements-for-stable-permissions">
  ## 权限稳定所需条件
</div>

- 相同路径：从固定位置运行应用程序（对于 OpenClaw，为 `dist/OpenClaw.app`）。
- 相同 bundle 标识符：更改 bundle ID 会创建一个新的权限身份标识。
- 已签名应用：未签名或临时（ad-hoc）签名的构建无法持久保留权限。
- 一致的签名：使用正式的 Apple Development 或 Developer ID 证书，
  以便在多次重建中保持签名稳定。

临时（ad-hoc）签名会在每次构建时生成一个新的身份标识。macOS 会遗忘之前授予的权限，而且权限弹窗可能会完全消失，直到清理掉这些陈旧条目为止。

<div id="recovery-checklist-when-prompts-disappear">
  ## 权限提示消失时的恢复检查清单
</div>

1. 退出应用。
2. 在“系统设置”-&gt;“隐私与安全性”中移除该应用的条目。
3. 从相同路径重新启动应用并重新授予权限。
4. 如果权限提示仍未出现，使用 `tccutil` 重置 TCC 条目后再试一次。
5. 某些权限提示只有在完全重启 macOS 后才会再次出现。

重置示例（按需替换 bundle ID）：

```bash
sudo tccutil reset Accessibility bot.molt.mac
sudo tccutil reset ScreenCapture bot.molt.mac
sudo tccutil reset AppleEvents
```

如果你在测试权限，一定要使用正式证书进行签名。Ad-hoc 构建只适用于本地快速运行且对权限无所谓的情况。
