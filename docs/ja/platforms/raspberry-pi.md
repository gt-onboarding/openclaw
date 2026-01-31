---
title: Raspberry Pi
summary: "Raspberry Pi での OpenClaw（低予算セルフホスト環境）"
read_when:
  - Raspberry Pi に OpenClaw をセットアップするとき
  - ARM デバイス上で OpenClaw を実行するとき
  - 低コストで常時稼働するパーソナル AI を構築するとき
---

<div id="openclaw-on-raspberry-pi">
  # Raspberry Pi での OpenClaw
</div>

<div id="goal">
  ## 目的
</div>

一度きりの初期費用 **約 35〜80 ドル** で、Raspberry Pi 上に常時稼働の OpenClaw Gateway を構築して運用する（追加の月額料金は不要）。

次のような用途に最適です:

* 24 時間 365 日稼働のパーソナル AI アシスタント
* ホームオートメーション用ハブ
* 低消費電力で常時利用可能な Telegram / WhatsApp ボット

<div id="hardware-requirements">
  ## ハードウェア要件
</div>

| Pi モデル | RAM | 動作可否 | 備考 |
|----------|-----|--------|-------|
| **Pi 5** | 4GB/8GB | ✅ 最適 | 最速、推奨 |
| **Pi 4** | 4GB | ✅ 良好 | 多くのユーザーにとってのスイートスポット |
| **Pi 4** | 2GB | ✅ 可 | 動作するが、スワップ領域を追加 |
| **Pi 4** | 1GB | ⚠️ ぎりぎり | スワップありで動作可能、最小限の構成 |
| **Pi 3B+** | 1GB | ⚠️ 遅い | 動作するがかなり重い |
| **Pi Zero 2 W** | 512MB | ❌ | 非推奨 |

**最小スペック:** RAM 1GB、1コア、ディスク容量 500MB\
**推奨:** RAM 2GB以上、64-bit OS、16GB以上の SD カード（または USB SSD）

<div id="what-youll-need">
  ## 必要なもの
</div>

* Raspberry Pi 4 または 5（2GB 以上を推奨）
* microSD カード（16GB 以上）または USB SSD（より高いパフォーマンス）
* 電源アダプター（公式の Pi 電源アダプター推奨）
* ネットワーク接続（有線 LAN または Wi‑Fi）
* 作業時間の目安: 約 30 分

<div id="1-flash-the-os">
  ## 1) OS を書き込む
</div>

**Raspberry Pi OS Lite (64-bit)** を使用します。ヘッドレスサーバーとして利用するため、デスクトップ環境は不要です。

