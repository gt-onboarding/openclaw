---
title: Exec ホスト
summary: "リファクタリング計画: exec ホストのルーティング、ノード承認、ヘッドレスランナー"
read_when:
  - exec ホストのルーティングや exec 承認を設計するとき
  - ノードランナーと UI 間の IPC を実装するとき
  - exec ホストのセキュリティモードやスラッシュコマンドを追加するとき
---

<div id="exec-host-refactor-plan">
  # Exec ホストのリファクタリング計画
</div>

<div id="goals">
  ## 目標
</div>

* 実行を**サンドボックス**、**Gateway**、および**ノード**間でルーティングするために、`exec.host` と `exec.security` を追加する。
* デフォルト設定を**安全**に保つ: 明示的に有効化されない限りクロスホスト実行は行わない。
* 実行を、ローカル IPC 経由でオプションの UI (macOS アプリ) を提供する**ヘッドレス実行サービス**として分離する。
* **エージェントごと**のポリシー、許可リスト、ask モード、ノードのバインディングを提供する。
* 許可リストの**有無にかかわらず**動作する **ask モード** をサポートする。
* クロスプラットフォーム: Unix ソケット + トークン認証 (macOS/Linux/Windows 間で同等の動作を確保)。

<div id="non-goals">
  ## 非目標
</div>

* レガシーの許可リスト移行やレガシースキーマのサポートは行わない。
* ノードでの exec に対する PTY/ストリーミングは提供しない（出力は集約済みのもののみ）。
* 既存の Bridge と Gateway を超える新たなネットワークレイヤーは導入しない。

<div id="decisions-locked">
  ## 決定事項（確定）
</div>

* **設定キー:** `exec.host` + `exec.security`（エージェントごとの上書きを許可）。
* **昇格:** Gateway へのフルアクセス用エイリアスとして `/elevated` を維持する。
* **確認のデフォルト:** `on-miss`。
* **承認ストア:** `~/.openclaw/exec-approvals.json`（JSON、旧形式からの移行なし）。
* **Runner:** ヘッドレスのシステムサービス。UI アプリは承認用の Unix ソケットをホストする。
* **ノード ID:** 既存の `nodeId` を使用する。
* **ソケット認証:** Unix ソケット + トークン（クロスプラットフォーム）。必要であれば後で方式を分割。
* **ノードホスト状態:** `~/.openclaw/node.json`（ノード ID + ペアリングトークン）。
* **macOS exec host:** macOS アプリ内で `system.run` を実行し、ノードホストサービスがローカル IPC 経由でリクエストを転送する。
* **XPC helper は使用しない:** Unix ソケット + トークン + ピアチェックに統一する。

<div id="key-concepts">
  ## 重要な概念
</div>

<div id="host">
  ### ホスト
</div>

* `sandbox`: Docker exec（現在の挙動）。
* `gateway`: Gateway ホスト上での exec。
* `node`: Bridge（`system.run`）経由でのノードランナー上での exec。

<div id="security-mode">
  ### セキュリティモード
</div>

* `deny`: 常に拒否します。
* `allowlist`: 一致するものだけを許可します。
* `full`: すべてを許可します（`elevated` と同等）。

<div id="ask-mode">
  ### Ask モード
</div>

* `off`: 一切確認しない。
* `on-miss`: 許可リストに一致しない場合にだけ確認する。
* `always`: 毎回確認する。

Ask は許可リストとは**独立**して動作し、許可リストは `always` や `on-miss` と併用できる。

<div id="policy-resolution-per-exec">
  ### ポリシー解決（exec 単位）
</div>

1. `exec.host` を解決する（tool param → エージェントによる override → グローバルのデフォルト値）。
2. `exec.security` と `exec.ask` を解決する（同じ優先度）。
3. host が `sandbox` の場合、ローカルサンドボックスでの exec 実行に進む。
4. host が `gateway` または `node` の場合、そのホスト上で `security` + `ask` ポリシーを適用する。

<div id="default-safety">
  ## デフォルトのセーフティ
</div>

* デフォルトでは `exec.host = sandbox`。
* デフォルトでは `gateway` および `node` に対して `exec.security = deny`。
* デフォルトでは `exec.ask = on-miss`（セキュリティで許可されている場合にのみ有効）。
* ノードバインディングが設定されていない場合、**エージェントは任意のノードをターゲットにできる**が、ポリシーで許可されている場合に限る。

<div id="config-surface">
  ## 設定項目
</div>

<div id="tool-parameters">
  ### ツールパラメータ
</div>

* `exec.host` (任意): `sandbox | gateway | node` のいずれか。
* `exec.security` (任意): `deny | allowlist | full` のいずれか。
* `exec.ask` (任意): `off | on-miss | always` のいずれか。
* `exec.node` (任意): `host=node` の場合に使用するノードの ID または名前。

<div id="config-keys-global">
  ### 設定キー（グローバル）
</div>

* `tools.exec.host`
* `tools.exec.security`
* `tools.exec.ask`
* `tools.exec.node`（デフォルトのノード割り当て）

