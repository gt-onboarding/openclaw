---
title: プレゼンス
summary: "OpenClaw のプレゼンスエントリがどのように生成・統合・表示されるか"
read_when:
  - Instances タブのデバッグ
  - 重複または古くなったインスタンス行の調査
  - Gateway の WS 接続や system-event ビーコンを変更する
---

<div id="presence">
  # プレゼンス
</div>

OpenClaw の「プレゼンス」は、次の情報を軽量なベストエフォート型のビューとして提供します:

- **Gateway** 本体
- **Gateway に接続しているクライアント**（mac アプリ、WebChat、CLI など）

プレゼンスは主に、macOS アプリの **Instances** タブの表示と、
オペレーターが状況をすばやく把握できるようにするために使用されます。

<div id="presence-fields-what-shows-up">
  ## Presence フィールド（表示内容）
</div>

Presence エントリは、次のようなフィールドを持つ構造化オブジェクトです:

- `instanceId`（任意だが強く推奨）: 安定したクライアントの識別子（通常は `connect.client.instanceId`）
- `host`: 人間にとってわかりやすいホスト名
- `ip`: ベストエフォートの IP アドレス
- `version`: クライアントのバージョン文字列
- `deviceFamily` / `modelIdentifier`: ハードウェアに関する補助情報
- `mode`: `ui`, `webchat`, `cli`, `backend`, `probe`, `test`, `node`, ...
- `lastInputSeconds`: 「最後のユーザー入力からの経過秒数」（取得できる場合）
- `reason`: `self`, `connect`, `node-connected`, `periodic`, ...
- `ts`: 最終更新のタイムスタンプ（エポックからのミリ秒）

<div id="producers-where-presence-comes-from">
  ## プロデューサー（Presence の生成元）
</div>

Presence エントリは複数のソースから生成され、それらが**マージ**されます。

<div id="1-gateway-self-entry">
  ### 1) Gateway 自身のエントリ
</div>

Gateway は起動時に必ず「self」エントリを作成するため、クライアントがまだ接続していない段階でも、UI 上に Gateway ホストが表示されます。

<div id="2-websocket-connect">
  ### 2) WebSocket connect
</div>

すべての WS クライアントは、まず `connect` リクエストを送信します。ハンドシェイクが成功すると、
Gateway はその接続に対するプレゼンスエントリを upsert（存在すれば更新し、なければ作成）します。

<div id="why-oneoff-cli-commands-dont-show-up">
  #### 単発の CLI コマンドが表示されない理由
</div>

CLI は短時間だけ接続する単発コマンドとして使われることがよくあります。Instances リストがそれらで埋め尽くされるのを避けるため、`client.mode === "cli"` はプレゼンスエントリとしては **登録されません**。

<div id="3-system-event-beacons">
  ### 3) `system-event` ビーコン
</div>

クライアントは `system-event` メソッドを使って、より詳細な定期ビーコンを送信できます。macOS アプリはこれを利用してホスト名、IP、および `lastInputSeconds` を報告します。

<div id="4-node-connects-role-node">
  ### 4) ノードが接続する（role: node）
</div>

ノードが `role: node` として Gateway の WebSocket に接続すると、Gateway は
そのノードの presence エントリを upsert します（他の WS クライアントと同じフロー）。

<div id="merge-dedupe-rules-why-instanceid-matters">
  ## マージと重複排除ルール（`instanceId` が重要な理由）
</div>

Presence エントリは 1 つのメモリ内マップに保存されます:

- エントリは **presence key** をキーとして持ちます。
- 最適なキーは、再起動後も維持される安定した `instanceId`（`connect.client.instanceId` に由来）です。
- キーは大文字小文字を区別しません。

クライアントが安定した `instanceId` なしで再接続すると、
**重複した** 行として表示される可能性があります。

<div id="ttl-and-bounded-size">
  ## TTL と上限サイズ
</div>

Presence は意図的に短命な設計になっています:

- **TTL:** 5 分より古いエントリは削除されます
- **最大エントリ数:** 200（最も古いものから順に削除）

これによりリストを最新の状態に保ち、メモリ使用量が無制限に増加するのを防ぎます。

<div id="remotetunnel-caveat-loopback-ips">
  ## リモート/トンネル時の注意事項（ループバック IP）
</div>

クライアントが SSH トンネル／ローカルポートフォワーディング経由で接続する場合、Gateway が
リモートアドレスを `127.0.0.1` として認識することがあります。クライアントが報告した
正しい IP を上書きしないようにするため、ループバックのリモートアドレスは無視されます。

<div id="consumers">
  ## コンシューマー
</div>

<div id="macos-instances-tab">
  ### macOS Instances タブ
</div>

macOS アプリは `system-presence` の出力を表示し、最後の更新からの経過時間に応じて、小さなステータスインジケーター（Active/Idle/Stale）を表示します。

<div id="debugging-tips">
  ## デバッグのヒント
</div>

- 生のリストを表示するには、Gateway に対して `system-presence` を呼び出す。
- 重複が見つかった場合:
  - ハンドシェイクでクライアントが安定した `client.instanceId` を送信していることを確認する
  - 定期ビーコンが同じ `instanceId` を使用していることを確認する
  - 接続由来のエントリで `instanceId` が欠けていないか確認する（欠けている場合は重複していても想定どおり）