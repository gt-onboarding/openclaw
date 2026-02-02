---
title: Clawnet
summary: "Clawnet リファクタリング：ネットワークプロトコル、ロール、認証、承認、アイデンティティの統一"
read_when:
  - ノードとオペレーター用クライアント向けの統一されたネットワークプロトコルを計画しているとき
  - デバイス間の承認、ペアリング、TLS、プレゼンス情報を再設計しているとき
---

<div id="clawnet-refactor-protocol-auth-unification">
  # Clawnet リファクタリング（プロトコルと認証の統一）
</div>

<div id="hi">
  ## こんにちは
</div>

こんにちは、Peter。とても良い方向性です。これで UX がよりシンプルになり、セキュリティもより強化できます。

<div id="purpose">
  ## 目的
</div>

次の内容を 1 つにまとめた厳密なドキュメント:

- 現状: プロトコル、フロー、トラストバウンダリー。
- 課題: 承認、マルチホップルーティング、UI の重複。
- 提案する新しい状態: 単一プロトコル、スコープが定義されたロール、統一された認証/ペアリング、TLS ピンニング。
- アイデンティティモデル: 安定した ID とかわいいスラッグ。
- マイグレーション計画、リスク、未解決の疑問点。

<div id="goals-from-discussion">
  ## 目標（ディスカッションより）
</div>

- すべてのクライアント（Macアプリ、CLI、iOS、Android、ヘッドレスノード）で同一のプロトコルを使用する。
- すべてのネットワーク参加者が認証され、ペアリングされていること。
- ノードとオペレーターの役割を明確にする。
- 集中管理された承認を、ユーザーがいる場所へルーティングする。
- すべてのリモートトラフィックに対して TLS 暗号化＋オプションのピンニングを行う。
- コードの重複を最小限にする。
- 1 台のマシンは 1 回だけ表示されること（UI／ノードの重複エントリがないこと）。

<div id="nongoals-explicit">
  ## 非目標（明示）
</div>

- 機能分離をなくすこと（最小権限の原則は依然として必要）。
- scope チェックなしで Gateway の完全なコントロールプレーンを公開すること。
- 認証を人間可読なラベルに依存させること（slug は今後もセキュリティ用途ではない）。

---

<div id="current-state-asis">
  # 現状（As-Is）
</div>

<div id="two-protocols">
  ## 2つのプロトコル
</div>

<div id="1-gateway-websocket-control-plane">
  ### 1) Gateway WebSocket (コントロールプレーン)
</div>

- API 全体: 設定、チャネル、モデル、セッション、エージェント実行、ログ、ノードなど。
- デフォルトのバインド先: ループバック。リモートアクセスは SSH/Tailscale 経由。
- 認証: `connect` によるトークン/パスワード。
- TLS ピンニングなし（ループバック／トンネルに依存）。
- コード:
  - `src/gateway/server/ws-connection/message-handler.ts`
  - `src/gateway/client.ts`
  - `docs/gateway/protocol.md`

<div id="2-bridge-node-transport">
  ### 2) Bridge (ノードトランスポート)
</div>

- 許可リストの露出範囲を絞り、ノードのアイデンティティとペアリングを前提とする。
- JSONL over TCP。オプションで TLS と証明書フィンガープリントのピンニングをサポート。
- TLS は discovery 用 TXT レコードでフィンガープリントをアドバタイズ。
- Code:
  - `src/infra/bridge/server/connection.ts`
  - `src/gateway/server-bridge.ts`
  - `src/node-host/bridge-client.ts`
  - `docs/gateway/bridge-protocol.md`

<div id="control-plane-clients-today">
  ## 現在のコントロールプレーンのクライアント
</div>

- CLI → Gateway WS（`callGateway` 経由、`src/gateway/call.ts`）。
- macOS app UI → Gateway WS（`GatewayConnection`）。
- Web Control UI → Gateway WS。
- ACP → Gateway WS。
- ブラウザのコントロールは、独自の HTTP コントロールサーバーを使用しています。

