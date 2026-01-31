---
title: メモリ
summary: "調査メモ: Clawd ワークスペース向けオフラインメモリシステム（Markdown を唯一の信頼できる情報源とし、そこから派生させたインデックスを利用）"
read_when:
  - 日次の Markdown ログを超えるワークスペースメモリ（~/.openclaw/workspace）を設計しているとき
  - スタンドアロンの CLI にするか、OpenClaw と深く統合するかを検討しているとき
  - オフラインでのリコール／リフレクション機能（retain/recall/reflect）を追加するとき
---

<div id="workspace-memory-v2-offline-research-notes">
  # Workspace メモリ v2（オフライン）：リサーチノート
</div>

対象は、Clawd-style のワークスペース（`agents.defaults.workspace`、デフォルトは `~/.openclaw/workspace`）で、「メモリ」が 1 日ごとの Markdown ファイル（`memory/YYYY-MM-DD.md`）と、少数の安定したファイル（例: `memory.md`、`SOUL.md`）として保存されているもの。

このドキュメントでは、Markdown を検証可能な唯一の信頼できる情報源として保持しつつ、派生インデックスを用いて **構造化されたリコール機能**（検索、エンティティ要約、信頼度更新）を追加する、**オフライン・ファースト**なメモリアーキテクチャを提案する。

<div id="why-change">
  ## なぜ変更するのか？
</div>

現在のセットアップ（1日1ファイル）は、次の用途には非常に優れています:

- 「追記専用」の日誌記録
- 人手による編集
- git による永続性 + 監査可能性
- 手軽なキャプチャ（「とりあえず書き留める」）

一方で、次のような用途には弱いです:

- 高い再現率での検索（「X について何を決めたっけ？」「前回 Y を試したときはどうだった？」）
- 多数のファイルを読み返さずに得られるエンティティ中心の回答（「Alice / The Castle / warelay について教えて」）
- 意見・嗜好の一貫性の把握（および、それが変化したときの根拠）
- 時間に関する問い（「2025 年 11 月の時点では何が成り立っていたか？」）や競合解消

<div id="design-goals">
  ## 設計目標
</div>

- **オフライン**: ネットワーク接続なしで動作し、laptop/Castle 上で実行可能で、クラウドに依存しないこと。
- **説明可能**: 取得された項目は（ファイル＋位置）に帰属可能であり、推論処理とは分離できること。
- **手軽さ**: 日々のログは Markdown のまま記録でき、重いスキーマ設計を必要としないこと。
- **インクリメンタル**: v1 は FTS のみでも有用であり、セマンティック/ベクター検索やグラフは任意のアップグレードとすること。
- **エージェントフレンドリー**: エージェントにとって扱いやすく、「トークン制限内での想起（リコール）」を容易にすること（小さな事実の束を返す）。

<div id="north-star-model-hindsight-letta">
  ## ノーススター・モデル（Hindsight × Letta）
</div>

ブレンドする 2 つの要素:

1) **Letta/MemGPT スタイルの制御ループ**

- 常にコンテキスト内に保持する小さな「コア」（ペルソナ + 主要なユーザー情報）
- それ以外はすべてコンテキスト外に出し、ツール経由で取得する
- メモリ書き込みは明示的なツール呼び出し（append/replace/insert）として行い、永続化したうえで、次のターンで再度コンテキストに注入する

2) **Hindsight スタイルのメモリ基盤**

- 観測されたこと vs 信じていること vs 要約されたことを切り分ける
- retain / recall / reflect をサポートする
- 証拠によって変化しうる、確信度付きの見解を持てる
- エンティティ認識型の検索 + 時系列クエリ（完全なナレッジグラフがなくても）

<div id="proposed-architecture-markdown-source-of-truth-derived-index">
  ## 提案アーキテクチャ（Markdown をソース・オブ・トゥルースとする + 派生インデックス）
</div>

<div id="canonical-store-git-friendly">
  ### 正準ストア（git に適した形式）
</div>

`~/.openclaw/workspace` を、人間が読める正準のメモリとして維持します。

ワークスペース構成の推奨レイアウト:

```
~/.openclaw/workspace/
  memory.md                    # 小規模: 永続的な事実 + 設定 (コア的なもの)
  memory/
    YYYY-MM-DD.md              # 日次ログ (追記; ナラティブ形式)
  bank/                        # "型付き" メモリページ (安定、レビュー可能)
    world.md                   # 世界に関する客観的事実
    experience.md              # エージェントが実行したこと (一人称視点)
    opinions.md                # 主観的な設定/判断 + 確信度 + 証拠へのポインタ
    entities/
      Peter.md
      The-Castle.md
      warelay.md
      ...
```

