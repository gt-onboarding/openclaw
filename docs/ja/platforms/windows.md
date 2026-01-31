---
title: Windows
summary: "Windows（WSL2）対応状況とコンパニオンアプリの状況"
read_when:
  - Windows に OpenClaw をインストールするとき
  - Windows 向けコンパニオンアプリの状況を確認したいとき
---

<div id="windows-wsl2">
  # Windows (WSL2)
</div>

Windows で OpenClaw を使う場合は、**WSL2 経由**（Ubuntu 推奨）を推奨します。
CLI と Gateway は Linux 上で実行されるため、実行環境を統一でき、
ツールチェーン（Node/Bun/pnpm、Linux バイナリ、スキル）との互換性が大幅に向上します。
Windows へのネイティブインストールは未検証であり、問題が発生しやすい構成です。

Windows ネイティブのコンパニオンアプリは今後提供予定です。

<div id="install-wsl2">
  ## インストール (WSL2)
</div>

* [はじめに](/ja/start/getting-started)（WSL 内で実行）
* [インストールと更新](/ja/install/updating)
* 公式 WSL2 ガイド (Microsoft)：https://learn.microsoft.com/windows/wsl/install

<div id="gateway">
  ## Gateway
</div>

* [Gateway 運用ランブック](/ja/gateway)
* [設定](/ja/gateway/configuration)

<div id="gateway-service-install-cli">
  ## Gateway サービスのインストール (CLI)
</div>

WSL2 環境内で:

```
openclaw onboard --install-daemon
```

または:

```
openclaw gateway install
```

または：

```
openclaw configure
```

プロンプトが表示されたら、**Gateway service** を選択します。

修復/移行:

```
openclaw doctor
```

<div id="advanced-expose-wsl-services-over-lan-portproxy">
  ## 上級者向け: WSL のサービスを LAN に公開する (portproxy)
</div>

WSL には独自の仮想ネットワークがあります。別のマシンから
**WSL 内** で動作しているサービス (SSH、ローカルの TTS サーバー、Gateway など) にアクセスする必要がある場合は、
Windows 側のポートを現在の WSL の IP アドレスに転送する必要があります。WSL の IP は再起動のたびに変わるため、
転送ルールを更新しなければならない場合があります。

例 (PowerShell を **管理者として** 実行):

```powershell
$Distro = "Ubuntu-24.04"
$ListenPort = 2222
$TargetPort = 22

$WslIp = (wsl -d $Distro -- hostname -I).Trim().Split(" ")[0]
if (-not $WslIp) { throw "WSL IPが見つかりません。" }

netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=$ListenPort `
  connectaddress=$WslIp connectport=$TargetPort
```

Windows ファイアウォールでポートを許可する（初回のみ）:

```powershell
New-NetFirewallRule -DisplayName "WSL SSH $ListenPort" -Direction Inbound `
  -Protocol TCP -LocalPort $ListenPort -Action Allow
```

WSL を再起動した後に portproxy を再設定する:

```powershell
netsh interface portproxy delete v4tov4 listenport=$ListenPort listenaddress=0.0.0.0 | Out-Null
netsh interface portproxy add v4tov4 listenport=$ListenPort listenaddress=0.0.0.0 `
  connectaddress=$WslIp connectport=$TargetPort | Out-Null
```

注意:

* 別のマシンから SSH するときは、**Windows ホストの IP** を指定します（例: `ssh user@windows-host -p 2222`）。
* リモートノードは、**到達可能な** Gateway の URL（`127.0.0.1` 以外）を指定する必要があります。`openclaw status --all` で確認してください。
* LAN からアクセスしたい場合は `listenaddress=0.0.0.0` を使用し、`127.0.0.1` を指定するとローカルのみのアクセスになります。
* これを自動化したい場合は、ログイン時にリフレッシュ手順を実行するように、タスク スケジューラでタスクを登録してください。

<div id="step-by-step-wsl2-install">
  ## WSL2 インストール手順
</div>

<div id="1-install-wsl2-ubuntu">
  ### 1) WSL2 と Ubuntu をインストールする
</div>

PowerShell（管理者として）を開きます:

```powershell
wsl --install
# または明示的にディストリビューションを選択する場合:
wsl --list --online
wsl --install -d Ubuntu-24.04
```

Windows から再起動を求められたら、再起動してください。

<div id="2-enable-systemd-required-for-gateway-install">
  ### 2) systemd を有効化する（Gateway のインストールに必須）
</div>

WSL のターミナルで実行します:

```bash
sudo tee /etc/wsl.conf >/dev/null <<'EOF'
[boot]
systemd=true
EOF
```

次に、PowerShell で以下を実行します：

```powershell
wsl --shutdown
```

Ubuntu を再度開き、次を確認してください:

```bash
systemctl --user status
```

<div id="3-install-openclaw-inside-wsl">
  ### 3) OpenClaw をインストールする（WSL 内）
</div>

WSL 内で Linux 版 Getting Started の手順に従ってください。

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # 初回実行時にUI依存関係を自動インストールします
pnpm build
openclaw onboard
```

詳しいガイド: [はじめに](/ja/start/getting-started)

<div id="windows-companion-app">
  ## Windows コンパニオンアプリ
</div>

現在、Windows 用のコンパニオンアプリはまだありません。実現に向けて開発に参加してくださる方からのコントリビュートを歓迎します。