---
title: ブラウザ
summary: "統合ブラウザ制御サービス＋アクションコマンド"
read_when:
  - エージェント制御のブラウザ自動化を追加するとき
  - openclaw が自分の Chrome の動作に干渉している原因をデバッグするとき
  - macOS アプリでブラウザ設定とライフサイクルを実装するとき
---

<div id="browser-openclaw-managed">
  # ブラウザ（openclaw 管理）
</div>

OpenClaw は、エージェントが制御する **専用の Chrome/Brave/Edge/Chromium プロファイル** を起動できます。
これはあなたの個人用ブラウザから分離されており、Gateway 内部の小さなローカル
制御サービス（ループバック専用）を通じて管理されます。

初心者向けのイメージ：

* **別枠の、エージェント専用ブラウザ** と考えてください。
* `openclaw` プロファイルは、あなたの個人用ブラウザプロファイルには **一切触れません**。
* エージェントは、安全なレーン（隔離環境）の中で **タブを開き、ページを閲覧し、クリックし、文字を入力** できます。
* デフォルトの `chrome` プロファイルは、拡張機能リレー経由で **システム標準の Chromium ブラウザ** を使用します。分離された管理ブラウザを使うには `openclaw` に切り替えてください。

<div id="what-you-get">
  ## 得られるもの
</div>

* **openclaw** という名前の別ブラウザプロファイル（デフォルトではオレンジのアクセント）。
* 予測可能で一貫したタブ制御（list/open/focus/close）。
* エージェントによる操作（click/type/drag/select）、スナップショット、スクリーンショット、PDF。
* オプションのマルチプロファイル対応（`openclaw`、`work`、`remote` など）。

このブラウザは日常利用向けでは**ありません**。エージェントの自動化と検証のための、安全に分離された実行環境です。

<div id="quick-start">
  ## クイックスタート
</div>

```bash
openclaw browser --browser-profile openclaw status
openclaw browser --browser-profile openclaw start
openclaw browser --browser-profile openclaw open https://example.com
openclaw browser --browser-profile openclaw snapshot
```

「Browser disabled」と表示された場合は、下記の設定で有効化してから Gateway を再起動してください。

<div id="profiles-openclaw-vs-chrome">
  ## プロファイル: `openclaw` と `chrome`
</div>

* `openclaw`: 管理された分離ブラウザ環境（拡張機能は不要）。
* `chrome`: **システムブラウザ** への拡張機能によるリレー（OpenClaw
  拡張機能をタブに紐付ける必要があります）。

デフォルトで管理モードにしたい場合は、`browser.defaultProfile: "openclaw"` を設定します。

<div id="configuration">
  ## 設定
</div>

ブラウザ関連の設定は `~/.openclaw/openclaw.json` に格納されています。

```json5
{
  browser: {
    enabled: true,                    // デフォルト: true
    // cdpUrl: "http://127.0.0.1:18792", // レガシー単一プロファイル上書き
    remoteCdpTimeoutMs: 1500,         // リモートCDP HTTPタイムアウト (ミリ秒)
    remoteCdpHandshakeTimeoutMs: 3000, // リモートCDP WebSocketハンドシェイクタイムアウト (ミリ秒)
    defaultProfile: "chrome",
    color: "#FF4500",
    headless: false,
    noSandbox: false,
    attachOnly: false,
    executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    profiles: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work: { cdpPort: 18801, color: "#0066CC" },
      remote: { cdpUrl: "http://10.0.0.42:9222", color: "#00AA00" }
    }
  }
}
```

Notes:

* ブラウザー制御サービスは、`gateway.port` から導出されるポートのループバックインターフェースにバインドされます
  （デフォルト: `18791`（gateway + 2））。リレーはその次のポート（`18792`）を使用します。
* Gateway ポート（`gateway.port` または `OPENCLAW_GATEWAY_PORT`）を上書きすると、
  導出されるブラウザー用ポートも同じ系列になるようにシフトします。