メモ:

* **Daily log はあくまで Daily log のまま**。JSON に変換する必要はない。
* `bank/` ファイルは **キュレーション済み** で、reflection ジョブによって生成されるが、手作業で編集することもできる。
* `memory.md` は「小さく + コア寄り」のまま維持する。Clawd にセッションごとに見せたい内容だけを置いておく。


<div id="derived-store-machine-recall">
  ### 派生ストア（マシンリコール）
</div>

ワークスペース配下に派生インデックスを追加する（git で追跡対象にする必要はない）：

```
~/.openclaw/workspace/.memory/index.sqlite
```

これを次で支える：

* facts、entity links、opinion メタデータ用の SQLite スキーマ
* 語彙ベース検索用の SQLite **FTS5**（高速・小型・オフライン）
* セマンティック検索用の任意の埋め込みテーブル（こちらもオフライン）

このインデックスは常に **Markdown から再構築可能**です。


<div id="retain-recall-reflect-operational-loop">
  ## 保持 / 想起 / 振り返り（運用ループ）
</div>

<div id="retain-normalize-daily-logs-into-facts">
  ### Retain: 日次ログを「ファクト」に正規化する
</div>

ここで重要になる Hindsight の主要な示唆: 小さな断片ではなく、**ストーリーとして成立した自己完結のファクト**として保存する。

`memory/YYYY-MM-DD.md` に関する実務的なルール:

* 1日の終わり（またはその途中）に、以下を満たす 2〜5 個の箇条書きを含む `## Retain` セクションを追加する:
  * ストーリーとして成立していること（ターンをまたぐコンテキストが保持されている）
  * 自己完結していること（単体で読んでも後から意味がわかる）
  * タイプとエンティティの言及でタグ付けされていること

例:

```
## 保持
- W @Peter: 現在マラケシュに滞在中（2025年11月27日〜12月1日）、アンディの誕生日のため。
- B @warelay: connection.updateハンドラーをtry/catchでラップすることでBaileys WSクラッシュを修正しました（memory/2025-11-27.mdを参照）。
- O(c=0.95) @Peter: WhatsAppでは簡潔な返信（1500文字未満）を好む。長いコンテンツはファイルに入れる。
```

最小限のパース:

* 型プレフィックス: `W` (world / 世界), `B` (experience/biographical / 経験・経歴), `O` (opinion / 意見), `S` (observation/summary / 観察・要約; 通常は生成される)
* エンティティ: `@Peter`, `@warelay` など（スラッグは `bank/entities/*.md` にマッピングされる）
* 意見の信頼度: 任意で `O(c=0.0..1.0)`

著者にこれを考えさせたくない場合は、reflect ジョブがログの残りからこれらの箇条書き項目を推論できるが、明示的な `## Retain` セクションを設けるのが、もっとも簡単な「品質を制御する手段」になる。


<div id="recall-queries-over-the-derived-index">
  ### Recall: 派生インデックスに対するクエリ
</div>

Recall は以下をサポートする必要がある:

- **レキシカル**: “特定の用語 / 名前 / コマンドを探す” (FTS5)
- **エンティティ**: “X について教えて” (エンティティページ + エンティティに紐づくファクト)
- **時間的**: “11月27日前後に何が起きた？” / “先週以降で何があった？”
- **意見**: “Peter は何を好む？” (確信度 + 根拠つき)

戻り値のフォーマットはエージェントにとって扱いやすく、かつ出典を引用できる形にすること:

- `kind` (`world|experience|opinion|observation`)
- `timestamp` (ソースとなる日付、もしくは存在する場合は抽出された時間範囲)
- `entities` (`["Peter","warelay"]`)
- `content` (文章としての事実内容)
- `source` (`memory/2025-11-27.md#L12` など)

<div id="reflect-produce-stable-pages-update-beliefs">
  ### Reflect: 安定したページを生成し、信念を更新する
</div>

Reflection はスケジュールされたジョブ（毎日、またはハートビート `ultrathink`）であり、次を行います：

- 最近の事実（エンティティ要約）から `bank/entities/*.md` を更新する
- 強化 / 反証に基づいて `bank/opinions.md` の確信度を更新する
- 必要に応じて `memory.md`（「コア寄り」の永続的な事実）への編集案を提案する

