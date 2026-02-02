---
title: 認証監視
summary: "モデルプロバイダーの OAuth 有効期限を監視する"
read_when:
  - OAuth 有効期限の監視やアラートを設定するとき
  - Claude Code / Codex の OAuth リフレッシュチェックを自動化するとき
---

<div id="auth-monitoring">
  # 認証モニタリング
</div>

OpenClaw は `openclaw models status` を通じて OAuth の有効期限ヘルス（状態）を公開しています。自動化とアラートにはそれを使用してください。スクリプトは、電話ワークフロー向けの任意の補助的手段です。

<div id="preferred-cli-check-portable">
  ## 推奨: CLI チェック（ポータブル版）
</div>

```bash
openclaw models status --check
```

終了コード:

* `0`: OK
* `1`: 有効期限切れ、または認証情報が存在しない
* `2`: まもなく有効期限切れ（24時間以内）

これは cron や systemd からそのまま利用でき、追加のスクリプトは不要です。


<div id="optional-scripts-ops-phone-workflows">
  ## オプションスクリプト（運用 / モバイル向けワークフロー）
</div>

これらは `scripts/` 配下にあり、**オプション** です。Gateway ホストへの SSH アクセスを前提としており、systemd + Termux 向けに調整されています。

- `scripts/claude-auth-status.sh` は、`openclaw models status --json` を
  信頼できる情報源として使用するようになりました（CLI が利用できない場合は直接ファイルを read する方式にフォールバックします）。
  そのため、タイマーで利用するために `openclaw` が `PATH` に入っている状態を維持してください。
- `scripts/auth-monitor.sh`: cron/systemd タイマーのターゲット。アラートを送信します（ntfy または電話）。
- `scripts/systemd/openclaw-auth-monitor.{service,timer}`: systemd ユーザータイマー。
- `scripts/claude-auth-status.sh`: Claude Code + OpenClaw の認証チェッカー（full/json/simple）。
- `scripts/mobile-reauth.sh`: SSH 経由の対話的な再認証フロー。
- `scripts/termux-quick-auth.sh`: ワンタップのウィジェットによるステータス表示 + 認証用 URL を開く処理。
- `scripts/termux-auth-widget.sh`: フルガイド付きウィジェットフロー。
- `scripts/termux-sync-widget.sh`: Claude Code の認証情報を OpenClaw に同期。

モバイル経由の自動化や systemd タイマーが不要な場合、これらのスクリプトはスキップしてかまいません。