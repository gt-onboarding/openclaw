---
title: 署名
summary: "パッケージングスクリプトで生成された macOS デバッグビルドの署名手順"
read_when:
  - macOS デバッグビルドのビルドまたは署名を行うとき
---

<div id="mac-signing-debug-builds">
  # mac 署名（デバッグビルド）
</div>

このアプリは通常 [`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) からビルドされます。このスクリプトは現在、次のことを行います:

* 安定したデバッグ用バンドル識別子 `ai.openclaw.mac.debug` を設定します
* Info.plist にそのバンドル ID を書き込みます（`BUNDLE_ID=...` で上書き可能）
* [`scripts/codesign-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/codesign-mac-app.sh) を呼び出してメインバイナリとアプリバンドルに署名し、macOS が各リビルドを同じ署名付きバンドルとして扱い、TCC 権限（通知、アクセシビリティ、画面収録、マイク、音声認識）を保持できるようにします。権限を安定させるには実際の署名 ID を使用してください。アドホック署名は任意選択であり不安定です（[macOS permissions](/ja/platforms/mac/permissions) を参照）。
* デフォルトで `CODESIGN_TIMESTAMP=auto` を使用します。Developer ID 署名に対して信頼されたタイムスタンプを有効にします。タイムスタンプ付与をスキップするには（オフラインのデバッグビルド用に）`CODESIGN_TIMESTAMP=off` を設定します。
* ビルドメタデータを Info.plist に注入します: `OpenClawBuildTimestamp`（UTC）および `OpenClawGitCommit`（短いハッシュ）。これにより About ペインでビルド情報、git、デバッグ/リリースチャネルを表示できます。
* **パッケージングには Node 22 以上が必要です**。このスクリプトは TS ビルドと Control UI のビルドを実行します。
* 環境変数から `SIGN_IDENTITY` を読み取ります。常に自分の証明書で署名するには、シェルの rc に `export SIGN_IDENTITY="Apple Development: Your Name (TEAMID)"`（または Developer ID Application 証明書）を追加してください。アドホック署名は `ALLOW_ADHOC_SIGNING=1` または `SIGN_IDENTITY="-"` を明示的に設定した場合のみ有効になります（権限テスト用途には非推奨）。
* 署名後に Team ID の監査を実行し、アプリバンドル内の任意の Mach-O が異なる Team ID で署名されている場合は失敗させます。これを回避するには `SKIP_TEAM_ID_CHECK=1` を設定します。

<div id="usage">
  ## 使用方法
</div>

```bash
# from repo root
scripts/package-mac-app.sh               # auto-selects identity; errors if none found
SIGN_IDENTITY="Developer ID Application: Your Name" scripts/package-mac-app.sh   # real cert
ALLOW_ADHOC_SIGNING=1 scripts/package-mac-app.sh    # ad-hoc (permissions will not stick)
SIGN_IDENTITY="-" scripts/package-mac-app.sh        # explicit ad-hoc (same caveat)
DISABLE_LIBRARY_VALIDATION=1 scripts/package-mac-app.sh   # 開発専用 Sparkle Team ID 不一致の回避策
```

<div id="ad-hoc-signing-note">
  ### アドホック署名に関する注意
</div>

`SIGN_IDENTITY="-"`（アドホック）で署名する場合、スクリプトは自動的に **Hardened Runtime**（`--options runtime`）を無効化します。これは、同じ Team ID を共有しない埋め込みフレームワーク（Sparkle など）をアプリが読み込もうとした際に発生するクラッシュを防ぐために必要です。アドホック署名では TCC 権限の永続化も機能しなくなるため、復旧手順については [macOS permissions](/ja/platforms/mac/permissions) を参照してください。

<div id="build-metadata-for-about">
  ## About 用のビルドメタデータ
</div>

`package-mac-app.sh` はバンドルに次の情報を書き込みます:

* `OpenClawBuildTimestamp`: パッケージ作成時点の ISO8601 形式の UTC
* `OpenClawGitCommit`: 短い git ハッシュ（取得できない場合は `unknown`）

About タブはこれらのキーを読み取り、バージョン、ビルド日時、git コミット、およびデバッグビルドかどうか（`#if DEBUG` 経由）を表示します。コードを変更したあとは、パッケージャを実行してこれらの値を更新してください。

<div id="why">
  ## 理由
</div>

TCC の権限は、バンドル識別子 *および* コード署名に紐づきます。UUID が毎回変わる署名なしのデバッグビルドでは、リビルドのたびに macOS が付与済みの権限を忘れてしまっていました。バイナリに署名（デフォルトではアドホック署名）し、固定のバンドル ID／パス（`dist/OpenClaw.app`）を維持することで、ビルド間で権限を保持でき、VibeTunnel と同じアプローチになります。