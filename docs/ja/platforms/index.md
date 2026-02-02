---
title: プラットフォーム
summary: "プラットフォーム対応状況の概要（Gateway + コンパニオンアプリ）"
read_when:
  - 対応OSやインストール先を確認したいとき
  - Gateway をどこで動かすか検討しているとき
---

<div id="platforms">
  # プラットフォーム
</div>

OpenClaw のコアは TypeScript で書かれています。**Node.js が推奨ランタイムです**。
Bun は Gateway では推奨されません（WhatsApp/Telegram 関連の不具合があります）。

macOS（メニューバーアプリ）およびモバイルノード（iOS/Android）向けのコンパニオンアプリが用意されています。Windows と
Linux 向けコンパニオンアプリも計画中ですが、Gateway 自体は現時点ですでに完全にサポートされています。
Windows 向けのネイティブコンパニオンアプリも計画中ですが、Gateway の利用は WSL2 経由を推奨します。

<div id="choose-your-os">
  ## お使いの OS を選択
</div>

* macOS: [macOS](/ja/platforms/macos)
* iOS: [iOS](/ja/platforms/ios)
* Android: [Android](/ja/platforms/android)
* Windows: [Windows](/ja/platforms/windows)
* Linux: [Linux](/ja/platforms/linux)

<div id="vps-hosting">
  ## VPS とホスティング
</div>

* VPS ハブ: [VPS ホスティング](/ja/vps)
* Fly.io: [Fly.io](/ja/platforms/fly)
* Hetzner（Docker）: [Hetzner](/ja/platforms/hetzner)
* GCP（Compute Engine）: [GCP](/ja/platforms/gcp)
* exe.dev（VM + HTTPS プロキシ）: [exe.dev](/ja/platforms/exe-dev)

<div id="common-links">
  ## 共通リンク
</div>

* インストールガイド: [はじめに](/ja/start/getting-started)
* Gateway ランブック: [Gateway](/ja/gateway)
* Gateway 設定ガイド: [Configuration](/ja/gateway/configuration)
* サービスの状態: `openclaw gateway status`

<div id="gateway-service-install-cli">
  ## Gateway サービスのインストール (CLI)
</div>

次のいずれかの方法を使用します（いずれもサポートされています）:

* ウィザード（推奨）: `openclaw onboard --install-daemon`
* 直接実行: `openclaw gateway install`
* 設定フロー: `openclaw configure` → **Gateway service** を選択
* 修復／移行: `openclaw doctor`（サービスのインストールまたは修復を行うオプションを提示）

サービスのインストール先は OS によって異なります:

* macOS: LaunchAgent（`bot.molt.gateway` または `bot.molt.<profile>`、レガシーは `com.openclaw.*`）
* Linux/WSL2: systemd ユーザーサービス（`openclaw-gateway[-<profile>].service`）