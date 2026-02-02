---
title: ログ
summary: "OpenClaw のログ: ローテーションされる診断用ファイルログ + Unified Log のプライバシーフラグ"
read_when:
  - macOS のログ取得やプライベートデータのログ出力を調査しているとき
  - 音声ウェイク／セッションのライフサイクル問題をデバッグしているとき
---

<div id="logging-macos">
  # ログ記録 (macOS)
</div>

<div id="rolling-diagnostics-file-log-debug-pane">
  ## ローリング診断ファイルログ（Debug ペイン）
</div>

OpenClaw は macOS アプリのログを swift-log（デフォルトでは unified logging）経由で出力し、必要に応じてローカルのローテーション（ローリング）ファイルログとしてディスクに書き出して、永続的に記録できます。

- 詳細度: **Debug ペイン → Logs → App logging → Verbosity**
- 有効化: **Debug ペイン → Logs → App logging → “Write rolling diagnostics log (JSONL)”**
- 保存場所: `~/Library/Logs/OpenClaw/diagnostics.jsonl`（自動でローテーションされ、古いファイルには `.1`、`.2` などのサフィックスが付きます）
- クリア: **Debug ペイン → Logs → App logging → “Clear”**

注意:

- これは **デフォルトでは無効** です。積極的にデバッグしている間だけ有効化してください。
- このファイルは機微情報を含みうるものとして扱い、レビューなしに共有しないでください。

<div id="unified-logging-private-data-on-macos">
  ## macOS における Unified Logging の機密データ
</div>

Unified Logging は、サブシステムが `privacy -off` にオプトインしない限り、ほとんどのペイロードをマスクします。Peter 氏による macOS の [logging privacy shenanigans](https://steipete.me/posts/2025/logging-privacy-shenanigans)（2025 年）の解説によると、これは `/Library/Preferences/Logging/Subsystems/` 内の、サブシステム名をキーとする plist ファイルによって制御されます。新規のログエントリのみがこのフラグを反映するため、問題を再現する前に有効化してください。

<div id="enable-for-openclaw-botmolt">
  ## OpenClaw（`bot.molt`）で有効化する
</div>

* まず plist を一時ファイルに書き出し、その後 root 権限でアトミックにインストールします:

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

* 再起動は不要です。`logd` はこのファイルをすぐに検知しますが、プライベートなペイロードが含まれるのは新しく書き込まれるログ行のみです。
* 既存のヘルパーを使って、よりリッチな出力を表示します（例: `./scripts/clawlog.sh --category WebChat --last 5m`）。


<div id="disable-after-debugging">
  ## デバッグ後に無効化する
</div>

- オーバーライド設定を削除します: `sudo rm /Library/Preferences/Logging/Subsystems/bot.molt.plist`。
- 必要に応じて `sudo log config --reload` を実行し、logd にオーバーライドを即座に破棄させます。
- このログ出力には電話番号やメッセージ本文が含まれる可能性があることを忘れないでください。追加の詳細が本当に必要なあいだだけ、この plist ファイルを残しておいてください。