<div id="config-keys-per-agent">
  ### 設定キー（エージェント単位）
</div>

* `agents.list[].tools.exec.host`
* `agents.list[].tools.exec.security`
* `agents.list[].tools.exec.ask`
* `agents.list[].tools.exec.node`

<div id="alias">
  ### エイリアス
</div>

* `/elevated on` = エージェントセッションに対して `tools.exec.host=gateway`、`tools.exec.security=full` を設定する。
* `/elevated off` = エージェントセッションの実行設定を元の状態に戻す。

<div id="approvals-store-json">
  ## 承認ストア（JSON）
</div>

パス: `~/.openclaw/exec-approvals.json`

目的:

* **実行ホスト**（Gateway またはノードランナー）向けのローカルポリシー + 許可リスト。
* UI が利用できない場合の問い合わせ用フォールバック。
* UI クライアント向けの IPC 用認証情報。

提案スキーマ（v1）:

```json
{
  "version": 1,
  "socket": {
    "path": "~/.openclaw/exec-approvals.sock",
    "token": "base64-opaque-token"
  },
  "defaults": {
    "security": "deny",
    "ask": "on-miss",
    "askFallback": "deny"
  },
  "agents": {
    "agent-id-1": {
      "security": "allowlist",
      "ask": "on-miss",
      "allowlist": [
        {
          "pattern": "~/Projects/**/bin/rg",
          "lastUsedAt": 0,
          "lastUsedCommand": "rg -n TODO",
          "lastResolvedPath": "/Users/user/Projects/.../bin/rg"
        }
      ]
    }
  }
}
```

Notes:

* 旧来の許可リスト形式は使用しないこと。
* `askFallback` は、`ask` が必須で、かつ UI にアクセスできない場合にのみ適用される。
* ファイルパーミッション: `0600`。

<div id="runner-service-headless">
  ## Runner サービス（ヘッドレス）
</div>

<div id="role">
  ### 役割
</div>

* ローカルで `exec.security` と `exec.ask` を強制適用する。
* システムコマンドを実行し、その出力を返す。
* exec のライフサイクルに対する Bridge イベントを送出する（任意だが推奨）。

<div id="service-lifecycle">
  ### サービスライフサイクル
</div>

* macOS では Launchd/daemon、Linux/Windows ではシステムサービスとして動作する。
* Approvals JSON は実行ホストローカルにのみ保持される。
* UI はローカルの Unix ソケットをホストし、runner は必要に応じて接続する。

<div id="ui-integration-macos-app">
  ## UI 連携（macOS アプリ）
</div>

<div id="ipc">
  ### IPC
</div>

* `~/.openclaw/exec-approvals.sock` (0600) にある Unix ソケット。
* トークンは `exec-approvals.json` (0600) に保存される。
* ピアチェック: 同一 UID のみ許可。
* チャレンジ/レスポンス: リプレイ攻撃防止のため、nonce + HMAC(token, request-hash) を使用。
* 短い TTL（例: 10 秒）+ ペイロードサイズの上限 + レート制限。

<div id="ask-flow-macos-app-exec-host">
  ### Ask フロー (macOS アプリ exec ホスト)
</div>

1. ノードサービスが Gateway から `system.run` を受信する。
2. ノードサービスがローカルソケットに接続し、プロンプト/exec リクエストを送信する。
3. アプリがピア + トークン + HMAC + TTL を検証し、必要に応じてダイアログを表示する。
4. アプリが UI コンテキストでコマンドを実行し、出力を返す。
5. ノードサービスが出力を Gateway に返す。

UI が存在しない場合は:

* `askFallback` (`deny|allowlist|full`) を適用する。

<div id="diagram-sci">
  ### 図（SCI）
</div>

```
Agent -> Gateway -> Bridge -> Node Service (TS)
                         |  IPC (UDS + token + HMAC + TTL)
                         v
                     Mac App (UI + TCC + system.run)
```

<div id="node-identity-binding">
  ## ノードの識別とバインディング
</div>

* 既存の Bridge ペアリング由来の `nodeId` を使用する。
* バインディングモデル:
  * `tools.exec.node` によりエージェントを特定のノードに制限する。
  * 未設定の場合、エージェントは任意のノードを選択できる（ポリシーによりデフォルトは引き続き適用される）。
* ノード選択の解決順序:
  * `nodeId` の完全一致
  * 正規化された `displayName`
  * `remoteIp`
  * `nodeId` の接頭辞（6 文字以上）

<div id="eventing">
  ## イベント処理
</div>

<div id="who-sees-events">
  ### 誰がイベントを見られるか
</div>

* システムイベントは**セッションごと**に管理され、次のプロンプト時にエージェントへ渡される。
* Gateway のメモリ内キュー（`enqueueSystemEvent`）に保存される。

<div id="event-text">
  ### イベントメッセージ
</div>

