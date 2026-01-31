---
title: トークンの利用
summary: "OpenClaw がどのようにプロンプトコンテキストを構築し、トークン使用量とコストを報告するか"
read_when:
  - トークン使用量、コスト、またはコンテキストウィンドウについて説明するとき
  - コンテキストの増大や圧縮の動作をデバッグするとき
---

<div id="token-use-costs">
  # トークンの使用量とコスト
</div>

OpenClaw は文字数ではなく **トークン** を計測します。トークンあたりの文字数はモデルごとに異なりますが、ほとんどの OpenAI 互換モデルでは、英語テキストの場合 1 トークンあたり平均 4 文字程度です。

<div id="how-the-system-prompt-is-built">
  ## システムプロンプトの構築方法
</div>

OpenClaw は実行のたびに独自のシステムプロンプトを組み立てます。これには次の内容が含まれます:

* ツール一覧 + 簡潔な説明
* スキル一覧（メタデータのみ。指示は `read` によってオンデマンドで読み込まれます）
* 自己更新用の手順
* ワークスペース + ブートストラップ用ファイル（`AGENTS.md`、`SOUL.md`、`TOOLS.md`、`IDENTITY.md`、`USER.md`、`HEARTBEAT.md`、新規時の `BOOTSTRAP.md`）。大きなファイルは `agents.defaults.bootstrapMaxChars`（デフォルト: 20000）でトリミングされます。
* 時刻（UTC + ユーザーのタイムゾーン）
* 返信タグ + ハートビートの動作
* ランタイムメタデータ（host/OS/model/thinking）

詳細な構成については [System Prompt](/ja/concepts/system-prompt) を参照してください。

<div id="what-counts-in-the-context-window">
  ## コンテキストウィンドウに含まれるもの
</div>

モデルが受け取るすべての情報がコンテキスト制限にカウントされます:

* システムプロンプト（上記のすべてのセクション）
* 会話履歴（ユーザー＋アシスタントのメッセージ）
* ツール呼び出しとツール結果
* 添付ファイル／書き起こし（画像、音声、ファイル）
* 圧縮サマリーおよびプルーニングの成果物
* プロバイダーのラッパーやセーフティヘッダー（ユーザーからは見えませんがカウントされます）

より実務的な内訳（インジェクトされたファイル単位、ツール、スキル、システムプロンプトサイズごと）を確認するには、`/context list` または `/context detail` を使用してください。[Context](/ja/concepts/context) を参照してください。

<div id="how-to-see-current-token-usage">
  ## 現在のトークン使用量を確認する方法
</div>

チャットでは次のコマンドを使います:

* `/status` → セッションのモデル、コンテキスト使用量、
  直近のレスポンスの入出力トークン数、**推定コスト**（APIキー使用時のみ）を含む **絵文字入りステータスカード** を表示します。
* `/usage off|tokens|full` → すべての返信に **レスポンスごとの使用量フッター** を付与します。
  * セッション単位で保持されます（`responseUsage` として保存）。
  * OAuth 認証時は **コストは非表示**（トークン数のみ）になります。
* `/usage cost` → OpenClaw のセッションログからローカルのコスト概要を表示します。

その他の場所:

* **TUI/Web TUI:** `/status` と `/usage` が利用可能です。
* **CLI:** `openclaw status --usage` と `openclaw channels list` で、
  プロバイダーのクォータ期間（レスポンスごとのコストではない）を確認できます。

<div id="cost-estimation-when-shown">
  ## コスト見積もり（表示される場合）
</div>

コストは、モデル料金設定に基づいて推定されます。

```
models.providers.<provider>.models[].cost
```

これらは `input`、`output`、`cacheRead`、`cacheWrite` それぞれについての、**100万トークンあたりの USD 建て料金**です。料金情報がない場合、OpenClaw はトークン数のみを表示します。OAuth トークンについては、ドル建てのコストは表示されません。

<div id="cache-ttl-and-pruning-impact">
  ## キャッシュ TTL とプルーニングの影響
</div>

プロバイダーのプロンプトキャッシュは、キャッシュ TTL ウィンドウ内でのみ有効です。OpenClaw はオプションで **cache-ttl pruning** を実行できます。これは、キャッシュ TTL が失効したタイミングでセッションをプルーニングし、その後キャッシュウィンドウをリセットして、以降のリクエストが履歴全体を再キャッシュする代わりに、新しくキャッシュされたコンテキストを再利用できるようにするものです。これにより、セッションが TTL を越えてアイドル状態になった場合でも、キャッシュへの書き込みコストを抑えられます。

[Gateway configuration](/ja/gateway/configuration) で設定し、挙動の詳細は [Session pruning](/ja/concepts/session-pruning) を参照してください。

ハートビートにより、アイドル期間をまたいでキャッシュを **ウォーム** な状態に保てます。モデルのキャッシュ TTL が `1h` の場合、ハートビート間隔をそれより少し短く（例: `55m`）設定することで、プロンプト全体の再キャッシュを避け、キャッシュへの書き込みコストを削減できます。

Anthropic の API 料金においては、キャッシュの読み取りは入力トークンよりも大幅に安価ですが、キャッシュへの書き込みはより高い係数で課金されます。最新の料金と TTL の乗数については、Anthropic のプロンプトキャッシュ料金を参照してください:
https://docs.anthropic.com/docs/build-with-claude/prompt-caching

<div id="example-keep-1h-cache-warm-with-heartbeat">
  ### 例: ハートビートでキャッシュを1時間ウォーム状態に保つ
</div>

```yaml
agents:
  defaults:
    model:
      primary: "anthropic/claude-opus-4-5"
    models:
      "anthropic/claude-opus-4-5":
        params:
          cacheControlTtl: "1h"
    heartbeat:
      every: "55m"
```

<div id="tips-for-reducing-token-pressure">
  ## トークン負荷を減らすためのヒント
</div>

* 長いセッションを要約するには `/compact` を使用します。
* ワークフロー内で大きなツール出力は切り詰めます。
* スキルの説明は短く保ちます（スキル一覧はプロンプトに挿入されます）。
* 冗長な試行錯誤的な作業には、より小さなモデルを優先して使用します。

スキル一覧によるオーバーヘッドの正確な計算式については、[Skills](/ja/tools/skills) を参照してください。