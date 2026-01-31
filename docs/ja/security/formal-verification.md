---
title: 形式検証（セキュリティモデル）
summary: OpenClaw の最もリスクの高いパスに対する機械検証済みセキュリティモデル。
permalink: /security/formal-verification/
---

<div id="formal-verification-security-models">
  # 形式検証（セキュリティモデル）
</div>

このページでは、OpenClaw の**形式的セキュリティモデル**（現状は TLA+/TLC、必要に応じて今後拡張）を追跡します。

> 注: 一部の古いリンクは、以前のプロジェクト名を参照している場合があります。

**目標（北極星）:** 明示的な前提条件の下で、OpenClaw がその意図されたセキュリティポリシー（認可、セッション分離、ツールのゲート制御、構成ミスに対する安全性）を確実に強制していることを、機械検証された論証として提示すること。

**現状これが意味するもの:** 実行可能で、攻撃者駆動の**セキュリティ回帰テストスイート**であること:

- 各主張には、有限状態空間上で実行可能なモデルチェックが付随している。
- 多くの主張には、現実的なバグクラスに対する反例トレースを生成する、対応する**反例モデル（negative model）**がある。

**まだそうではないもの:** 「OpenClaw があらゆる側面で安全である」こと、または TypeScript 実装全体が正しいことの証明ではない。

<div id="where-the-models-live">
  ## モデルの配置場所
</div>