* `cdpUrl` は未設定の場合、リレー用ポートがデフォルトになります。
* `remoteCdpTimeoutMs` は、リモート（非 loopback）CDP の到達性チェックに適用されます。
* `remoteCdpHandshakeTimeoutMs` は、リモート CDP WebSocket の到達性チェックに適用されます。
* `attachOnly: true` は「ローカルブラウザーを起動せず、すでに動作している場合にのみ attach する」ことを意味します。
* `color` とプロファイルごとの `color` によってブラウザー UI が着色され、どのプロファイルがアクティブか判別しやすくなります。
* デフォルトプロファイルは `chrome`（拡張機能リレー）です。管理対象ブラウザーには `defaultProfile: "openclaw"` を使用します。
* 自動検出の順序: システムのデフォルトブラウザーが Chromium ベースならそれを使用し、それ以外の場合は Chrome → Brave → Edge → Chromium → Chrome Canary の順に試行します。
* ローカルの `openclaw` プロファイルでは `cdpPort` / `cdpUrl` は自動割り当てされます。これらはリモート CDP 用にのみ設定してください。

<div id="use-brave-or-another-chromium-based-browser">
  ## Brave（または他の Chromium ベースのブラウザ）を使う
</div>

**システムのデフォルト** ブラウザが Chromium ベース（Chrome/Brave/Edge など）の場合、
OpenClaw はそれを自動的に使用します。自動検出を上書きしたい場合は、`browser.executablePath` を設定します:

CLI の例:

```bash
openclaw config set browser.executablePath "/usr/bin/google-chrome"
```

```json5
// macOS
{
  browser: {
    executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser"
  }
}

// Windows
{
  browser: {
    executablePath: "C:\\Program Files\\BraveSoftware\\Brave-Browser\\Application\\brave.exe"
  }
}

// Linux
{
  browser: {
    executablePath: "/usr/bin/brave-browser"
  }
}
```

<div id="local-vs-remote-control">
  ## ローカル制御とリモート制御
</div>

* **ローカル制御（デフォルト）：** Gateway がループバック制御サービスを起動し、ローカルのブラウザを立ち上げられます。
* **リモート制御（ノードホスト）：** ブラウザがあるマシン上でノードホストを実行し、Gateway がそのノード経由でブラウザ操作をプロキシします。
* **リモート CDP：** `browser.profiles.<name>.cdpUrl`（または `browser.cdpUrl`）を設定して、
  リモートの Chromium ベースブラウザにアタッチします。この場合、OpenClaw はローカルのブラウザを起動しません。

リモート CDP の URL には認証情報を含められます：

* クエリトークン（例：`https://provider.example?token=<token>`）
* HTTP Basic 認証（例：`https://user:pass@provider.example`）

OpenClaw は `/json/*` エンドポイントを呼び出す際と CDP WebSocket へ接続する際に、
これらの認証情報を引き継ぎます。トークンは設定ファイルにコミットするのではなく、
環境変数やシークレットマネージャーを使用することを推奨します。

<div id="node-browser-proxy-zero-config-default">
  ## ノードブラウザプロキシ（ゼロコンフィグのデフォルト）
</div>

ブラウザを使用しているマシン上で**ノードホスト**を実行している場合、OpenClaw は
追加のブラウザ設定なしに、そのノードへブラウザツール呼び出しを自動的にルーティングできます。
これはリモート Gateway 向けのデフォルトの経路です。

注意:

* ノードホストは、**proxy コマンド**を通じてローカルのブラウザ制御サーバーを公開します。
* プロファイルは、そのノード自身の `browser.profiles` 設定（ローカルと同じ）から取得されます。
* 不要な場合は無効化してください:
  * ノード側: `nodeHost.browserProxy.enabled=false`
  * Gateway 側: `gateway.nodes.browser.mode="off"`

<div id="browserless-hosted-remote-cdp">
  ## Browserless（ホスト型リモート CDP）
</div>