1. [Raspberry Pi Imager](https://www.raspberrypi.com/software/) をダウンロードする
2. OS を選択：**Raspberry Pi OS Lite (64-bit)**
3. 歯車アイコン (⚙️) をクリックして事前設定を行う:
   * ホスト名を設定: `gateway-host`
   * SSH を有効化
   * ユーザー名 / パスワードを設定
   * WiFi を設定 (Ethernet を使用しない場合)
4. SD カード / USB ドライブに書き込む
5. SD カード / USB ドライブを挿入して Pi を起動する

<div id="2-connect-via-ssh">
  ## 2) SSH 経由で接続する
</div>

```bash
ssh user@gateway-host
# または IP アドレスを使用する
ssh user@192.168.x.x
```

<div id="3-system-setup">
  ## 3) システムのセットアップ
</div>

```bash
# システムを更新
sudo apt update && sudo apt upgrade -y

# 必須パッケージをインストール
sudo apt install -y git curl build-essential

# タイムゾーンを設定（cronやリマインダーに重要）
sudo timedatectl set-timezone America/Chicago  # 自分のタイムゾーンに変更してください
```

<div id="4-install-nodejs-22-arm64">
  ## 4) Node.js 22 (ARM64) をインストール
</div>

```bash
# Install Node.js via NodeSource
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs

# Verify
node --version  # v22.x.xが表示されるはずです
npm --version
```

<div id="5-add-swap-important-for-2gb-or-less">
  ## 5) スワップ領域を追加する（2GB 以下では重要）
</div>

スワップ領域はメモリ不足によるクラッシュを防ぐためのものです。

```bash
# Create 2GB swap file
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make permanent
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# 低RAM環境向けに最適化（swappinessを減らす）
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

<div id="6-install-openclaw">
  ## 6) OpenClaw のインストール
</div>

<div id="option-a-standard-install-recommended">
  ### オプションA：標準インストール（推奨）
</div>

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

<div id="option-b-hackable-install-for-tinkering">
  ### オプションB: ハックしやすいインストール（試行・実験向け）
</div>

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
npm install
npm run build
npm link
```

このハックしやすいインストールでは、ログやコードに直接アクセスできるため、ARM 固有の問題をデバッグする際に役立ちます。

<div id="7-run-onboarding">
  ## 7) オンボーディングを行う
</div>

```bash
openclaw onboard --install-daemon
```

ウィザードに従って進めてください:

1. **Gateway モード:** Local を選択
2. **Auth:** API keys の利用を推奨（ヘッドレスな Pi では OAuth は挙動が不安定になりがち）
3. **Channels:** まずは Telegram がもっとも簡単です
4. **Daemon:** Yes を選択（systemd）

<div id="8-verify-installation">
  ## 8) インストールを確認する
</div>

```bash
# ステータスを確認
openclaw status

# サービスを確認
sudo systemctl status openclaw

# ログを表示
journalctl -u openclaw -f
```

<div id="9-access-the-dashboard">
  ## 9) ダッシュボードにアクセスする
</div>

Pi はヘッドレス構成のため、SSH トンネルを使ってアクセスします：

```bash
# ラップトップ/デスクトップから
ssh -L 18789:localhost:18789 user@gateway-host

# ブラウザで開く
open http://localhost:18789
```

または、常時アクセスを可能にするために Tailscale を利用します：

```bash
# On the Pi
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up

# 設定を更新
openclaw config set gateway.bind tailnet
sudo systemctl restart openclaw
```

***

<div id="performance-optimizations">
  ## パフォーマンスの最適化
</div>

<div id="use-a-usb-ssd-huge-improvement">
  ### USB SSD を使用する（大幅な改善）
</div>

SD カードは遅く、書き換えで早く劣化します。USB SSD を使うとパフォーマンスが大幅に向上します。

```bash
# USB起動しているか確認
lsblk
```

セットアップ方法については、[Pi USB ブートガイド](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#usb-mass-storage-boot)を参照してください。

<div id="reduce-memory-usage">
  ### メモリ使用量を抑える
</div>

```bash
# GPUメモリ割り当てを無効化（ヘッドレス環境）
echo 'gpu_mem=16' | sudo tee -a /boot/config.txt

# Disable Bluetooth if not needed
sudo systemctl disable bluetooth
```

<div id="monitor-resources">
  ### リソース監視
</div>

```bash
# メモリを確認
free -h

# CPU 温度を確認
vcgencmd measure_temp

# ライブモニタリング
htop
```

***

<div id="arm-specific-notes">
  ## ARM 向けの注意事項
</div>

<div id="binary-compatibility">
  ### バイナリ互換性
</div>

OpenClaw の機能のほとんどは ARM64 環境でも動作しますが、一部の外部バイナリについては ARM 向けビルドが必要になる場合があります。

| Tool | ARM64 対応状況 | Notes |
|------|--------------|-------|
| Node.js | ✅ | 問題なく動作 |
| WhatsApp (Baileys) | ✅ | Pure JS 実装のため問題なし |
| Telegram | ✅ | Pure JS 実装のため問題なし |
| gog (Gmail CLI) | ⚠️ | ARM 版があるか確認 |
| Chromium (browser) | ✅ | `sudo apt install chromium-browser` |

スキルが動作しない場合は、そのバイナリに ARM 向けビルドが提供されているか確認してください。多くの Go/Rust 製ツールには用意されていますが、そうでないものもあります。

<div id="32-bit-vs-64-bit">
  ### 32ビット vs 64ビット
</div>

**必ず64ビットOSを使用してください。** Node.jsや多くの最新ツールで必須です。次のコマンドで確認できます:

```bash
uname -m
# 表示されるべき値: aarch64 (64ビット)、armv7l (32ビット) ではないこと
```

***

<div id="recommended-model-setup">
  ## 推奨モデル構成
</div>

Pi は Gateway としてのみ動作し、モデルはクラウド上で実行されるため、API ベースのモデルを使用してください。

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-sonnet-4-20250514",
        "fallbacks": ["openai/gpt-4o-mini"]
      }
    }
  }
}
```

**Raspberry Pi でローカル LLM を動かそうとしないでください** — 小規模なモデルでも遅すぎます。重い処理は Claude/GPT に任せましょう。

***

<div id="auto-start-on-boot">
  ## ブート時の自動起動
</div>

オンボーディングウィザードで自動設定されますが、念のため以下を確認してください:

```bash
# サービスが有効になっているか確認
sudo systemctl is-enabled openclaw