<div id="nodes-today">
  ## 現在のノード
</div>

- ノードモードの macOS アプリが Gateway ブリッジ（`MacNodeBridgeSession`）に接続する。
- iOS/Android アプリが Gateway ブリッジに接続する。
- ペアリングとノードごとのトークンが Gateway 側に保存される。

<div id="current-approval-flow-exec">
  ## 現在の承認フロー（exec）
</div>

- エージェントが Gateway 経由で `system.run` を呼び出す。
- Gateway がブリッジ越しにノードを呼び出す。
- ノードのランタイムが承認するかどうかを判定する。
- （ノード == Mac アプリの場合）Mac アプリが UI プロンプトを表示する。
- ノードが `invoke-res` を Gateway に返す。
- 多段ホップ構成で、UI はノードのホストに紐づいている。

<div id="presence-identity-today">
  ## 現在のプレゼンスとアイデンティティ
</div>

- WS クライアントからの Gateway のプレゼンス情報エントリ。
- ブリッジ経由のノードのプレゼンス情報エントリ。
- mac アプリでは、同一マシンに対して 2 つのエントリ（UI + ノード）が表示される場合がある。
- ノードのアイデンティティはペアリングストアに保存され、UI のアイデンティティは別々に管理される。

---

<div id="problems-pain-points">
  # 問題点 / ペインポイント
</div>

- 2つのプロトコルスタック（WS + Bridge）を維持する必要がある。
- リモートノードでの承認: 承認プロンプトがユーザーのいる側ではなくノードホスト側に表示される。
- TLS pinning は Bridge にしか存在せず、WS は SSH/Tailscale に依存している。
- ID の重複: 同じマシンが複数のインスタンスとして表示される。
- 役割があいまい: UI、ノード、CLI の機能が明確に分離されていない。

---

<div id="proposed-new-state-clawnet">
  # 提案されている新しい状態（Clawnet）
</div>

<div id="one-protocol-two-roles">
  ## 1つのプロトコル、2つのロール
</div>

ロールとスコープを持つ単一のWSプロトコル。

- **ロール: ノード**（機能ホスト）
- **ロール: オペレーター**（コントロールプレーン）
- オペレーター向けの任意の**スコープ**:
  - `operator.read`（ステータス確認 + 閲覧）
  - `operator.write`（エージェント実行、送信）
  - `operator.admin`（設定、チャネル、モデル）

<div id="role-behaviors">
  ### ロールの挙動
</div>

**ノード**

- 機能（`caps`、`commands`、`permissions`）を登録できる。
- `invoke` コマンド（`system.run`、`camera.*`、`canvas.*`、`screen.record` など）を受信できる。
- `voice.transcript`、`agent.request`、`chat.subscribe` などのイベントを送信できる。
- config/models/channels/sessions/agent のコントロールプレーン API を呼び出すことはできない。

**オペレーター**

- スコープによって制御された、コントロールプレーン API 全体にアクセスできる。
- すべての承認を受け取る。
- OS アクションを直接実行せず、ノードへルーティングする。

<div id="key-rule">
  ### 重要なルール
</div>

ロールはデバイス単位ではなく接続単位です。1 台のデバイスが、接続ごとに両方のロールを持つことができます。

---

<div id="unified-authentication-pairing">
  # 統合認証とペアリング
</div>

<div id="client-identity">
  ## クライアントの識別情報
</div>

すべてのクライアントは次を含みます:

- `deviceId`（デバイスキーから導出される安定した ID）。
- `displayName`（人間向けの名称）。
- `role` + `scope` + `caps` + `commands`。

<div id="pairing-flow-unified">
  ## ペアリングフロー（統合）
</div>

- クライアントが未認証のまま接続する。
- Gateway はその `deviceId` に対する **ペアリングリクエスト**を作成する。
- オペレーターにプロンプトが表示され、承認または拒否する。
- Gateway は以下にひも付いた認証情報を発行する:
  - デバイスの公開鍵
  - ロール
  - スコープ
  - 機能/コマンド
