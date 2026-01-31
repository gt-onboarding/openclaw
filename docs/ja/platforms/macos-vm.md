---
title: Macos Vm
summary: "分離環境や iMessage が必要な場合に、サンドボックス化された macOS VM（ローカルまたはホスト型）で OpenClaw を実行する"
read_when:
  - OpenClaw をメインの macOS 環境から分離したい
  - サンドボックス内で iMessage 連携（BlueBubbles）を使いたい
  - クローン可能かつリセット可能な macOS 環境を用意したい
  - ローカルとホスト型の macOS VM オプションを比較したい
---

<div id="openclaw-on-macos-vms-sandboxing">
  # macOS 仮想マシン上での OpenClaw（サンドボックス実行）
</div>

<div id="recommended-default-most-users">
  ## 推奨デフォルト（ほとんどのユーザー向け）
</div>

* 常時稼働の Gateway を低コストで運用するには、**小規模な Linux VPS** がおすすめです。[VPS hosting](/ja/vps) を参照してください。
* ブラウザ自動化用に完全な制御と **レジデンシャル IP アドレス** が必要な場合は、**専用ハードウェア**（Mac mini または Linux マシン）を使用します。多くのサイトはデータセンター IP をブロックするため、ローカルからのブラウジングの方がうまくいくことが多くなります。
* **ハイブリッド:** Gateway は安価な VPS 上で稼働させ、ブラウザ/UI 自動化が必要なときだけ Mac を **ノード** として接続します。[Nodes](/ja/nodes) と [Gateway remote](/ja/gateway/remote) を参照してください。

macOS 専用機能（iMessage/BlueBubbles）が必要な場合や、日常利用している Mac から厳密に分離したい場合は、macOS VM を使用してください。

<div id="macos-vm-options">
  ## macOS VM オプション
</div>

<div id="local-vm-on-your-apple-silicon-mac-lume">
  ### Apple Silicon Mac 上でのローカル VM（Lume）
</div>

