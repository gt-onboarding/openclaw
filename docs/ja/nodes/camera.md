---
title: カメラ
summary: "エージェントが利用するカメラキャプチャ（iOS ノード + macOS アプリ）：写真（jpg）および短い動画クリップ（mp4）"
read_when:
  - iOS ノードまたは macOS でカメラキャプチャを追加・変更するとき
  - エージェントがアクセス可能な MEDIA 一時ファイルワークフローを拡張するとき
---

<div id="camera-capture-agent">
  # Camera capture (agent)
</div>

OpenClaw は、エージェントのワークフローにおける**カメラキャプチャ**をサポートしています：

- **iOS ノード**（Gateway を介してペアリング）：`node.invoke` を通じて **写真**（`jpg`）または **短い動画クリップ**（オプションの音声付き `mp4`）を撮影できます。
- **Android ノード**（Gateway を介してペアリング）：`node.invoke` を通じて **写真**（`jpg`）または **短い動画クリップ**（オプションの音声付き `mp4`）を撮影できます。
- **macOS アプリ**（Gateway 経由のノード）：`node.invoke` を通じて **写真**（`jpg`）または **短い動画クリップ**（オプションの音声付き `mp4`）を撮影できます。

すべてのカメラアクセスは、**ユーザーが管理する設定**によって制御されます。

<div id="ios-node">
  ## iOS ノード
</div>

<div id="user-setting-default-on">
  ### ユーザー設定（デフォルトはオン）
</div>

- iOS の［設定］タブ → **Camera** → **Allow Camera**（`camera.enabled`）
  - デフォルト: **オン**（キーが存在しない場合は有効として扱われます）。
  - オフの場合: `camera.*` コマンドは `CAMERA_DISABLED` を返します。

<div id="commands-via-gateway-nodeinvoke">
  ### コマンド（Gateway の `node.invoke` 経由）
</div>

- `camera.list`
  - レスポンスペイロード:
    - `devices`: `{ id, name, position, deviceType }` の配列

- `camera.snap`
  - パラメータ:
    - `facing`: `front|back`（デフォルト: `front`）
    - `maxWidth`: 数値（オプション。iOS ノード上でのデフォルトは `1600`）
    - `quality`: `0..1`（オプション。デフォルトは `0.9`）
    - `format`: 現在は `jpg`
    - `delayMs`: 数値（オプション。デフォルトは `0`）
    - `deviceId`: 文字列（オプション。`camera.list` の結果で得られる値）
  - レスポンスペイロード:
    - `format: "jpg"`
    - `base64: "<...>"`
    - `width`, `height`
  - ペイロードガード: base64 ペイロードを 5 MB 未満に抑えるため、写真は再圧縮されます。

- `camera.clip`
  - パラメータ:
    - `facing`: `front|back`（デフォルト: `front`）
    - `durationMs`: 数値（デフォルト `3000`。最大値 `60000` までに制限）
    - `includeAudio`: 真偽値（デフォルト `true`）
    - `format`: 現在は `mp4`
    - `deviceId`: 文字列（オプション。`camera.list` の結果で得られる値）
  - レスポンスペイロード:
    - `format: "mp4"`
    - `base64: "<...>"`
    - `durationMs`
    - `hasAudio`

<div id="foreground-requirement">
  ### フォアグラウンド要件
</div>

`canvas.*` と同様に、iOS ノードは **フォアグラウンド** でのみ `camera.*` コマンドを許可します。バックグラウンドでの呼び出しは `NODE_BACKGROUND_UNAVAILABLE` を返します。

<div id="cli-helper-temp-files-media">
  ### CLI ヘルパー（一時ファイル + MEDIA）
</div>

添付ファイルを取得する最も簡単な方法は、CLI ヘルパーを使うことです。これはデコードしたメディアを一時ファイルに書き込み、`MEDIA:<path>` を出力します。

例：

```bash
openclaw nodes camera snap --node <id>               # デフォルト: 前面カメラと背面カメラの両方 (2つのMEDIA行)
openclaw nodes camera snap --node <id> --facing front
openclaw nodes camera clip --node <id> --duration 3000
openclaw nodes camera clip --node <id> --no-audio
```

