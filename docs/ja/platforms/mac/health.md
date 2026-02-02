---
title: ヘルスチェック
summary: "macOS アプリが Gateway/Baileys のヘルス状態をどのように報告するか"
read_when:
  - macOS アプリのヘルスインジケーターをデバッグするとき
---

<div id="health-checks-on-macos">
  # macOS でのヘルスチェック
</div>

リンク済みチャネルが正常に動作しているかを、メニューバーアプリから確認する方法を説明します。

<div id="menu-bar">
  ## メニューバー
</div>

* ステータスドットが Baileys のヘルス状態を反映するようになりました：
  * 緑: リンク済み + ソケットが最近オープンされた状態。
  * オレンジ: 接続中／再試行中。
  * 赤: ログアウト済み、またはプローブ失敗。
* 2 行目には &quot;linked · auth 12m&quot; のような表示、または失敗理由が表示されます。
* &quot;Run Health Check&quot; メニュー項目を選ぶと、その場でプローブが実行されます。

<div id="settings">
  ## 設定
</div>

* General タブに Health カードが追加され、リンク済み認証の経過時間、セッションストアのパス/件数、最終チェック時刻、直近のエラー/ステータスコード、および「ヘルスチェックを実行」「ログを表示」ボタンが表示されます。
* キャッシュされたスナップショットを使用するため、UI は即座に読み込まれ、オフライン時もスムーズにフォールバックします。
* **Channels タブ** には、WhatsApp/Telegram 向けのチャンネルステータスとコントロール（ログイン QR、ログアウト、疎通確認、直近の切断/エラー）が表示されます。

<div id="how-the-probe-works">
  ## プローブの動作
</div>

* アプリは約60秒ごとおよび要求に応じて、`ShellExecutor` を介して `openclaw health --json` を実行します。プローブは認証情報を読み込み、メッセージを送信せずにステータスを報告します。
* 表示のちらつきを避けるため、直近の正常なスナップショットと直近のエラーを別々にキャッシュし、それぞれのタイムスタンプを表示します。

<div id="when-in-doubt">
  ## 迷ったときは
</div>

* 迷った場合でも、[Gateway health](/ja/gateway/health) での CLI フロー（`openclaw status`、`openclaw status --deep`、`openclaw health --json`）を使って状態を確認でき、`tail` コマンドで `/tmp/openclaw/openclaw-*.log` を追いながら `web-heartbeat` / `web-reconnect` を確認できます。