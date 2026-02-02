---
title: 日志
summary: "OpenClaw 日志：滚动诊断日志文件 + 统一日志隐私标志"
read_when:
  - 捕获 macOS 日志或排查与私有数据相关的日志问题
  - 调试语音唤醒/会话生命周期相关问题
---

<div id="logging-macos">
  # 日志记录（macOS）
</div>

<div id="rolling-diagnostics-file-log-debug-pane">
  ## 滚动诊断日志文件（Debug 面板）
</div>

OpenClaw 通过 swift-log 路由 macOS 应用日志（默认使用 unified logging 统一日志系统），并且在你需要持久化记录时，可以将本地的滚动日志文件写入磁盘。

- 日志详细程度：**Debug pane → Logs → App logging → Verbosity**
- 启用：**Debug pane → Logs → App logging → “Write rolling diagnostics log (JSONL)”**
- 位置：`~/Library/Logs/OpenClaw/diagnostics.jsonl`（自动轮转；旧文件会加上 `.1`、`.2` 等后缀）
- 清除：**Debug pane → Logs → App logging → “Clear”**

注意：

- 默认 **处于关闭状态**。仅在主动调试时启用。
- 将该文件视为敏感信息；未经审查不要共享。

<div id="unified-logging-private-data-on-macos">
  ## 在 macOS 上记录 Unified Logging 的私有数据
</div>

Unified Logging 会对大多数日志负载进行脱敏，除非某个 subsystem 显式启用 `privacy -off`。根据 Peter 在 2025 年关于 macOS [logging privacy shenanigans](https://steipete.me/posts/2025/logging-privacy-shenanigans) 的说明，这是由 `/Library/Preferences/Logging/Subsystems/` 中一个以 subsystem 名称为键的 plist 控制的。只有新的日志记录才会应用该标志，因此要在复现问题之前先启用它。

<div id="enable-for-openclaw-botmolt">
  ## 启用 OpenClaw（`bot.molt`）
</div>

* 先将 plist 写入临时文件，然后以 root 身份以原子方式安装它：

```bash
cat <<'EOF' >/tmp/bot.molt.plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>DEFAULT-OPTIONS</key>
    <dict>
        <key>Enable-Private-Data</key>
        <true/>
    </dict>
</dict>
</plist>
EOF
sudo install -m 644 -o root -g wheel /tmp/bot.molt.plist /Library/Preferences/Logging/Subsystems/bot.molt.plist
```

* 不需要重启；`logd` 会很快检测到该文件，但只有新的日志行会包含私有数据。
* 使用现有的辅助脚本查看更详细的输出，例如：`./scripts/clawlog.sh --category WebChat --last 5m`。


<div id="disable-after-debugging">
  ## 调试完成后禁用
</div>

- 删除覆盖配置：`sudo rm /Library/Preferences/Logging/Subsystems/bot.molt.plist`。
- 如有需要，运行 `sudo log config --reload`，强制 logd 立即丢弃该覆盖配置。
- 请注意，该日志输出中可能包含电话号码和消息正文；仅在你确实需要这些额外细节时才保留该 plist 文件。