- クライアントはトークンを保存し、認証済みとして再接続する。

<div id="devicebound-auth-avoid-bearer-token-replay">
  ## デバイス紐づけ認証（ベアラートークンのリプレイ回避）
</div>

推奨：デバイスごとの鍵ペア。

- デバイスが一度だけ鍵ペアを生成する。
- `deviceId = fingerprint(publicKey)`。
- Gateway が nonce を送信し、デバイスが署名し、Gateway が検証する。
- トークンは文字列ではなく、公開鍵（所有証明（proof-of-possession））に対して発行される。

代替案:

- mTLS（クライアント証明書）：最も強力だが、運用の複雑さが増す。
- 有効期限の短いベアラートークンは一時的なフェーズとしてのみ使用する（適宜ローテーションし、早期に失効させる）。

<div id="silent-approval-ssh-heuristic">
  ## サイレント承認（SSH ヒューリスティック）
</div>

弱点にならないよう、挙動を厳密に定義すること。次のいずれか 1 つを選択して用いることを推奨する:

- **ローカルのみ**: クライアントがループバック／Unix ソケット経由で接続した場合に自動ペアリングする。
- **SSH 経由のチャレンジ**: Gateway がノンスを発行し、クライアントはそれを取得することで SSH を証明する。
- **物理的プレゼンス・ウィンドウ**: Gateway ホストの UI 上でローカル承認が行われた後、短時間（例: 10 分間）だけ自動ペアリングを許可する。

自動承認は必ずログに記録すること。

---

<div id="tls-everywhere-dev-prod">
  # すべての環境で TLS を徹底する（開発 + 本番）
</div>

<div id="reuse-existing-bridge-tls">
  ## 既存のブリッジ TLS を再利用する
</div>

現在の TLS ランタイムとフィンガープリントピンニングを使用します:

- `src/infra/bridge/server/tls.ts`
- `src/node-host/bridge-client.ts` 内のフィンガープリント検証ロジック

<div id="apply-to-ws">
  ## WS への適用
</div>

- WS サーバーは、同じ証明書／秘密鍵とフィンガープリントを用いた TLS をサポートする。
- WS クライアントは、フィンガープリントをピン留めできる（任意）。
- Discovery は、すべてのエンドポイントについて TLS とフィンガープリントを通知する。
  - Discovery はロケーション解決のための手掛かりに過ぎず、決して信頼の拠り所（トラストアンカー）にはならない。

<div id="why">
  ## なぜ
</div>

- 機密性の確保を SSH/Tailscale に過度に依存しないようにする。
- リモートからのモバイル接続をデフォルトで安全なものにする。

---

<div id="approvals-redesign-centralized">
  # 承認ワークフローの再設計（集中管理型）
</div>

<div id="current">
  ## 現状
</div>

承認はノードのホスト側（mac アプリのノードランタイム）で行われます。プロンプトはノードが実行されている環境に表示されます。

<div id="proposed">
  ## 提案
</div>

Approval は **Gateway 上でホストされ**、UI はオペレーターのクライアントに配信されます。

<div id="new-flow">
  ### 新しいフロー
</div>

1) Gateway が `system.run` インテント（エージェント）を受信する。
2) Gateway が承認レコード `approval.requested` を作成する。
3) オペレーター向けの UI がプロンプトを表示する。
4) 承認の可否の決定が Gateway に送信される: `approval.resolve`。
5) 承認された場合、Gateway がノードのコマンドを呼び出す。
6) ノードがコマンドを実行し、`invoke-res` を返す。

<div id="approval-semantics-hardening">
  ### 承認セマンティクス（ハードニング）
</div>

- すべてのオペレーターにブロードキャストされるが、モーダルを表示するのはアクティブなUIのみ（その他はトースト通知を受け取る）。
- 最初の応答が有効となり、Gatewayはそれ以降の確定操作を「すでに確定済み」として拒否する。
- デフォルトのタイムアウト: N秒後に拒否（例: 60秒）、理由をログに記録する。
- 承認結果の確定には `operator.approvals` スコープが必要。