# 有効になっていない場合は有効化
sudo systemctl enable openclaw

# 起動時に開始
sudo systemctl start openclaw
```

***

<div id="troubleshooting">
  ## トラブルシューティング
</div>

<div id="out-of-memory-oom">
  ### メモリ不足（OOM）
</div>

```bash
# メモリを確認
free -h

# スワップを追加する（ステップ5を参照）
# またはPi上で実行中のサービスを減らす
```

<div id="slow-performance">
  ### 動作が遅い場合
</div>

* SDカードの代わりに USB SSD を使用する
* 使っていないサービスを無効化する: `sudo systemctl disable cups bluetooth avahi-daemon`
* CPU のサーマルスロットリング状態を確認する: `vcgencmd get_throttled`（`0x0` が返ることを確認する）

<div id="service-wont-start">
  ### サービスが起動しない場合
</div>

```bash
# ログを確認
journalctl -u openclaw --no-pager -n 100

# 一般的な修正: リビルド
cd ~/openclaw  # hackable install を使用している場合
npm run build
sudo systemctl restart openclaw
```

<div id="arm-binary-issues">
  ### ARM バイナリの問題
</div>

スキルが「exec format error」で失敗する場合:

1. そのバイナリに ARM64 向けビルドが存在するか確認する
2. ソースからビルドしてみる
3. または ARM 対応の Docker コンテナを利用する

<div id="wifi-drops">
  ### WiFi の切断
</div>

WiFi 接続で動作しているヘッドレス Raspberry Pi の場合:

```bash
# WiFi電源管理を無効化
sudo iwconfig wlan0 power off

# Make permanent
echo 'wireless-power off' | sudo tee -a /etc/network/interfaces
```

***

<div id="cost-comparison">
  ## コスト比較
</div>

| セットアップ | 初期費用 | 月額費用 | 備考 |
|-------|---------------|--------------|-------|
| **Pi 4 (2GB)** | 約 $45 | $0 | ＋電力代（年間約 $5） |
| **Pi 4 (4GB)** | 約 $55 | $0 | 推奨構成 |
| **Pi 5 (4GB)** | 約 $60 | $0 | 最高のパフォーマンス |
| **Pi 5 (8GB)** | 約 $80 | $0 | オーバースペックだが将来性あり |
| DigitalOcean | $0 | $6/月 | 年間 $72 |
| Hetzner | $0 | €3.79/月 | 年間約 $50 |

**損益分岐点:** クラウド VPS と比較すると、Raspberry Pi は約 6〜12 か月で元が取れます。

***

<div id="see-also">
  ## 関連情報
</div>

* [Linux ガイド](/ja/platforms/linux) — 一般的な Linux セットアップ
* [DigitalOcean ガイド](/ja/platforms/digitalocean) — クラウドでの代替手段
* [Hetzner ガイド](/ja/platforms/hetzner) — Docker セットアップ
* [Tailscale](/ja/gateway/tailscale) — リモートアクセス
* [ノード](/ja/nodes) — ノートPCやスマートフォンを Raspberry Pi 上の Gateway とペアリングする