---
title: 形式検証（セキュリティモデル）
summary: OpenClaw の最もリスクの高いパスに対して機械的に検証されたセキュリティモデル。
permalink: /security/formal-verification/
---

<div id="formal-verification-security-models">
  # 形式検証（セキュリティモデル）
</div>

このページでは、OpenClaw の **形式的セキュリティモデル**（現時点では TLA+/TLC。必要に応じて追加）をまとめています。

> 注: 一部の古いリンクは、以前のプロジェクト名を参照している場合があります。

**ゴール（ノーススター）:** 明示的な前提条件の下で、OpenClaw が
意図したセキュリティポリシー（認可、セッション分離、ツール利用のゲーティング、
誤設定に対する安全性）を強制していることを、機械的に検証された論拠として示すこと。

**現時点での内容:** 攻撃者駆動の実行可能な **セキュリティ回帰テストスイート**:

- 各主張には、有限状態空間上で実行可能なモデルチェックが用意されています。
- 多くの主張には、現実的なバグクラスに対して反例トレースを生成する **ネガティブモデル** が対応付けられています。

**まだ「ない」もの:** 「OpenClaw があらゆる点で安全である」ことや、TypeScript 実装全体が正しいことを示す完全な証明。

<div id="where-the-models-live">
  ## モデルの配置場所
</div>