<div id="benefits">
  ## 利点
</div>

- プロンプトがユーザーのいる場所（Mac／スマートフォン）に表示される。
- リモートノードに対する承認が一貫している。
- ノードのランタイムはヘッドレスのままで、UI に依存しない。

---

<div id="role-clarity-examples">
  # ロールの明確化例
</div>

<div id="iphone-app">
  ## iPhoneアプリ
</div>

- マイク、カメラ、ボイスチャット、位置情報、プッシュトゥトーク用の **ノードロール**。
- ステータスおよびチャットビュー用のオプションの **operator.read**。
- 明示的に有効化された場合にのみ利用できるオプションの **operator.write/admin**。

<div id="macos-app">
  ## macOS アプリ
</div>

- デフォルトではオペレーター・ロール（Control UI）。
- 「Mac node」を有効化するとノード・ロール（system.run、screen、camera）。
- 両方の接続が同じ deviceId の場合 → UI 上では 1 つのエントリとして統合。

<div id="cli">
  ## CLI
</div>

- ロールは常にオペレーター。
- サブコマンドからスコープが決定される：
  - `status`, `logs` → read
  - `agent`, `message` → write
  - `config`, `channels` → admin
  - 承認 + ペアリング → `operator.approvals` / `operator.pairing`

---

<div id="identity-slugs">
  # ID + スラッグ
</div>

<div id="stable-id">
  ## 固定 ID
</div>

認証に必須で、決して変わらない。
推奨:

- キーペアのフィンガープリント（公開鍵ハッシュ）。

<div id="cute-slug-lobsterthemed">
  ## 可愛いスラッグ（ロブスターテーマ）
</div>

人間が読むためのラベル専用。

- 例: `scarlet-claw`, `saltwave`, `mantis-pinch`
- Gateway レジストリに保存され、編集可能。
- 衝突時の処理: `-2`, `-3`

<div id="ui-grouping">
  ## UI のグルーピング
</div>

異なるロールで `deviceId` が同一の場合 → 1 つの「Instance」行として表示:

- バッジ: `operator`、`node`。
- 対応機能と最終確認時刻を表示。

---

<div id="migration-strategy">
  # マイグレーション戦略
</div>

<div id="phase-0-document-align">
  ## フェーズ 0: ドキュメント化と認識合わせ
</div>

- このドキュメントを公開する。
- すべてのプロトコル呼び出しと承認フローを棚卸しする。

<div id="phase-1-add-rolesscopes-to-ws">
  ## フェーズ 1: WS へのロール/スコープの追加
</div>

- `connect` パラメータを `role`、`scope`、`deviceId` で拡張する。
- ノードのロール向けに、許可リストに基づくアクセス制御を追加する。

<div id="phase-2-bridge-compatibility">
  ## フェーズ 2: ブリッジ互換性
</div>

- ブリッジは稼働させたままにしておく。
- 並行して WS ノードのサポートを追加する。
- 機能は設定フラグで有効／無効を切り替えられるようにする。

<div id="phase-3-central-approvals">
  ## フェーズ 3: 承認の一元化
</div>

- 承認リクエストと resolve イベントを WS に追加する。
- mac アプリの UI を更新し、ユーザーへのプロンプト表示と応答送信を行う。
- ノードのランタイムは UI に直接プロンプトを出さないようにする。

<div id="phase-4-tls-unification">
  ## フェーズ 4: TLS の統一
</div>

- bridge TLS ランタイムを使用して WS 向けの TLS 設定を追加します。
- クライアントに証明書ピンニングを追加します。

<div id="phase-5-deprecate-bridge">
  ## フェーズ5: bridge の非推奨化
</div>

- iOS/Android/macOS ノードを WS に移行する。
- bridge をフォールバックとして残し、安定したら削除する。

<div id="phase-6-devicebound-auth">
  ## フェーズ 6: デバイスバインド認証
