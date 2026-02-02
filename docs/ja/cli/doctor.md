---
title: Doctor
summary: "`openclaw doctor` の CLI リファレンス（ヘルスチェック + ガイド付き修復）"
read_when:
  - 接続や認証に問題があり、ガイド付きの修復手順を実行したいとき
  - アップデート後に簡易な健全性チェック（サニティチェック）を行いたいとき
---

<div id="openclaw-doctor">
  # `openclaw doctor`
</div>

Gateway とチャネルのヘルスチェックおよび簡易な問題修正を行います。

関連:

* トラブルシューティング: [Troubleshooting](/ja/gateway/troubleshooting)
* セキュリティ: [Security](/ja/gateway/security)

<div id="examples">
  ## 使用例
</div>

```bash
openclaw doctor
openclaw doctor --repair
openclaw doctor --deep
```

Notes:

* 対話的なプロンプト（キーチェーンや OAuth の修復など）は、標準入力が TTY であり、かつ `--non-interactive` が **指定されていない** 場合にのみ実行されます。ヘッドレス実行（cron、Telegram、ターミナルなしなど）ではプロンプトはスキップされます。
* `--fix`（`--repair` のエイリアス）は、`~/.openclaw/openclaw.json.bak` にバックアップを書き出し、未知の設定キーを削除して、削除したキーを一覧表示します。

<div id="macos-launchctl-env-overrides">
  ## macOS: `launchctl` 環境変数の上書き
</div>

以前に `launchctl setenv OPENCLAW_GATEWAY_TOKEN ...`（または `...PASSWORD`）を実行していた場合、その値が設定ファイルの内容を上書きし、継続的に「unauthorized」エラーが発生する原因になることがあります。

```bash
launchctl getenv OPENCLAW_GATEWAY_TOKEN
launchctl getenv OPENCLAW_GATEWAY_PASSWORD

launchctl unsetenv OPENCLAW_GATEWAY_TOKEN
launchctl unsetenv OPENCLAW_GATEWAY_PASSWORD
```