モデルは別のリポジトリで管理されています: [vignesh07/openclaw-formal-models](https://github.com/vignesh07/openclaw-formal-models)。

<div id="important-caveats">
  ## 重要な注意事項
</div>

- ここで示すのはあくまで**モデル**であり、完全な TypeScript 実装そのものではありません。モデルとコードの間に乖離が生じる可能性があります。
- 結果は TLC が探索した状態空間によって制約されています。TLC の結果が「グリーン」であっても、モデル化された前提および境界を超えた安全性を意味するものではありません。
- 一部の主張は、明示的な環境上の前提（例：正しいデプロイ、正しい設定入力）に依存しています。

<div id="reproducing-results">
  ## 結果の再現
</div>

現在は、models リポジトリをローカルにクローンし、TLC を実行することで結果を再現します（後述）。将来のバージョンでは、次のようなものを提供できる可能性があります:

* CI 上で実行されたモデルと、その公開アーティファクト（反例トレース、実行ログ）
* 小規模かつ有限な検査向けの、ホスト型の「このモデルを実行する」ワークフロー

はじめに:

```bash
git clone https://github.com/vignesh07/openclaw-formal-models
cd openclaw-formal-models

# Java 11以上が必要です(TLCはJVM上で実行されます)。
# このリポジトリには固定バージョンの `tla2tools.jar` (TLA+ツール)が同梱されており、`bin/tlc` とMakeターゲットが提供されています。

make <target>
```


<div id="gateway-exposure-and-open-gateway-misconfiguration">
  ### Gateway の公開とオープンな Gateway の誤設定
</div>

**主張:** ループバックアドレス以外へのバインドを認証なしで行うと、リモートからの侵害が可能になり得る／攻撃面が広がる。本モデルの前提に基づけば、トークン／パスワードにより未認証の攻撃者は遮断される。

- Green の実行:
  - `make gateway-exposure-v2`
  - `make gateway-exposure-v2-protected`
- Red（想定どおり）:
  - `make gateway-exposure-v2-negative`

参照: models リポジトリ内の `docs/gateway-exposure-matrix.md`。

<div id="nodesrun-pipeline-highest-risk-capability">
  ### Nodes.run パイプライン（最高リスクの機能）
</div>

**主張:** `nodes.run` には、(a) ノードコマンドの許可リストと宣言済みコマンド、そして (b) 設定されている場合はライブ承認が必要である。承認は（モデル内で）リプレイ攻撃を防ぐためにトークン化される。

- グリーン（成功するケース）:
  - `make nodes-pipeline`
  - `make approvals-token`
- レッド（失敗が想定されるケース）:
  - `make nodes-pipeline-negative`
  - `make approvals-token-negative`

<div id="pairing-store-dm-gating">
  ### ペアリングストア（DM ゲーティング）
</div>

**主張:** ペアリングリクエストは TTL と保留中リクエスト数の上限を正しく守る。

- Green（成功）ラン:
  - `make pairing`
  - `make pairing-cap`
- Red（想定どおり失敗）ラン:
  - `make pairing-negative`
  - `make pairing-cap-negative`

<div id="ingress-gating-mentions-control-command-bypass">
  ### Ingress ゲーティング（メンション + control-command バイパス）
</div>

**主張:** メンションが必須となるグループコンテキストにおいて、未認可の「control command」はメンションによるゲーティングをバイパスできない。

- Green:
  - `make ingress-gating`
- Red（期待どおり）:
  - `make ingress-gating-negative`

<div id="routingsession-key-isolation">
  ### ルーティング／セッションキーの分離
</div>

**主張:** 明示的にリンク／設定しない限り、異なるピアからのDMが同一のセッションに統合されることはない。

- 緑:
  - `make routing-isolation`
- 赤（想定どおり）:
  - `make routing-isolation-negative`

<div id="v1-additional-bounded-models-concurrency-retries-trace-correctness">
  ## v1++: 追加の有界モデル（並行性、リトライ、トレースの正当性）
</div>

これらは、実環境での障害モード（非アトミックな更新、リトライ、メッセージのファンアウト）に対する検証の忠実度をさらに高めるための後続モデルです。

<div id="pairing-store-concurrency-idempotency">
  ### ペアリングストアの並行性 / 冪等性
</div>

**主張:** ペアリングストアは、インターリーブされた実行（すなわち「確認してから書き込み」の処理がアトミック / ロックされていること）の下でも `MaxPending` と冪等性を保証すべきであり、refresh によって重複が生成されてはならない。

この主張が意味すること:

- 並行リクエスト下でも、任意のチャネルについて `MaxPending` を超えてはならない。
- 同じ `(channel, sender)` に対する繰り返しのリクエスト / refresh は、重複する live な pending 行を作成してはならない。

- Green（成功するケース）:
  - `make pairing-race`（アトミック / ロックされた上限チェック）
  - `make pairing-idempotency`
  - `make pairing-refresh`
  - `make pairing-refresh-race`
- Red（意図された失敗）:
  - `make pairing-race-negative`（非アトミックな begin/commit による上限レース）
  - `make pairing-idempotency-negative`
  - `make pairing-refresh-negative`
  - `make pairing-refresh-race-negative`

<div id="ingress-trace-correlation-idempotency">
  ### Ingress トレース相関 / 冪等性
</div>

**主張:** 取り込み処理は、ファンアウト全体でトレース相関を保持し、プロバイダーによるリトライがあっても冪等でなければならない。

ここでの意味:

- 1つの外部イベントが複数の内部メッセージになる場合でも、すべての要素が同じトレース/イベント識別子を保持する。
- リトライによって二重処理が発生しない。
- プロバイダーのイベント ID が欠落している場合、重複排除処理は安全なキー（例: トレース ID）へのフォールバックを行い、別個のイベントを取りこぼさないようにする。

- Green:
  - `make ingress-trace`
  - `make ingress-trace2`
  - `make ingress-idempotency`
  - `make ingress-dedupe-fallback`
- Red (expected):
  - `make ingress-trace-negative`
  - `make ingress-trace2-negative`
  - `make ingress-idempotency-negative`
  - `make ingress-dedupe-fallback-negative`

<div id="routing-dmscope-precedence-identitylinks">
  ### Routing dmScope 優先順位 + identityLinks
</div>

**主張:** routing はデフォルトで DM セッションを分離したまま維持し、チャネルの優先順位と identity links によって明示的に設定された場合にのみ、セッションをまとめて扱わなければならない。

具体的には次のとおりです:

- チャネル固有の dmScope オーバーライドは、グローバルなデフォルトよりも優先されなければならない。
- identityLinks は、無関係なピア間ではなく、明示的にリンクされたグループ内でのみセッションを統合すべきである。

- 緑:
  - `make routing-precedence`
  - `make routing-identitylinks`
- 赤（想定どおり）:
  - `make routing-precedence-negative`
  - `make routing-identitylinks-negative`