既存の Apple Silicon Mac 上で [Lume](https://cua.ai/docs/lume) を使い、サンドボックス化された macOS VM 内で OpenClaw を実行します。

これにより次のことが可能になります:

* 分離されたフル macOS 環境（ホスト環境はクリーンなまま）
* BlueBubbles 経由の iMessage サポート（Linux/Windows では不可能）
* VM をクローンすることで即座にリセット可能
* 追加のハードウェアやクラウド費用が不要

<div id="hosted-mac-providers-cloud">
  ### ホスト型Macプロバイダー（クラウド）
</div>

クラウド上でmacOSを使いたい場合は、ホスト型Macプロバイダーも利用できます：

* [MacStadium](https://www.macstadium.com/)（ホスト型Mac）
* 他のホスト型Macベンダーも利用可能です。そのベンダーが提供する VM＋SSH ドキュメントに従ってください

macOS VM への SSH アクセスを確保できたら、以下のステップ6に進んでください。

***

<div id="quick-path-lume-experienced-users">
  ## クイック手順（Lume に慣れたユーザー向け）
</div>

1. Lume をインストールする
2. `lume create openclaw --os macos --ipsw latest`
3. Setup Assistant を完了し、Remote Login（SSH）を有効化する
4. `lume run openclaw --no-display`
5. SSH 接続して OpenClaw をインストールし、チャネルを設定する
6. 完了

***

<div id="what-you-need-lume">
  ## 必要なもの（Lume）
</div>

* Apple Silicon Mac (M1/M2/M3/M4)
* ホストの macOS Sequoia 以降
* VM 1 台あたり約 60 GB の空きディスク容量
* 所要時間は約 20 分

***

<div id="1-install-lume">
  ## 1) Lume をインストールする
</div>

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/trycua/cua/main/libs/lume/scripts/install.sh)"
```

もし `~/.local/bin` が PATH 環境変数に含まれていない場合:

```bash
echo 'export PATH="$PATH:$HOME/.local/bin"' >> ~/.zshrc && source ~/.zshrc
```

確認:

```bash
lume --version
```

ドキュメント: [Lume のインストール](https://cua.ai/docs/lume/guide/getting-started/installation)

***

<div id="2-create-the-macos-vm">
  ## 2) macOS 仮想マシンを作成する
</div>

```bash
lume create openclaw --os macos --ipsw latest
```

これにより macOS がダウンロードされ、VM が作成されます。VNC ウィンドウが自動的に開きます。

注意: ネットワーク環境によっては、ダウンロードに時間がかかる場合があります。

***

<div id="3-complete-setup-assistant">
  ## 3) セットアップアシスタントを完了する
</div>

VNC ウィンドウで次を実行します:

1. 言語と地域を選択する
2. Apple ID をスキップする（または、後で iMessage を使いたい場合はサインインする）
3. ユーザーアカウントを作成する（ユーザー名とパスワードを忘れないようにする）
4. すべてのオプション項目をスキップする

セットアップ完了後、SSH を有効にします:

1. 「システム設定」→「一般」→「共有」を開く
2. 「リモートログイン」を有効にする

***

<div id="4-get-the-vms-ip-address">
  ## 4) VM の IP アドレスを確認する
</div>

```bash
lume get openclaw
```

IP アドレス（通常は `192.168.64.x`）を確認します。

***

<div id="5-ssh-into-the-vm">
  ## 5) VM に SSH で接続する
</div>

```bash
ssh youruser@192.168.64.X
```

`youruser` を作成したアカウント名に、IP を VM の IP アドレスに置き換えてください。

***

<div id="6-install-openclaw">
  ## 6) OpenClaw をインストールする
</div>

VM 内で:

```bash
npm install -g openclaw@latest
openclaw onboard --install-daemon
```

オンボーディングプロンプトに従ってモデルプロバイダー（Anthropic、OpenAI など）を設定してください。

***

<div id="7-configure-channels">
  ## 7) チャンネルを設定する
</div>

設定ファイルを編集します：

```bash
nano ~/.openclaw/openclaw.json
```

チャネルを追加する：

```json
{
  "channels": {
    "whatsapp": {
      "dmPolicy": "allowlist",
      "allowFrom": ["+15551234567"]
    },
    "telegram": {
      "botToken": "YOUR_BOT_TOKEN"
    }
  }
}
```

次に、QRコードをスキャンして WhatsApp にログインします:

```bash
openclaw channels login
```

***

<div id="8-run-the-vm-headlessly">
  ## 8) VM をヘッドレスで実行する
</div>

VM を停止し、ディスプレイなしで起動し直します:

```bash
lume stop openclaw
lume run openclaw --no-display
```

VM はバックグラウンドで動作します。OpenClaw のデーモンが Gateway を稼働させ続けます。

ステータスを確認するには：

```bash
ssh youruser@192.168.64.X "openclaw status"
```

***

<div id="bonus-imessage-integration">
  ## おまけ: iMessage 連携
</div>

macOS 上で動かす最大の目玉機能がこれです。[BlueBubbles](https://bluebubbles.app) を使って iMessage を OpenClaw で使えるようにします。

VM 内で以下を実行します:

1. bluebubbles.app から BlueBubbles をダウンロードする
2. Apple ID でサインインする
3. Web API を有効化してパスワードを設定する
4. BlueBubbles の webhook の送信先を Gateway に設定する（例: `https://your-gateway-host:3000/bluebubbles-webhook?password=<password>`）

OpenClaw の設定に以下を追加します:

```json
{
  "channels": {
    "bluebubbles": {
      "serverUrl": "http://localhost:1234",
      "password": "your-api-password",
      "webhookPath": "/bluebubbles-webhook"
    }
  }
}
```

Gateway を再起動してください。これでエージェントは iMessage を送受信できるようになります。

セットアップの詳細: [BlueBubbles チャンネル](/ja/channels/bluebubbles)

***

<div id="save-a-golden-image">
  ## ゴールデンイメージを保存する
</div>

さらにカスタマイズを進める前に、クリーンな状態のスナップショットを作成しておいてください。

```bash
lume stop openclaw
lume clone openclaw openclaw-golden
```

いつでもリセット可能です：

```bash
lume stop openclaw && lume delete openclaw
lume clone openclaw-golden openclaw
lume run openclaw --no-display
```

***

<div id="running-247">
  ## 24時間365日の稼働
</div>

VM を常時稼働させるには、次の点を確認してください:

* Mac の電源アダプタを接続したままにする
* システム設定 → 省エネルギー でスリープを無効にする
* 必要に応じて `caffeinate` を使用する

本当に「常時オン」にしたい場合は、専用の Mac mini や小規模な VPS の利用を検討してください。[VPS ホスティング](/ja/vps) を参照してください。

***

<div id="troubleshooting">
  ## トラブルシューティング
</div>

| 問題 | 解決策 |
|---------|----------|
| VM に SSH 接続できない | VM のシステム設定で「リモートログイン」が有効になっているか確認する |
| VM の IP が表示されない | VM の起動が完了するまで待ってから、もう一度 `lume get openclaw` を実行する |
| Lume コマンドが見つからない | `~/.local/bin` を PATH 環境変数に追加する |
| WhatsApp の QR コードが読み取れない | `openclaw channels login` を実行する際に、ホストではなく VM にログインしていることを確認する |

***

<div id="related-docs">
  ## 関連ドキュメント
</div>

* [VPS ホスティング](/ja/vps)
* [ノード](/ja/nodes)
* [Gateway リモート](/ja/gateway/remote)
* [BlueBubbles チャンネル](/ja/channels/bluebubbles)
* [Lume クイックスタート](https://cua.ai/docs/lume/guide/getting-started/quickstart)
* [Lume CLI リファレンス](https://cua.ai/docs/lume/reference/cli-reference)
* [無人 VM セットアップ](https://cua.ai/docs/lume/guide/fundamentals/unattended-setup) (上級者向け)
* [Docker サンドボックス](/ja/install/docker) (別の分離アプローチ)