モデルは別のリポジトリで管理されています: [vignesh07/openclaw-formal-models](https://github.com/vignesh07/openclaw-formal-models)。

<div id="important-caveats">
  ## 重要な注意事項
</div>

- ここで示すのはあくまで**モデル**であり、完全な TypeScript 実装ではありません。モデルとコードの間に乖離が生じる可能性があります。
- 結果は TLC が探索した状態空間の範囲に限定されます。「緑」であることは、モデル化された前提および境界を超えたセキュリティを保証するものではありません。
- 一部の主張は、明示的に記述された環境上の前提条件（例: 正しいデプロイ、正しい設定入力）に依存しています。

<div id="reproducing-results">
  ## 結果の再現
</div>

現在は、models リポジトリをローカルにクローンし、TLC を実行することで結果を再現します（後述）。将来のバージョンでは、次のようなものを提供できる可能性があります:

* CI 上で実行され、公開アーティファクト（反例トレース、実行ログ）を生成するモデル
* 小規模かつ有界な検査向けの、ホスト型の「このモデルを実行する」ワークフロー

開始手順:

```bash
git clone https://github.com/vignesh07/openclaw-formal-models
cd openclaw-formal-models

# Java 11以上が必要です(TLCはJVM上で実行されます)。
# このリポジトリには固定バージョンの `tla2tools.jar` (TLA+ツール)が同梱されており、`bin/tlc` とMakeターゲットが提供されています。

make <target>
```


<div id="gateway-exposure-and-open-gateway-misconfiguration">
  ### Gateway 公開とオープンな Gateway の誤設定
</div>

**主張:** ループバック以外へのバインドを認証なしで行うと、リモートからの侵害を招く可能性があり、露出が増大する。トークン／パスワードによる保護は（このモデルの前提において）未認証の攻撃者をブロックする。

- Green の実行:
  - `make gateway-exposure-v2`
  - `make gateway-exposure-v2-protected`
- Red（想定どおり）:
  - `make gateway-exposure-v2-negative`

関連: models リポジトリ内の `docs/gateway-exposure-matrix.md` を参照。

<div id="nodesrun-pipeline-highest-risk-capability">
  ### Nodes.run パイプライン（最もリスクの高い機能）
</div>

**前提:** `nodes.run` には、(a) ノードコマンドの許可リストおよび宣言済みコマンドと、(b) 設定されている場合はライブ（リアルタイム）承認が必要である。承認は（モデル内で）リプレイ攻撃を防ぐためにトークン化される。

- グリーン（成功することが期待される）実行:
  - `make nodes-pipeline`
  - `make approvals-token`
- レッド（失敗が期待される）実行:
  - `make nodes-pipeline-negative`
  - `make approvals-token-negative`

<div id="pairing-store-dm-gating">
  ### ペアリングストア（DM ゲーティング）
</div>

**主張:** ペアリングリクエストは TTL と保留中リクエスト数の上限を正しく順守する。

- Green（成功パス）:
  - `make pairing`
  - `make pairing-cap`
- Red（失敗が期待されるパス）:
  - `make pairing-negative`
  - `make pairing-cap-negative`

<div id="ingress-gating-mentions-control-command-bypass">
  ### Ingress ゲーティング（メンション + control-command バイパス）
</div>

**主張:** メンション必須のグループコンテキストでは、認可されていない「control command」は、メンションによるゲーティングをバイパスできない。

- Green:
  - `make ingress-gating`
- Red（想定どおり）:
  - `make ingress-gating-negative`

<div id="routingsession-key-isolation">
  ### ルーティング／セッションキー分離
</div>

**主張:** 明示的にリンクまたは設定されない限り、異なるピアからのDMは同じセッションに統合されない。

- Green:
  - `make routing-isolation`
- Red (expected):
  - `make routing-isolation-negative`

<div id="v1-additional-bounded-models-concurrency-retries-trace-correctness">
  ## v1++: 追加の有界モデル（並行性、リトライ、トレースの正確性）
</div>

これらは、実環境での障害モード（非アトミックな更新、リトライ、およびメッセージのファンアウト）をより厳密にモデル化するための追加の有界モデル群です。

<div id="pairing-store-concurrency-idempotency">
  ### ペアリングストアの並行性 / 冪等性
</div>

**主張:** ペアリングストアは、操作がインターリーブ（処理が入り交じる）する状況でも `MaxPending` と冪等性を必ず満たさなければならない（つまり「チェックしてから書く」処理はアトミック / ロック付きでなければならず、refresh によって重複エントリが生成されてはならない）。

ここでの意味は次のとおり:

- 並行リクエスト下でも、チャネルごとの `MaxPending` を超過してはならない。
- 同じ `(channel, sender)` に対する繰り返しのリクエスト / refresh で、有効な pending 行が重複して生成されてはならない。

- Green で通るもの:
  - `make pairing-race`（アトミック / ロック付きの上限チェック）
  - `make pairing-idempotency`
  - `make pairing-refresh`
  - `make pairing-refresh-race`
- Red（意図的に失敗するケース）:
  - `make pairing-race-negative`（非アトミックな begin/commit による上限競合）
  - `make pairing-idempotency-negative`
  - `make pairing-refresh-negative`
  - `make pairing-refresh-race-negative`

<div id="ingress-trace-correlation-idempotency">
  ### Ingress トレース相関 / 冪等性
</div>

**主張:** 取り込みはファンアウト全体でトレース相関を保持し、プロバイダーによるリトライが発生しても冪等であるべきである。

意味するところは次のとおり:

- 1つの外部イベントが複数の内部メッセージにファンアウトされる場合でも、すべての部分が同じトレース／イベント ID を保持する。
- リトライによって二重処理が発生しない。
- プロバイダーのイベント ID が欠落している場合でも、重複排除は安全なキー（例: トレース ID）にフォールバックし、異なるイベントを取りこぼさない。

- Green:
  - `make ingress-trace`
  - `make ingress-trace2`
  - `make ingress-idempotency`
  - `make ingress-dedupe-fallback`
- Red（想定どおり）:
  - `make ingress-trace-negative`
  - `make ingress-trace2-negative`
  - `make ingress-idempotency-negative`
  - `make ingress-dedupe-fallback-negative`

<div id="routing-dmscope-precedence-identitylinks">
  ### Routing dmScope precedence + identityLinks
</div>

**主張:** Routing はデフォルトで DM セッションを分離した状態に保ち、（チャネルの優先順位 + identity links）によって明示的に設定された場合にのみセッションを統合しなければならない。

ここでの意味:

- チャネル固有の dmScope のオーバーライドは、グローバルデフォルトよりも必ず優先されなければならない。
- identityLinks は、無関係なピア同士ではなく、明示的にリンクされたグループ内でのみセッションを統合すべきである。

- Green:
  - `make routing-precedence`
  - `make routing-identitylinks`
- Red (expected):
  - `make routing-precedence-negative`
  - `make routing-identitylinks-negative`