---
title: セッションプルーニング
summary: "セッションプルーニング：ツール結果をトリミングしてコンテキスト肥大化を抑える"
read_when:
  - ツール出力による LLM のコンテキスト増大を抑えたいとき
  - agents.defaults.contextPruning をチューニングしているとき
---

<div id="session-pruning">
  # セッションのプルーニング
</div>

セッションのプルーニングは、各 LLM 呼び出しの直前に、メモリ上のコンテキストから**古いツール結果**を削除します。ディスク上のセッション履歴（`*.jsonl`）を**変更・書き換えたりはしません**。

<div id="when-it-runs">
  ## 実行タイミング
</div>

* `mode: "cache-ttl"` が有効で、そのセッションに対する最後の Anthropic 呼び出しが `ttl` よりも前のものである場合。
* そのリクエストでモデルに送信されるメッセージにのみ影響します。
* Anthropic API 呼び出し（および OpenRouter の Anthropic モデル）でのみ有効です。
* 最良の結果を得るには、`ttl` をモデルの `cacheControlTtl` に合わせてください。
* プルーニング後は TTL ウィンドウがリセットされるため、その後のリクエストも `ttl` が再度失効するまでキャッシュを維持します。

<div id="smart-defaults-anthropic">
  ## スマートなデフォルト設定（Anthropic）
</div>

* **OAuth または setup-token** プロファイル: `cache-ttl` のプルーニングを有効化し、ハートビートを `1h` に設定します。
* **API key** プロファイル: `cache-ttl` のプルーニングを有効化し、ハートビートを `30m` に設定し、Anthropic モデルのデフォルト `cacheControlTtl` を `1h` に設定します。
* これらの値のいずれかを明示的に設定した場合、OpenClaw はそれらを**上書きしません**。

<div id="what-this-improves-cost-cache-behavior">
  ## これによって改善されるもの（コスト + キャッシュ動作）
</div>

* **なぜ枝刈りするか:** Anthropic のプロンプトキャッシュは TTL の範囲内にしか適用されません。セッションが TTL を超えてアイドル状態になると、あらかじめ履歴をトリミングしていない限り、次のリクエストでフルプロンプトが再キャッシュされます。
* **何が安くなるか:** 枝刈りにより、TTL 失効後の最初のリクエストにおける **cacheWrite** のサイズが小さくなります。
* **なぜ TTL のリセットが重要か:** 一度枝刈りを行うと、キャッシュウィンドウがリセットされるため、後続のリクエストはフル履歴を再キャッシュするのではなく、新しくキャッシュされたプロンプトを再利用できます。
* **何をしないか:** 枝刈りはトークンを追加したり、コストを「二重に」したりしません。変わるのは、TTL 経過後最初のリクエストで何がキャッシュされるかだけです。

<div id="what-can-be-pruned">
  ## どのメッセージが間引き対象になるか
</div>

* `toolResult` メッセージのみ。
* User + assistant メッセージは **決して** 変更されない。
* 直近の `keepLastAssistants` 件分の assistant メッセージは保護され、そのカットオフより新しい `toolResult` メッセージも間引き対象外となる。
* カットオフを決めるのに十分な assistant メッセージが存在しない場合、間引きはスキップされる。
* **画像ブロック** を含む `toolResult` メッセージはスキップされ（トリム／クリアは一切行われない）。

<div id="context-window-estimation">
  ## コンテキストウィンドウの推定
</div>

プルーニングでは推定されたコンテキストウィンドウ（chars ≈ tokens × 4）を使用します。ウィンドウサイズは次の優先順で決定されます:

1. モデル定義の `contextWindow`（モデルレジストリ由来）。
2. `models.providers.*.models[].contextWindow` による上書き。
3. `agents.defaults.contextTokens`。
4. 既定値の `200000` トークン。

<div id="mode">
  ## モード
</div>

<div id="cache-ttl">
  ### cache-ttl
</div>

* 最後の Anthropic 呼び出しが `ttl`（デフォルトは `5m`）より古い場合にのみ、プルーニング処理が実行されます。
* 実行時の動作は、以前と同じくソフトトリム＋ハードクリアです。

<div id="soft-vs-hard-pruning">
  ## ソフトプルーニング vs ハードプルーニング
</div>

* **Soft-trim**: サイズ超過しているツール結果にのみ適用されます。
  * 先頭部分と末尾部分を残し、その間に `...` を挿入し、元のサイズを示す注記を末尾に追加します。
  * 画像ブロックを含む結果はスキップします。
* **Hard-clear**: ツール結果全体を `hardClear.placeholder` に置き換えます。

<div id="tool-selection">
  ## ツール選択
</div>

* `tools.allow` / `tools.deny` は `*` ワイルドカードをサポートします。
* `deny` 設定が優先されます。
* マッチングは大文字小文字を区別しません。
* `allow` リストが空の場合 =&gt; すべてのツールが許可されます。

<div id="interaction-with-other-limits">
  ## 他の制限との関係
</div>

* 組み込みツールは、すでに自前で出力を切り詰めます。セッションのプルーニングはその上に重ねる追加レイヤーとして機能し、長時間にわたるチャットでツール出力がモデルコンテキストに過度に蓄積するのを防ぎます。
* コンパクションは別機能です。コンパクションは要約して永続化し、プルーニングはリクエストごとに一時的に行われます。[/concepts/compaction](/ja/concepts/compaction) を参照してください。

<div id="defaults-when-enabled">
  ## デフォルト値（有効な場合）
</div>

* `ttl`: `"5m"`
* `keepLastAssistants`: `3`
* `softTrimRatio`: `0.3`
* `hardClearRatio`: `0.5`
* `minPrunableToolChars`: `50000`
* `softTrim`: `{ maxChars: 4000, headChars: 1500, tailChars: 1500 }`
* `hardClear`: `{ enabled: true, placeholder: "[古いツール結果の内容を削除しました]" }`

<div id="examples">
  ## 例
</div>

デフォルト設定（無効）:

```json5
{
  agent: {
    contextPruning: { mode: "off" }
  }
}
```

TTL を考慮したプルーニングを有効にする：

```json5
{
  agent: {
    contextPruning: { mode: "cache-ttl", ttl: "5m" }
  }
}
```

プルーニング対象を特定のツールに限定する:

```json5
{
  agent: {
    contextPruning: {
      mode: "cache-ttl",
      tools: { allow: ["exec", "read"], deny: ["*image*"] }
    }
  }
}
```

設定リファレンスについては、[Gateway の設定](/ja/gateway/configuration)を参照してください
