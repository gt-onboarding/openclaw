---
title: リリース
summary: "OpenClaw macOS リリースチェックリスト（Sparkle フィード、パッケージング、署名）"
read_when:
  - OpenClaw macOS リリースを作成または検証するとき
  - Sparkle appcast またはフィード用アセットを更新するとき
---

<div id="openclaw-macos-release-sparkle">
  # OpenClaw macOS リリース（Sparkle）
</div>

このアプリは現在、Sparkle による自動アップデートに対応しています。リリースビルドは Developer ID 署名を行い、zip アーカイブにし、署名済みの appcast エントリとともに公開する必要があります。

<div id="prereqs">
  ## 前提条件
</div>

- Developer ID Application 証明書がインストールされていること（例: `Developer ID Application: <Developer Name> (<TEAMID>)`）。
- Sparkle の秘密鍵パスが環境変数 `SPARKLE_PRIVATE_KEY_FILE` に設定されていること（Sparkle ed25519 秘密鍵へのパス。公開鍵は Info.plist に埋め込まれる）。設定されていない場合は `~/.profile` を確認すること。
- Gatekeeper で安全な DMG/zip 配布を行う場合は、`xcrun notarytool` 用の公証用認証情報（Keychain プロファイルまたは API キー）が必要。
  - シェルプロファイル内の App Store Connect API キー用環境変数から作成した、`openclaw-notary` という名前の Keychain プロファイルを使用する:
    - `APP_STORE_CONNECT_API_KEY_P8`, `APP_STORE_CONNECT_KEY_ID`, `APP_STORE_CONNECT_ISSUER_ID`
    - `echo "$APP_STORE_CONNECT_API_KEY_P8" | sed 's/\\n/\n/g' > /tmp/openclaw-notary.p8`
    - `xcrun notarytool store-credentials "openclaw-notary" --key /tmp/openclaw-notary.p8 --key-id "$APP_STORE_CONNECT_KEY_ID" --issuer "$APP_STORE_CONNECT_ISSUER_ID"`
- `pnpm` の依存関係がインストールされていること（`pnpm install --config.node-linker=hoisted`）。
- Sparkle ツールは SwiftPM 経由で `apps/macos/.build/artifacts/sparkle/Sparkle/bin/` に自動取得される（`sign_update`、`generate_appcast` など）。

<div id="build-package">
  ## ビルドとパッケージング
</div>

注意:

* `APP_BUILD` は `CFBundleVersion`/`sparkle:version` に対応します。数値かつ単調増加（`-beta` などは付けない）にしてください。そうしないと Sparkle に同一バージョンとして扱われてしまいます。
* デフォルトでは現在のアーキテクチャ（`$(uname -m)`）になります。リリース／ユニバーサルビルドでは `BUILD_ARCHS="arm64 x86_64"`（または `BUILD_ARCHS=all`）を設定してください。
* リリース用アーティファクト（zip + DMG + notarization）には `scripts/package-mac-dist.sh` を使用します。ローカル／開発用パッケージングには `scripts/package-mac-app.sh` を使用します。

```bash
# リポジトリルートから実行; Sparkle フィードを有効にするためリリース ID を設定。
# APP_BUILD は Sparkle の比較のため数値かつ単調増加である必要がある。
BUNDLE_ID=bot.molt.mac \
APP_VERSION=2026.1.27-beta.1 \
APP_BUILD="$(git rev-list --count HEAD)" \
BUILD_CONFIG=release \
SIGN_IDENTITY="Developer ID Application: <Developer Name> (<TEAMID>)" \
scripts/package-mac-app.sh

# 配布用 Zip (Sparkle デルタサポート用のリソースフォークを含む)
ditto -c -k --sequesterRsrc --keepParent dist/OpenClaw.app dist/OpenClaw-2026.1.27-beta.1.zip

# オプション: ユーザー向けにスタイル付き DMG もビルド (/Applications へドラッグ)
scripts/create-dmg.sh dist/OpenClaw.app dist/OpenClaw-2026.1.27-beta.1.dmg

# 推奨: zip + DMG のビルド + 公証/ステープル
# 最初に、キーチェーンプロファイルを一度作成:
#   xcrun notarytool store-credentials "openclaw-notary" \
#     --apple-id "<apple-id>" --team-id "<team-id>" --password "<app-specific-password>"
NOTARIZE=1 NOTARYTOOL_PROFILE=openclaw-notary \
BUNDLE_ID=bot.molt.mac \
APP_VERSION=2026.1.27-beta.1 \
APP_BUILD="$(git rev-list --count HEAD)" \
BUILD_CONFIG=release \
SIGN_IDENTITY="Developer ID Application: <Developer Name> (<TEAMID>)" \
scripts/package-mac-dist.sh

# オプション: リリースと共に dSYM を配布
ditto -c -k --keepParent apps/macos/.build/release/OpenClaw.app.dSYM dist/OpenClaw-2026.1.27-beta.1.dSYM.zip
```


<div id="appcast-entry">
  ## Appcast エントリ
</div>

Sparkle に書式付き HTML のリリースノートをレンダリングさせるため、リリースノートジェネレーターを使用してください。

```bash
SPARKLE_PRIVATE_KEY_FILE=/path/to/ed25519-private-key scripts/make_appcast.sh dist/OpenClaw-2026.1.27-beta.1.zip https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml
```

`CHANGELOG.md` から HTML のリリースノートを生成し（[`scripts/changelog-to-html.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/changelog-to-html.sh) 経由）、それらを appcast エントリに埋め込みます。
公開時には、更新した `appcast.xml` をリリースアセット（zip + dSYM）とあわせてコミットします。


<div id="publish-verify">
  ## 公開と検証
</div>

- `OpenClaw-2026.1.27-beta.1.zip`（および `OpenClaw-2026.1.27-beta.1.dSYM.zip`）を、タグ `v2026.1.27-beta.1` 用の GitHub リリースにアップロードする。
- 生の appcast URL（raw URL）が組み込み済みフィードと一致していることを確認する: `https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml`。
- サニティチェック:
  - `curl -I https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml` が 200 を返すこと。
  - アセットのアップロード後に `curl -I <enclosure url>` が 200 を返すこと。
  - 以前の公開ビルドで、About タブから “Check for Updates…” を実行し、Sparkle が新しいビルドを問題なくインストールできることを検証する。

完了の定義: 署名済みアプリと appcast が公開されており、古いインストール済みバージョンからのアップデートフローが正常に動作し、リリースアセットが GitHub リリースに添付されていること。