オピニオンの進化（シンプルで説明可能）:

- 各オピニオンは以下を持つ:
  - statement
  - 確信度 `c ∈ [0,1]`
  - last_updated
  - 証拠リンク（支持 + 反証となる fact ID）
- 新しい事実が追加されたとき:
  - エンティティの重なり + 類似度によって候補オピニオンを見つける（まず FTS、あとで埋め込み）
  - 確信度を小さな差分で更新する; 大きなジャンプには強い反証 + 繰り返し得られる証拠が必要

<div id="cli-integration-standalone-vs-deep-integration">
  ## CLI 統合: スタンドアロン vs 深い統合
</div>

推奨: **OpenClaw への深い統合** を行いつつ、分離可能なコアライブラリは保持しておくこと。

<div id="why-integrate-into-openclaw">
  ### なぜ OpenClaw に統合するのか？
</div>

- OpenClaw はすでに次の情報を持っている:
  - ワークスペースのパス (`agents.defaults.workspace`)
  - セッションモデルとハートビート
  - ログおよびトラブルシューティングのパターン
- エージェント自身にツールを直接呼び出させたい:
  - `openclaw memory recall "…" --k 25 --since 30d`
  - `openclaw memory reflect --since 7d`

<div id="why-still-split-a-library">
  ### それでもライブラリとして分割しておく理由
</div>

- Gateway/ランタイムなしでもメモリロジックをテスト可能に保つため
- 他のコンテキスト（ローカルスクリプト、将来のデスクトップアプリなど）から再利用できるようにするため

構成:
メモリ関連ツールは小さな CLI＋ライブラリ層として設計されているが、現時点ではあくまで探索的な試みである。

<div id="s-collide-suco-when-to-use-it-research">
  ## “S-Collide” / SuCo: いつ使うべきか（リサーチ）
</div>

“S-Collide” が **SuCo (Subspace Collision)** を指す場合: これはサブスペース内の学習済み／構造化されたコリジョンを利用して、リコール（再現率）とレイテンシのトレードオフを狙う近似最近傍 (ANN) 検索手法（論文: arXiv 2411.14754, 2024）を意味する。

`~/.openclaw/workspace` 向けの実務的な判断基準:

- **最初から** SuCo を使い始めないこと。
- まずは SQLite FTS ＋（オプションで）シンプルな埋め込みを使う。これだけで UX 上のメリットの大半はすぐに得られる。
- SuCo / HNSW / ScaNN クラスのソリューションは、次のような状況になってから検討する:
  - コーパスが大きい（数万〜数十万チャンク）
  - 埋め込みの総当たり検索が遅くなりすぎる
  - 語彙ベース検索がボトルネックになって、リコール品質が有意に制限されている

オフライン運用向きの代替案（下に行くほど複雑）:

- SQLite FTS5 + メタデータフィルタ（ML 不要）
- 埋め込み + 総当たり検索（チャンク数が少なければ意外なほど長く通用する）
- HNSW インデックス（一般的で堅牢；ライブラリバインディングが必要）
- SuCo（リサーチグレード; 組み込める堅実な実装があるなら有力候補）

未解決の問い:

- あなたのマシン（ノート PC + デスクトップ）上で、「パーソナルアシスタントのメモリ」に最適なオフライン埋め込みモデルは何か？
  - すでに Ollama があるなら: ローカルモデルで埋め込みを行う。そうでない場合は、ツールチェーンの一部として小さな埋め込みモデルをバンドルする。

<div id="smallest-useful-pilot">
  ## 最小限の有用なパイロット
</div>

最小限でも実用的なバージョンにしたい場合は、次を行います。

- `bank/` エンティティページを追加し、日次ログに `## Retain` セクションを追加する。
- SQLite FTS を使って、出典情報（パス＋行番号）付きで検索できるようにする。
- 検索精度やスケールの要件から必要になる場合にのみ、埋め込み（embeddings）を追加する。

<div id="references">
  ## 参考文献
</div>

- Letta / MemGPT の概念：「core memory blocks（コアメモリブロック）」＋「archival memory（アーカイブメモリ）」＋ツール駆動の自己編集メモリ。
- Hindsight Technical Report：「retain / recall / reflect（保持／想起／内省）」、4ネットワーク構成のメモリアーキテクチャ、ナラティブからの事実抽出、意見の確信度の推移。
- SuCo: arXiv 2411.14754 (2024)：「Subspace Collision（部分空間衝突）」に基づく近似最近傍探索。