[Browserless](https://browserless.io) は、CDP エンドポイントを HTTPS 経由で公開するホスト型の Chromium サービスです。OpenClaw のブラウザープロファイルを Browserless のリージョンエンドポイントに指定し、API キーで認証できます。

例:

```json5
{
  browser: {
    enabled: true,
    defaultProfile: "browserless",
    remoteCdpTimeoutMs: 2000,
    remoteCdpHandshakeTimeoutMs: 4000,
    profiles: {
      browserless: {
        cdpUrl: "https://production-sfo.browserless.io?token=<BROWSERLESS_API_KEY>",
        color: "#00AA00"
      }
    }
  }
}
```

Notes:

* `<BROWSERLESS_API_KEY>` を有効な Browserless トークンに置き換えてください。
* 利用中の Browserless アカウントのリージョンに対応するエンドポイントを選択してください（詳細は Browserless のドキュメントを参照してください）。

<div id="security">
  ## セキュリティ
</div>

重要なポイント:

* ブラウザ制御はループバック経由のみで行われ、アクセスは Gateway の認証またはノードのペアリングを通じて制御されます。
* Gateway とすべてのノードのホストはプライベートネットワーク (例: Tailscale) 上に配置し、公開インターネットへの直接露出は避けてください。
* リモート CDP の URL／トークンは機密情報として扱い、環境変数やシークレットマネージャーの利用を優先してください。

リモート CDP のヒント:

* 可能な限り HTTPS エンドポイントと有効期間の短いトークンを使用してください。
* 長期間有効なトークンを設定ファイル内に直接埋め込むことは避けてください。

<div id="profiles-multi-browser">
  ## プロファイル（マルチブラウザ）
</div>

OpenClaw は複数の名前付きプロファイル（ルーティング設定）をサポートします。プロファイルには次の種類があります:

* **openclaw-managed**: 独自のユーザーデータディレクトリと CDP ポートを持つ、専用の Chromium ベースのブラウザインスタンス
* **remote**: 明示的な CDP URL（別の場所で動作している Chromium ベースのブラウザ）
* **extension relay**: ローカルのリレー + Chrome 拡張機能経由で、既存の Chrome タブを利用

デフォルト:

* `openclaw` プロファイルは、存在しない場合に自動作成されます。
* `chrome` プロファイルは Chrome extension relay 用の組み込みプロファイルです（デフォルトで `http://127.0.0.1:18792` を指します）。
* ローカル CDP ポートは、デフォルトで **18800–18899** の範囲から割り当てられます。
* プロファイルを削除すると、そのローカルデータディレクトリはゴミ箱に移動されます。

すべてのコントロールエンドポイントは `?profile=<name>` を受け付けます。CLI では `--browser-profile` を使用します。

<div id="chrome-extension-relay-use-your-existing-chrome">
  ## Chrome extension relay (use your existing Chrome)
</div>

OpenClaw は、ローカルの CDP リレー + Chrome 拡張機能を利用して、（別の「openclaw」用 Chrome インスタンスではなく）**既存の Chrome タブ**を操作することもできます。

詳細ガイド: [Chrome extension](/ja/tools/chrome-extension)

フロー:

* Gateway をローカル（同一マシン）で動かすか、ブラウザが動作しているマシン上でノードホストを実行します。
* ローカルの **リレーサーバー** がループバック `cdpUrl`（デフォルト: `http://127.0.0.1:18792`）で待ち受けます。
* 接続したいタブ上で **OpenClaw Browser Relay** 拡張機能のアイコンをクリックしてアタッチします（自動的にはアタッチされません）。
* エージェントは、適切なプロファイルを選択することで、通常の `browser` ツール経由でそのタブを制御します。

Gateway が別ホストで動作している場合は、Gateway がブラウザ操作をプロキシできるように、ブラウザが動作しているマシン上でノードホストを実行してください。

<div id="sandboxed-sessions">
  ### サンドボックス化されたセッション
</div>

エージェントのセッションがサンドボックス化されている場合、`browser` ツールはデフォルトで `target="sandbox"`（サンドボックスブラウザ）を使用するように設定されています。
Chrome extension relay のテイクオーバーにはホスト側ブラウザの制御が必要なため、次のいずれかを行ってください:

* セッションをサンドボックス化せずに実行する、または
* `agents.defaults.sandbox.browser.allowHostControl: true` を設定し、ツールを呼び出す際に `target="host"` を使用する

<div id="setup">
  ### セットアップ
</div>

1. 拡張機能を読み込む（開発モード／パッケージ化されていない拡張機能）:

```bash
openclaw browser extension install
```

* Chrome → `chrome://extensions` → 「デベロッパーモード」をオンにする
* 「パッケージ化されていない拡張機能を読み込む」→ `openclaw browser extension path` が出力したディレクトリを選択
* 拡張機能をピン留めし、制御したいタブ上でアイコンをクリックする（バッジに `ON` と表示される）。

2. 使い方:

* CLI: `openclaw browser --browser-profile chrome tabs`
* エージェントツール: `browser` を使用する（`profile="chrome"` を指定）

任意: 別の名前やリレー用ポートを使いたい場合は、自分用のプロファイルを作成する:

```bash
openclaw browser create-profile \
  --name my-chrome \
  --driver extension \
  --cdp-url http://127.0.0.1:18792 \
  --color "#00AA00"
```

注意:

* このモードは、ほとんどの操作（スクリーンショット／スナップショット／アクション）に Playwright-on-CDP を使用します。
* 拡張機能アイコンをもう一度クリックすると切り離せます。

<div id="isolation-guarantees">
  ## 分離に関する保証
</div>

* **専用のユーザーデータディレクトリ**: 個人のブラウザプロファイルには一切触れません。
* **専用ポート**: 開発ワークフローとの競合を避けるため、`9222` は使用しません。
* **決定論的なタブ制御**: 「最後に開いたタブ」ではなく、`targetId` で対象タブを指定します。

<div id="browser-selection">
  ## ブラウザの選択
</div>

ローカルで起動する場合、OpenClaw は利用可能なブラウザのうち、次の順で最初に見つかったものを選択します:

1. Chrome
2. Brave
3. Edge
4. Chromium
5. Chrome Canary

`browser.executablePath` で明示的に指定できます。

プラットフォーム別の探索先:

* macOS: `/Applications` と `~/Applications` を確認します。
* Linux: `google-chrome`、`brave`、`microsoft-edge`、`chromium` などを探します。
* Windows: 一般的なインストール場所を確認します。

<div id="control-api-optional">
  ## Control API (任意)
</div>

ローカル連携向けにのみ、Gateway は小さなループバック HTTP API を提供します:

* ステータス/開始/停止: `GET /`, `POST /start`, `POST /stop`
* タブ: `GET /tabs`, `POST /tabs/open`, `POST /tabs/focus`, `DELETE /tabs/:targetId`
* スナップショット/スクリーンショット: `GET /snapshot`, `POST /screenshot`
* アクション: `POST /navigate`, `POST /act`
* フック: `POST /hooks/file-chooser`, `POST /hooks/dialog`
* ダウンロード: `POST /download`, `POST /wait/download`
* デバッグ: `GET /console`, `POST /pdf`
* デバッグ: `GET /errors`, `GET /requests`, `POST /trace/start`, `POST /trace/stop`, `POST /highlight`
* ネットワーク: `POST /response/body`
* 状態: `GET /cookies`, `POST /cookies/set`, `POST /cookies/clear`
* 状態: `GET /storage/:kind`, `POST /storage/:kind/set`, `POST /storage/:kind/clear`
* 設定: `POST /set/offline`, `POST /set/headers`, `POST /set/credentials`, `POST /set/geolocation`, `POST /set/media`, `POST /set/timezone`, `POST /set/locale`, `POST /set/device`

すべてのエンドポイントは `?profile=<name>` クエリパラメータを受け付けます。

<div id="playwright-requirement">
  ### Playwright の要件
</div>

一部の機能（navigate/act/AI snapshot/role snapshot、要素スクリーンショット、PDF）には
Playwright が必要です。Playwright がインストールされていない場合、それらのエンドポイントはわかりやすい 501
エラー応答を返します。ARIA スナップショットと基本的なスクリーンショットは、openclaw が管理する Chrome では引き続き動作します。
Chrome 拡張機能リレー・ドライバーの場合、ARIA スナップショットとスクリーンショットには Playwright が必要です。

`Playwright is not available in this gateway build` と表示される場合は、完全な
Playwright パッケージ（`playwright-core` ではない）をインストールして Gateway を再起動するか、
ブラウザーサポートを有効にして OpenClaw を再インストールしてください。

<div id="how-it-works-internal">
  ## 仕組み（内部動作）
</div>

高レベルな処理フロー:

* 小さな **制御サーバー** が HTTP リクエストを受け付けます。
* このサーバーが **CDP** 経由で Chromium ベースのブラウザ（Chrome/Brave/Edge/Chromium）に接続します。
* 高度なアクション（クリック/入力/スナップショット/PDF）のために、CDP の上に **Playwright** を重ねて使用します。
* Playwright がインストールされていない場合は、Playwright を使わない操作のみが利用可能です。

この設計により、エージェントは安定かつ決定論的なインターフェイス上に保たれつつ、
ローカル/リモートのブラウザやプロファイルを差し替えることができます。

<div id="cli-quick-reference">
  ## CLI クイックリファレンス
</div>

すべてのコマンドは、対象とするプロファイルを指定するために `--browser-profile <name>` オプションを受け付けます。
すべてのコマンドは、マシンリーダブルな出力（安定したペイロード）のために `--json` も受け付けます。

基本コマンド:

* `openclaw browser status`
* `openclaw browser start`
* `openclaw browser stop`
* `openclaw browser tabs`
* `openclaw browser tab`
* `openclaw browser tab new`
* `openclaw browser tab select 2`
* `openclaw browser tab close 2`
* `openclaw browser open https://example.com`
* `openclaw browser focus abcd1234`
* `openclaw browser close abcd1234`

検査・インスペクション:

* `openclaw browser screenshot`
* `openclaw browser screenshot --full-page`
* `openclaw browser screenshot --ref 12`
* `openclaw browser screenshot --ref e12`
* `openclaw browser snapshot`
* `openclaw browser snapshot --format aria --limit 200`
* `openclaw browser snapshot --interactive --compact --depth 6`
* `openclaw browser snapshot --efficient`
* `openclaw browser snapshot --labels`
* `openclaw browser snapshot --selector "#main" --interactive`
* `openclaw browser snapshot --frame "iframe#main" --interactive`
* `openclaw browser console --level error`
* `openclaw browser errors --clear`
* `openclaw browser requests --filter api --clear`
* `openclaw browser pdf`
* `openclaw browser responsebody "**/api" --max-chars 5000`

操作:

* `openclaw browser navigate https://example.com`
* `openclaw browser resize 1280 720`
* `openclaw browser click 12 --double`
* `openclaw browser click e12 --double`
* `openclaw browser type 23 "hello" --submit`
* `openclaw browser press Enter`
* `openclaw browser hover 44`
* `openclaw browser scrollintoview e12`
* `openclaw browser drag 10 11`
* `openclaw browser select 9 OptionA OptionB`
* `openclaw browser download e12 /tmp/report.pdf`
* `openclaw browser waitfordownload /tmp/report.pdf`
* `openclaw browser upload /tmp/file.pdf`
* `openclaw browser fill --fields '[{"ref":"1","type":"text","value":"Ada"}]'`
* `openclaw browser dialog --accept`
* `openclaw browser wait --text "Done"`
* `openclaw browser wait "#main" --url "**/dash" --load networkidle --fn "window.ready===true"`
* `openclaw browser evaluate --fn '(el) => el.textContent' --ref 7`
* `openclaw browser highlight e12`
* `openclaw browser trace start`
* `openclaw browser trace stop`

状態:

* `openclaw browser cookies`
* `openclaw browser cookies set session abc123 --url "https://example.com"`
* `openclaw browser cookies clear`
* `openclaw browser storage local get`
* `openclaw browser storage local set theme dark`
* `openclaw browser storage session clear`
* `openclaw browser set offline on`
* `openclaw browser set headers --json '{"X-Debug":"1"}'`
* `openclaw browser set credentials user pass`
* `openclaw browser set credentials --clear`
* `openclaw browser set geo 37.7749 -122.4194 --origin "https://example.com"`
* `openclaw browser set geo --clear`
* `openclaw browser set media dark`
* `openclaw browser set timezone America/New_York`
* `openclaw browser set locale en-US`
* `openclaw browser set device "iPhone 14"`

備考:

* `upload` と `dialog` は **アーミング（arming）** 呼び出しです。ファイル選択ダイアログ／チューザーを起動するクリック／キー押下を行う**前に**実行してください。
* `upload` は `--input-ref` または `--element` を使って、ファイル入力要素に直接ファイルを設定することもできます。
* `snapshot`:
  * `--format ai`（Playwright がインストールされている場合のデフォルト）: 数値の ref（`aria-ref="<n>"`）付きの AI スナップショットを返します。
  * `--format aria`: アクセシビリティツリーを返します（ref なし・検査専用）。
  * `--efficient`（または `--mode efficient`）: コンパクトなロールスナップショットのプリセットです（interactive + compact + depth + 低い maxChars）。
  * デフォルト設定（tool/CLI のみ）: 呼び出し側が mode を渡さない場合に効率的なスナップショットを使うには、`browser.snapshotDefaults.mode: "efficient"` を設定します（[Gateway configuration](/ja/gateway/configuration#browser-openclaw-managed-browser) を参照）。
  * ロールスナップショットのオプション（`--interactive`, `--compact`, `--depth`, `--selector`）は、`ref=e12` のような ref を含むロールベースのスナップショット出力を強制します。
  * `--frame "<iframe selector>"` は、ロールスナップショットの対象を特定の iframe にスコープします（`e12` のようなロール ref と組み合わせて使用）。
  * `--interactive` は、操作対象として選びやすいインタラクティブ要素のフラットなリストを出力します（アクションを駆動する用途に最適）。
  * `--labels` は、ref ラベルをオーバーレイしたビューポート領域のスクリーンショットを追加します（`MEDIA:<path>` を出力）。
* `click` / `type` / そのほかのアクションは、`snapshot` から取得した `ref`（数値 `12` またはロール ref `e12` のどちらか）が必要です。
  CSS セレクタによるアクションは、意図的にサポートしていません。

<div id="snapshots-and-refs">
  ## スナップショットと参照
</div>

OpenClaw は 2 種類の「スナップショット」スタイルをサポートしています:

* **AI スナップショット（数値参照）**: `openclaw browser snapshot`（デフォルト; `--format ai`）
  * 出力: 数値参照を含むテキストスナップショット。
  * アクション: `openclaw browser click 12`, `openclaw browser type 23 "hello"`。
  * 内部的には、参照は Playwright の `aria-ref` を通じて解決されます。

* **ロールスナップショット（`e12` のようなロール参照）**: `openclaw browser snapshot --interactive`（または `--compact`, `--depth`, `--selector`, `--frame`）
  * 出力: `[ref=e12]`（およびオプションの `[nth=1]`）を含むロールベースのリスト/ツリー。
  * アクション: `openclaw browser click e12`, `openclaw browser highlight e12`。
  * 内部的には、参照は `getByRole(...)`（および重複に対する `nth()`）を通じて解決されます。
  * `--labels` を追加すると、`e12` ラベルを重ねて表示したビューポートのスクリーンショットが含まれます。

参照の挙動:

* 参照は**ナビゲーションをまたいで安定しません**。失敗した場合は、`snapshot` を再実行して新しい参照を使用してください。
* ロールスナップショットを `--frame` 付きで取得した場合、ロール参照は次のロールスナップショットが取得されるまで、その iframe にスコープされます。

<div id="wait-power-ups">
  ## 待機のパワーアップ
</div>

待機対象は時間やテキストだけではありません:

* URL が一致するまで待機（Playwright の glob パターンに対応）:
  * `openclaw browser wait --url "**/dash"`
* 特定の読み込み状態になるまで待機:
  * `openclaw browser wait --load networkidle`
* JS の述語が真になるまで待機:
  * `openclaw browser wait --fn "window.ready===true"`
* セレクタが表示されるまで待機:
  * `openclaw browser wait "#main"`

これらは組み合わせて指定できます:

```bash
openclaw browser wait "#main" \
  --url "**/dash" \
  --load networkidle \
  --fn "window.ready===true" \
  --timeout-ms 15000
```

<div id="debug-workflows">
  ## デバッグ用ワークフロー
</div>

アクションが失敗した場合（例: 「not visible」「strict mode violation」「covered」など）:

1. `openclaw browser snapshot --interactive`
2. `click <ref>` / `type <ref>` を使用する（インタラクティブモードでは role の参照を優先して使用する）
3. それでも失敗する場合: `openclaw browser highlight <ref>` を実行して、Playwright がどこをターゲットにしているか確認する
4. ページの挙動がおかしい場合:
   * `openclaw browser errors --clear`
   * `openclaw browser requests --filter api --clear`
5. 詳細なデバッグが必要な場合は、トレースを記録する:
   * `openclaw browser trace start`
   * 問題を再現する
   * `openclaw browser trace stop`（`TRACE:<path>` が出力される）

<div id="json-output">
  ## JSON 出力
</div>

`--json` は、スクリプトや構造化されたツールから扱うためのオプションです。

例:

```bash
openclaw browser status --json
openclaw browser snapshot --interactive --json
openclaw browser requests --filter api --json
openclaw browser cookies --json
```

JSON 形式のロールスナップショットには、`refs` に加えて、小さな `stats` ブロック（lines/chars/refs/interactive）が含まれており、ツールがペイロードのサイズや密度を判断できるようになっています。

<div id="state-and-environment-knobs">
  ## 状態と環境の調整用オプション
</div>

これらは「サイトの挙動を X のようにする」ワークフローに有用です:

* Cookies: `cookies`, `cookies set`, `cookies clear`
* Storage: `storage local|session get|set|clear`
* オフライン: `set offline on|off`
* ヘッダー: `set headers --json '{"X-Debug":"1"}'`（または `--clear`）
* HTTP ベーシック認証: `set credentials user pass`（または `--clear`）
* 位置情報: `set geo <lat> <lon> --origin "https://example.com"`（または `--clear`）
* メディア: `set media dark|light|no-preference|none`
* タイムゾーン / ロケール: `set timezone ...`, `set locale ...`
* デバイス / ビューポート:
  * `set device "iPhone 14"`（Playwright のデバイスプリセット）
  * `set viewport 1280 720`

<div id="security-privacy">
  ## セキュリティとプライバシー
</div>

* openclaw のブラウザープロファイルには、ログイン済みのセッションが含まれる場合があります。機密情報として扱ってください。
* `browser act kind=evaluate` / `openclaw browser evaluate` と `wait --fn` は、
  ページコンテキスト内で任意の JavaScript を実行します。プロンプトインジェクションによって挙動を誘導される可能性があります。
  不要な場合は `browser.evaluateEnabled=false` で無効化してください。
* ログインやボット対策に関する注意事項（X/Twitter など）については、[Browser login + X/Twitter posting](/ja/tools/browser-login) を参照してください。
* Gateway/ノードをホストしているマシンは非公開に保ってください（ループバックまたは tailnet 限定など）。
* リモート CDP エンドポイントは強力です。必ずトンネル経由（例: SSH トンネルなど）とし、適切に保護してください。

<div id="troubleshooting">
  ## トラブルシューティング
</div>

Linux 特有の問題（特に snap 版 Chromium）については、
[ブラウザーのトラブルシューティング](/ja/tools/browser-linux-troubleshooting)
を参照してください。

<div id="agent-tools-how-control-works">
  ## エージェントのツールと制御の仕組み
</div>

エージェントにはブラウザ自動化用の **1つのツール** だけが用意されています:

* `browser` — status/start/stop/tabs/open/focus/close/snapshot/screenshot/navigate/act

動作の対応関係は次のとおりです:

* `browser snapshot` は安定した UI ツリー（AI または ARIA ベース）を返します。
* `browser act` はスナップショットの `ref` ID を使って click/type/drag/select を実行します。
* `browser screenshot` はピクセル（ページ全体または要素）をキャプチャします。
* `browser` は次を受け付けます:
  * 名前付きブラウザプロファイルを選択するための `profile`（openclaw, chrome, または remote CDP）。
  * ブラウザがどこで動作するかを選択するための `target`（`sandbox` | `host` | `node`）。
  * サンドボックス化されたセッションでは、`target: "host"` には `agents.defaults.sandbox.browser.allowHostControl=true` が必要です。
  * `target` が省略された場合: サンドボックス化セッションはデフォルトで `sandbox`、非サンドボックスセッションはデフォルトで `host` になります。
  * ブラウザ機能を持つノードが接続されている場合、`target="host"` または `target="node"` を固定しない限り、ツールは自動的にそのノードへルーティングされる場合があります。

これにより、エージェントの動作は決定的になり、壊れやすいセレクタへの依存を避けられます。