</div>

- すべての非ローカル接続に対してキーベースのアイデンティティを必須にする。
- 失効 + ローテーション用の UI を追加する。

---

<div id="security-notes">
  # セキュリティに関する注意事項
</div>

- Gateway の境界でロールと許可リストを強制適用します。
- オペレーターのスコープなしで「フル」API を利用できるクライアントは存在しません。
- *すべての* 接続にはペアリングが必要です。
- TLS と証明書ピンニングにより、モバイル環境での MITM リスクを低減します。
- SSH のサイレント承認は利便性のための機能ですが、処理はすべて記録されており、いつでも取り消し可能です。
- Discovery が信頼のアンカーになることは決してありません。
- ケイパビリティのクレームは、プラットフォーム／タイプごとにサーバー側の許可リストに対して検証されます。

<div id="streaming-large-payloads-node-media">
  # ストリーミングと大きなペイロード（ノードのメディア）
</div>

WS 制御プレーンは小さなメッセージには十分だが、ノードでは次のようなものも扱う:

- カメラクリップ
- 画面録画
- 音声ストリーム

オプション:

1) WS のバイナリフレーム + チャンク分割 + バックプレッシャー制御のルール。
2) 別のストリーミング用エンドポイント（TLS と認証は維持）。
3) メディア負荷の高いコマンドではブリッジを長めに維持し、移行は最後に回す。

設計のブレを防ぐため、実装前にどれか 1 つに決めておくこと。

<div id="capability-command-policy">
  # 機能とコマンドのポリシー
</div>

- ノードが報告する機能／コマンドは **主張（claim）** として扱われる。
- Gateway はプラットフォームごとの許可リストを適用する。
- 新しいコマンドには、オペレーターの承認または明示的な許可リスト変更が必要となる。
- 変更をタイムスタンプ付きで記録・監査する。

<div id="audit-rate-limiting">
  # 監査とレート制限
</div>

- 次をログに記録する: ペアリング要求、承認/却下、トークンの発行/ローテーション/失効。
- ペアリングスパムや承認プロンプトをレート制限する。

<div id="protocol-hygiene">
  # プロトコルの衛生
</div>

- 明示的なプロトコルバージョンとエラーコード。
- 再接続ルールとハートビートポリシー。
- プレゼンス TTL と last-seen（最終確認時刻）のセマンティクス。

---

<div id="open-questions">
  # 未解決事項
</div>

1) 単一デバイスで両ロールを実行する場合: トークンモデル
   - ロールごと（ノード vs オペレーター）に別々のトークンを推奨。
   - 同じ deviceId でスコープを分けることで、失効処理を明確化できる。

2) オペレーターのスコープ粒度
   - read/write/admin + approvals + ペアリング（最小限の構成）。
   - 機能単位のスコープは後で検討する。

3) トークンのローテーションと失効の UX
   - ロール変更時に自動ローテーション。
   - deviceId + ロールで失効できる UI。

4) Discovery
   - 既存の Bonjour TXT を拡張して、WS TLS フィンガープリント + ロールのヒントを含める。
   - 位置特定用のヒントとしてのみ扱う。

5) ネットワークをまたいだ承認
   - すべてのオペレータークライアントにブロードキャストし、アクティブな UI がモーダルを表示。
   - 最初の応答を優先し、Gateway がアトミック性を保証する。

---

<div id="summary-tldr">
  # 要約（TL;DR）
</div>

- 現状: WS 制御プレーン + Bridge ノードのトランスポート。
- 課題: 承認フローの煩雑さ + 重複 + 2 つの別スタック。
- 提案: 明示的なロールとスコープを持つ単一の WS プロトコル、統一されたペアリング + TLS ピニング、Gateway ホスト型の承認フロー、安定したデバイス ID + 覚えやすく愛嬌のあるスラッグ。
- 結果: よりシンプルな UX、より強固なセキュリティ、重複削減、モバイル向けのより良いルーティング。