* `Exec started (node=<id>, id=<runId>)`
* `Exec finished (node=<id>, id=<runId>, code=<code>)` + optional output tail
* `Exec denied (node=<id>, id=<runId>, <reason>)`

<div id="transport">
  ### トランスポート
</div>

オプションA（推奨）:

* Runner が Bridge に `exec.started` / `exec.finished` の `event` フレームを送信する。
* Gateway の `handleBridgeEvent` がこれらを `enqueueSystemEvent` にマッピングする。

オプションB:

* Gateway の `exec` ツールがライフサイクルを直接管理する（同期処理のみ）。

<div id="exec-flows">
  ## 実行フロー
</div>

<div id="sandbox-host">
  ### サンドボックスホスト
</div>

* 既存の `exec` の動作（サンドボックス無効時は Docker コンテナまたはホスト上で実行）。
* PTY は非サンドボックスモードでのみサポートされる。

<div id="gateway-host">
  ### Gateway ホスト
</div>

* Gateway プロセスは専用のマシン上で実行される。
* ローカルの `exec-approvals.json`（セキュリティ／承認／許可リスト）を強制する。

<div id="node-host">
  ### ノードホスト
</div>

* Gateway は `system.run` を指定して `node.invoke` を呼び出す。
* Runner はローカル承認ポリシーを適用する。
* Runner は集約された stdout/stderr を返す。
* 開始／終了／拒否に対するオプションの Bridge イベント。

<div id="output-caps">
  ## 出力の上限
</div>

* 結合した stdout と stderr を最大 **200k** に制限し、イベント用として末尾 **20k** を保持する。
* 明確な接尾辞（例: `"… (truncated)"`）を付けて出力を切り詰める。

<div id="slash-commands">
  ## スラッシュコマンド
</div>

* `/exec host=<sandbox|gateway|node> security=<deny|allowlist|full> ask=<off|on-miss|always> node=<id>`
* エージェントごと・セッションごとのオーバーライド。config で保存しないかぎり永続化されません。
* `/elevated on|off|ask|full` は引き続き `host=gateway security=full` のショートカットです（`full` の場合は承認をスキップします）。

<div id="cross-platform-story">
  ## クロスプラットフォーム戦略
</div>

* runner サービスはポータブルな実行ターゲットです。
* UI は任意であり、存在しない場合は `askFallback` が適用されます。
* Windows/Linux は同じ approvals 用 JSON とソケットプロトコルをサポートします。

<div id="implementation-phases">
  ## 実装フェーズ
</div>

<div id="phase-1-config-exec-routing">
  ### フェーズ 1: 設定 + exec ルーティング
</div>

* `exec.host`、`exec.security`、`exec.ask`、`exec.node` 用の設定スキーマを追加する。
* ツールの内部処理を、`exec.host` を参照するように更新する。
* `/exec` スラッシュコマンドを追加し、`/elevated` エイリアスはそのまま残す。

<div id="phase-2-approvals-store-gateway-enforcement">
  ### フェーズ 2: 承認ストア + Gateway による強制適用
</div>

* `exec-approvals.json` のリーダー/ライターを実装する。
* `gateway` ホストに対して、許可リスト + ask モードを強制適用する。
* 出力上限値を追加する。

<div id="phase-3-node-runner-enforcement">
  ### フェーズ 3: ノードランナーの制御強化
</div>

* ノードランナーを更新し、許可リスト + ask を強制適用する。
* macOS アプリの UI に Unix ソケットのプロンプトブリッジを追加する。
* `askFallback` を連携させる。

<div id="phase-4-events">
  ### フェーズ4：イベント
</div>

* exec のライフサイクル用の node → Gateway Bridge イベントを追加する。
* エージェントのプロンプトを `enqueueSystemEvent` にマッピングする。

<div id="phase-5-ui-polish">
  ### フェーズ5: UIのブラッシュアップ
</div>

* Macアプリ: 許可リストエディタ、エージェント別スイッチャー、確認ポリシー用UI。
* ノードバインディング用コントロール (任意)。

<div id="testing-plan">
  ## テスト計画
</div>

* ユニットテスト: 許可リストのマッチング (glob パターン + 大文字小文字を区別しない)。
* ユニットテスト: ポリシー解決時の優先順位 (ツールパラメータ → エージェントのオーバーライド → グローバル設定)。
* 統合テスト: ノードランナーの deny/allow/ask 処理フロー。
* ブリッジイベントテスト: ノードイベント → システムイベントのルーティング。

<div id="open-risks">
  ## 想定されるリスク
</div>

* UI が利用できない場合: `askFallback` が必ず尊重されるようにすること。
* 長時間実行されるコマンド: タイムアウトと出力上限で制御すること。
* 複数ノード環境での曖昧さ: ノードのバインドまたは明示的なノードパラメータがない場合はエラーとすること。

<div id="related-docs">
  ## 関連資料
</div>

* [Exec ツール](/ja/tools/exec)
* [Exec 承認](/ja/tools/exec-approvals)
* [ノード](/ja/nodes)
* [昇格モード](/ja/tools/elevated)