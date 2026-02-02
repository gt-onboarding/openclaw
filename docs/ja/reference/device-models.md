---
title: デバイスモデル
summary: "OpenClaw が macOS アプリ内で Apple デバイスのモデル識別子をユーザーフレンドリーな名称に対応付ける仕組み。"
read_when:
  - デバイスモデル識別子のマッピングや NOTICE／ライセンスファイルを更新する場合
  - Instances UI におけるデバイス名の表示方法を変更する場合
---

<div id="device-model-database-friendly-names">
  # デバイスモデルデータベース（わかりやすい名称）
</div>

macOS コンパニオンアプリは、Apple のモデル識別子（例: `iPad16,6`、`Mac16,6`）をわかりやすい名称に対応付けることで、**Instances** UI にユーザー向けの Apple デバイスモデル名を表示します。

このマッピングは、次のパス配下の JSON としてリポジトリに含まれています:

- `apps/macos/Sources/OpenClaw/Resources/DeviceModels/`

<div id="data-source">
  ## データソース
</div>

現在、以下の MIT ライセンスのリポジトリからマッピングを取り込んでいます:

- `kyle-seongwoo-jun/apple-device-identifiers`

ビルドの再現性を保つため、JSON ファイルは上流リポジトリの特定のコミットに固定しています（`apps/macos/Sources/OpenClaw/Resources/DeviceModels/NOTICE.md` に記録されています）。

<div id="updating-the-database">
  ## データベースの更新
</div>

1. ピン留めしたい upstream のコミットを選択します（iOS 用に 1 つ、macOS 用に 1 つ）。
2. `apps/macos/Sources/OpenClaw/Resources/DeviceModels/NOTICE.md` 内のコミットハッシュを更新します。
3. それらのコミットにピン留めした状態で JSON ファイルを再ダウンロードします。

```bash
IOS_COMMIT="<ios-device-identifiers.json のコミット SHA>"
MAC_COMMIT="<mac-device-identifiers.json のコミット SHA>"

curl -fsSL "https://raw.githubusercontent.com/kyle-seongwoo-jun/apple-device-identifiers/${IOS_COMMIT}/ios-device-identifiers.json" \
  -o apps/macos/Sources/OpenClaw/Resources/DeviceModels/ios-device-identifiers.json

curl -fsSL "https://raw.githubusercontent.com/kyle-seongwoo-jun/apple-device-identifiers/${MAC_COMMIT}/mac-device-identifiers.json" \
  -o apps/macos/Sources/OpenClaw/Resources/DeviceModels/mac-device-identifiers.json
```

4. `apps/macos/Sources/OpenClaw/Resources/DeviceModels/LICENSE.apple-device-identifiers.txt` が引き続き upstream と一致していることを確認します（upstream のライセンスが変更された場合は、このファイルを置き換えます）。
5. macOS アプリがクリーンにビルドできること（警告が一切ないこと）を確認します。

```bash
swift build --package-path apps/macos
```