注意:

* `nodes camera snap` は、エージェントに両方のビューを提供するため、デフォルトで前面・背面の**両方**のカメラを使用します。
* 独自のラッパーを用意しない限り、出力ファイルは一時ファイルとして扱われます（OS の一時ディレクトリ内に保存されます）。


<div id="android-node">
  ## Android ノード
</div>

### ユーザー設定（デフォルトで有効）

- Android 設定シート → **Camera** → **Allow Camera**（`camera.enabled`）
  - デフォルト: **オン**（キーが存在しない場合は有効として扱われます）。
  - オフの場合: `camera.*` コマンドは `CAMERA_DISABLED` を返します。

<div id="permissions">
  ### 権限
</div>

- Android では実行時権限が必要です:
  - `camera.snap` と `camera.clip` の両方に `CAMERA`。
  - `includeAudio=true` のときの `camera.clip` には `RECORD_AUDIO`。

権限が不足している場合は、可能であればアプリがダイアログで許可を求めます。拒否された場合、`camera.*` リクエストは
`*_PERMISSION_REQUIRED` エラーで失敗します。

<div id="foreground-requirement">
  ### フォアグラウンド要件
</div>

`canvas.*` と同様に、Android ノードではアプリが **フォアグラウンド** にある場合にのみ `camera.*` コマンドが許可されます。バックグラウンドから呼び出した場合は `NODE_BACKGROUND_UNAVAILABLE` が返されます。

<div id="payload-guard">
  ### ペイロードガード
</div>

base64 ペイロードを 5 MB 未満に抑えるため、写真データは再圧縮されます。

<div id="macos-app">
  ## macOS アプリ
</div>

<div id="user-setting-default-off">
  ### ユーザー設定（デフォルトはオフ）
</div>

macOS コンパニオンアプリには次のチェックボックスがあります：

- **Settings → General → Allow Camera** (`openclaw.cameraEnabled`)
  - デフォルト: **off**
  - off の場合: カメラ要求は「Camera disabled by user」（ユーザーによってカメラが無効化されています）というメッセージを返します。

<div id="cli-helper-node-invoke">
  ### CLI ヘルパー（ノード呼び出し）
</div>

メインの `openclaw` CLI を使用して、macOS ノード上でカメラコマンドを実行します。

例:

```bash
openclaw nodes camera list --node <id>            # list camera ids
openclaw nodes camera snap --node <id>            # prints MEDIA:<path>
openclaw nodes camera snap --node <id> --max-width 1280
openclaw nodes camera snap --node <id> --delay-ms 2000
openclaw nodes camera snap --node <id> --device-id <id>
openclaw nodes camera clip --node <id> --duration 10s          # prints MEDIA:<path>
openclaw nodes camera clip --node <id> --duration-ms 3000      # MEDIA:<path>を出力 (レガシーフラグ)
openclaw nodes camera clip --node <id> --device-id <id>
openclaw nodes camera clip --node <id> --no-audio
```

Notes:

* `openclaw nodes camera snap` は、明示的に変更しない限り `maxWidth=1600` をデフォルト値として使用します。
* macOS では、`camera.snap` はウォームアップと露出が安定してからキャプチャする前に、`delayMs`（デフォルト 2000ms）だけ待機します。
* 写真のペイロードは、Base64 が 5 MB 未満になるように再圧縮されます。


<div id="safety-practical-limits">
  ## 安全性と実務上の制約
</div>

- カメラとマイクへのアクセス時には、通常どおり OS の権限確認プロンプトが表示されます（また、Info.plist 内に使用目的文字列が必要です）。
- 動画クリップは、過大なノードペイロード（base64 のオーバーヘッド＋メッセージ上限）を避けるため、長さを制限しています（現在は `<= 60s`）。

<div id="macos-screen-video-os-level">
  ## macOS 画面録画（OS レベル）
</div>

カメラではなく *画面* を録画する場合は、macOS コンパニオンアプリを使用してください。

```bash
openclaw nodes screen record --node <id> --duration 10s --fps 15   # MEDIA:<path> を出力します
```

注意:

* macOS の **画面収録（Screen Recording）** の権限（TCC）が必要です。
