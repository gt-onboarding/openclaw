---
title: リモート
summary: "SSH 経由でリモートの OpenClaw Gateway を制御するための macOS アプリのフロー"
read_when:
  - リモート Mac 制御の設定やデバッグを行うとき
---

<div id="remote-openclaw-macos-remote-host">
  # リモート OpenClaw（macOS ⇄ リモートホスト）
</div>

このフローでは、macOS アプリが、別のホスト（デスクトップ／サーバー）上で動作している OpenClaw Gateway をフルリモートで制御するコントローラとして機能します。これはアプリの **Remote over SSH**（リモート実行）機能です。ヘルスチェック、Voice Wake の転送、Web Chat を含むすべての機能は、*Settings → General* で設定した同じリモート SSH 設定を再利用します。

<div id="modes">
  ## モード
</div>

* **ローカル (この Mac)**: すべてがこの Mac 上で動作します。SSH は使用しません。
* **SSH 経由のリモート (デフォルト)**: OpenClaw コマンドはリモートホスト上で実行されます。macOS アプリは、`-o BatchMode` と選択したアイデンティティ／鍵、およびローカルポートフォワーディングを指定して SSH 接続を確立します。
* **リモート直接接続 (ws/wss)**: SSH トンネルは使用しません。macOS アプリは Gateway の URL に直接接続します（例: Tailscale Serve や公開された HTTPS リバースプロキシ経由）。

<div id="remote-transports">
  ## リモートトランスポート
</div>

リモートモードでは次の 2 種類のトランスポート方式をサポートします:

* **SSH トンネル**（デフォルト）: `ssh -N -L ...` を使用して Gateway のポートを localhost にフォワードします。トンネルがループバックであるため、Gateway からはノードの IP は `127.0.0.1` として見えます。
* **ダイレクト（ws/wss）**: Gateway の URL に直接接続します。Gateway からはクライアントの実 IP アドレスが見えます。

<div id="prereqs-on-the-remote-host">
  ## リモートホスト側の前提条件
</div>

1. Node と pnpm をインストールし、OpenClaw CLI をビルド／インストールします（`pnpm install && pnpm build && pnpm link --global`）。
2. 非対話シェルでも `openclaw` が PATH に含まれていることを確認します（必要であれば `/usr/local/bin` や `/opt/homebrew/bin` にシンボリックリンクを作成します）。
3. SSH を鍵認証で受け付けるようにします。LAN 外からも安定して到達できるようにするため、**Tailscale** の IP アドレスの利用を推奨します。

<div id="macos-app-setup">
  ## macOS アプリのセットアップ
</div>

1. *Settings → General* を開きます。
2. **OpenClaw runs** の項目で **Remote over SSH** を選択し、次を設定します:
   * **Transport**: **SSH tunnel** または **Direct (ws/wss)**。
   * **SSH target**: `user@host`（任意で `:port` を追加）。
     * Gateway が同一 LAN 上にあり Bonjour でアドバタイズされている場合は、検出済みリストから選択するとこのフィールドが自動入力されます。
   * **Gateway URL**（Direct のみ）: `wss://gateway.example.ts.net`（ローカル/LAN 用には `ws://...`）。
   * **Identity file**（上級者向け）: 使用する鍵ファイルへのパス。
   * **Project root**（上級者向け）: コマンド実行に使用するリモート側のチェックアウトパス。
   * **CLI path**（上級者向け）: 実行可能な `openclaw` エントリポイント/バイナリへの任意のパス（アドバタイズされている場合は自動入力）。
3. **Test remote** をクリックします。成功した場合、リモートで `openclaw status --json` が正常に実行できていることを示します。失敗する場合は、ほとんどが PATH/CLI の問題であり、終了コード 127 の場合はリモート側で CLI が見つからないことを意味します。
4. 以後、ヘルスチェックと Web Chat は自動的にこの SSH トンネル経由で動作します。

<div id="web-chat">
  ## Web Chat
</div>

* **SSH トンネル**: Web Chat は、転送された WebSocket 制御ポート（デフォルト: 18789）経由で Gateway に接続します。
* **直接接続 (ws/wss)**: Web Chat は、設定された Gateway の URL に直接接続します。
* 別個の Web Chat 用 HTTP サーバーは、もう存在しません。

<div id="permissions">
  ## 権限
</div>

* リモートホストにはローカルと同じ TCC 承認（Automation、Accessibility、Screen Recording、Microphone、Speech Recognition、Notifications）が必要です。そのマシン上でオンボーディングを実行して、一度だけこれらを付与してください。
* ノードは `node.list` / `node.describe` を通じて自分の権限状態を公開し、エージェントが何を利用できるか把握できるようにします。

<div id="security-notes">
  ## セキュリティに関する注意事項
</div>

* リモートホスト上ではループバックインターフェースへのバインドを優先し、SSH または Tailscale 経由で接続してください。
* Gateway をループバック以外のインターフェースにバインドする場合は、必ずトークン／パスワードによる認証を必須にしてください。
* [Security](/ja/gateway/security) および [Tailscale](/ja/gateway/tailscale) を参照してください。

<div id="whatsapp-login-flow-remote">
  ## WhatsApp ログインフロー（リモート）
</div>

* **リモートホスト上で** `openclaw channels login --verbose` を実行します。スマートフォンの WhatsApp で QR コードをスキャンします。
* 認証の有効期限が切れた場合は、そのホスト上でログインを再度実行します。ヘルスチェックによってリンクの問題が検出されます。

<div id="troubleshooting">
  ## トラブルシューティング
</div>

* **exit 127 / not found**: 非ログインシェルで `openclaw` が PATH に含まれていません。`/etc/paths`、使用しているシェルの rc に追加するか、`/usr/local/bin` または `/opt/homebrew/bin` へのシンボリックリンクを作成してください。
* **Health probe failed**: SSH で到達可能か、PATH が正しく設定されているか、Baileys にログイン済みか（`openclaw status --json`）を確認してください。
* **Web Chat stuck**: リモートホスト上で Gateway が動作していること、および転送しているポートが Gateway の WS ポートと一致していることを確認してください。UI には正常な WS 接続が必要です。
* **Node IP shows 127.0.0.1**: SSH トンネル経由の場合は想定どおりの動作です。Gateway に実際のクライアント IP を見せたい場合は、**Transport** を **Direct (ws/wss)** に切り替えてください。
* **Voice Wake**: トリガーフレーズはリモートモードで自動的に転送されるため、別途フォワーダーを用意する必要はありません。

<div id="notification-sounds">
  ## 通知音
</div>

`openclaw` と `node.invoke` を使ったスクリプトから、通知ごとに再生するサウンドを選択できます。例：

```bash
openclaw nodes notify --node <id> --title "Ping" --body "Remote gateway ready" --sound Glass
```

アプリにはグローバルな「デフォルトサウンド」トグルはもうありません。呼び出し側がリクエストごとにサウンド（または無音）を選択します。
