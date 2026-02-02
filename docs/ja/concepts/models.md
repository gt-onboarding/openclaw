---
title: モデル
summary: "Models CLI：list、set、aliases、fallbacks、scan、status"
read_when:
  - Models CLI（models list/set/scan/aliases/fallbacks）を追加または変更するとき
  - モデルのフォールバック動作や選択UXを変更するとき
  - モデルスキャン用のプローブ（tools/images）を更新するとき
---

<div id="models-cli">
  # Models CLI
</div>

認証プロファイルのローテーションやクールダウン、それがフォールバックとどのように関係するかについては、[/concepts/model-failover](/ja/concepts/model-failover) を参照してください。
プロバイダーの概要と例については、[/concepts/model-providers](/ja/concepts/model-providers) を参照してください。

<div id="how-model-selection-works">
  ## モデル選択の仕組み
</div>

OpenClaw はモデルを次の優先順位で選択します:

1. **Primary** モデル（`agents.defaults.model.primary` または `agents.defaults.model`）。
2. `agents.defaults.model.fallbacks` 内の **Fallbacks**（記載順）。
3. **プロバイダーの認証フェイルオーバー** は、次のモデルへ移る前にプロバイダー内部で処理されます。

関連情報:

* `agents.defaults.models` は、OpenClaw が使用できるモデル（およびそのエイリアス）の許可リスト／カタログです。
* `agents.defaults.imageModel` は、Primary モデルが画像を受け付けられない場合に **のみ** 使用されます。
* エージェントごとのデフォルト値は、`agents.list[].model` とそのバインディングにより `agents.defaults.model` をオーバーライドできます（[/concepts/multi-agent](/ja/concepts/multi-agent) を参照）。

<div id="quick-model-picks-anecdotal">
  ## クイックモデル候補（あくまで個人的な印象）
</div>

* **GLM**: コード作成やツール呼び出しがやや得意。
* **MiniMax**: 文章作成やノリ・雰囲気重視の用途に向いている。

<div id="setup-wizard-recommended">
  ## セットアップウィザード（推奨）
</div>

設定ファイルを手で編集したくない場合は、初期設定ウィザードを実行してください。

```bash
openclaw onboard
```

一般的なプロバイダー向けにモデルと認証をセットアップできます。これには **OpenAI Code (Codex) サブスクリプション**（OAuth）や **Anthropic**（API キーの利用を推奨。`claude
setup-token` にも対応）などが含まれます。

<div id="config-keys-overview">
  ## 設定キー（概要）
</div>

* `agents.defaults.model.primary` と `agents.defaults.model.fallbacks`
* `agents.defaults.imageModel.primary` と `agents.defaults.imageModel.fallbacks`
* `agents.defaults.models`（許可リスト + エイリアス + プロバイダー用パラメータ）
* `models.providers`（`models.json` に記述するカスタムプロバイダー）

モデル参照名は小文字に正規化されます。`z.ai/*` のようなプロバイダーエイリアスは
`zai/*` に正規化されます。

プロバイダーの設定例（OpenCode Zen を含む）は
[/gateway/configuration](/ja/gateway/configuration#opencode-zen-multi-model-proxy)
にあります。

<div id="model-is-not-allowed-and-why-replies-stop">
  ## 「Model is not allowed」（および応答が止まる理由）
</div>

`agents.defaults.models` が設定されている場合、それが `/model` と
セッションでのモデル指定（オーバーライド）に対する**許可リスト**として機能します。ユーザーがその許可リストに含まれていないモデルを選択すると、
OpenClaw は次のように返します：

```
モデル "provider/model" は許可されていません。利用可能なモデルを一覧表示するには /model を使用してください。
```

これは通常の返信が生成される**前に**発生するため、メッセージが「反応しなかった」ように感じられることがあります。対処方法はいずれかです:

* 対象のモデルを `agents.defaults.models` に追加する
* 許可リストをクリアする（`agents.defaults.models` を削除する）
* `/model list` からモデルを選択する

許可リストの設定例:

```json5
{
  agent: {
    model: { primary: "anthropic/claude-sonnet-4-5" },
    models: {
      "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
      "anthropic/claude-opus-4-5": { alias: "Opus" }
    }
  }
}
```

<div id="switching-models-in-chat-model">
  ## チャット中のモデル切り替え（`/model`）
</div>

現在のセッションを再起動することなくモデルを切り替えることができます。

```
/model
/model list
/model 3
/model openai/gpt-5.2
/model status
```

注意:

* `/model`（および `/model list`）は、コンパクトな番号付きピッカー（モデルファミリー + 利用可能なプロバイダー）です。
* `/model <#>` は、そのピッカーからの選択を行います。
* `/model status` は詳細ビューです（認証候補と、設定されている場合はプロバイダーエンドポイントの `baseUrl` と `api` モード）。
* モデル参照は、**最初の** `/` で分割してパースされます。`/model <ref>` を入力する際は `provider/model` を使用してください。
* モデル ID 自体に `/` が含まれている場合（OpenRouter スタイル）、プロバイダープレフィックスを含める必要があります（例: `/model openrouter/moonshotai/kimi-k2`）。
* プロバイダーを省略した場合、OpenClaw は入力をエイリアス、または**デフォルトプロバイダー**向けのモデルとして扱います（モデル ID に `/` が含まれていない場合のみ動作します）。

コマンドの挙動と設定の詳細: [Slash commands](/ja/tools/slash-commands)。

<div id="cli-commands">
  ## CLI コマンド
</div>

```bash
openclaw models list
openclaw models status
openclaw models set <provider/model>
openclaw models set-image <provider/model>

openclaw models aliases list
openclaw models aliases add <alias> <provider/model>
openclaw models aliases remove <alias>

openclaw models fallbacks list
openclaw models fallbacks add <provider/model>
openclaw models fallbacks remove <provider/model>
openclaw models fallbacks clear

openclaw models image-fallbacks list
openclaw models image-fallbacks add <provider/model>
openclaw models image-fallbacks remove <provider/model>
openclaw models image-fallbacks clear
```

`openclaw models`（サブコマンドなし）は、`models status` を実行するためのショートカットです。

<div id="models-list">
  ### `models list`
</div>

デフォルトでは、設定済みモデルを表示します。主なフラグ:

* `--all`: モデルカタログ全体を表示
* `--local`: ローカルプロバイダーのみ
* `--provider <name>`: 指定したプロバイダーで絞り込み
* `--plain`: 1 行につき 1 モデル
* `--json`: マシン判読可能な出力

<div id="models-status">
  ### `models status`
</div>

解決されたプライマリモデル、フォールバック、画像モデル、および設定済みプロバイダーの認証概要を表示します。さらに、auth ストア内のプロファイルについて OAuth の有効期限ステータスも表示します（デフォルトでは、有効期限まで 24 時間以内になると警告します）。`--plain` は解決されたプライマリモデルのみを出力します。
OAuth ステータスは常に表示され（`--json` 出力にも含まれます）、設定済みプロバイダーに認証情報が存在しない場合は、`models status` は **Missing auth** セクションを表示します。
JSON には `auth.oauth`（警告の猶予期間とプロファイル）および `auth.providers`（プロバイダーごとの有効な認証）が含まれます。
自動化用途では `--check` を使用します（認証情報の欠如/期限切れ時は終了コード `1`、まもなく期限切れの場合は `2` を返します）。

推奨される Anthropic 認証は、Claude Code CLI の setup-token を使ったセットアップです（どこでも実行でき、必要に応じて Gateway ホスト側にトークンを貼り付けます）:

```bash
claude setup-token
openclaw models status
```

<div id="scanning-openrouter-free-models">
  ## スキャン（OpenRouter 無料モデル）
</div>

`openclaw models scan` は OpenRouter の **無料モデルカタログ**を走査し、
オプションでツールおよび画像対応をプローブできます。

主なフラグ:

* `--no-probe`: ライブプローブをスキップする（メタデータのみ）
* `--min-params <b>`: 最小パラメータ数（10 億単位）
* `--max-age-days <days>`: 古いモデルをスキップする
* `--provider <name>`: プロバイダーのプレフィックスによるフィルター
* `--max-candidates <n>`: フォールバック候補リストの最大数
* `--set-default`: 最初に選択されたモデルを `agents.defaults.model.primary` に設定する
* `--set-image`: 最初に選択された画像モデルを `agents.defaults.imageModel.primary` に設定する

プローブには OpenRouter API キー（認証プロファイルまたは
`OPENROUTER_API_KEY`）が必要です。キーがない場合は、`--no-probe` を使って
候補の一覧のみを取得してください。

スキャン結果は次の優先順位でランク付けされます:

1. 画像対応
2. ツールのレイテンシ
3. コンテキストサイズ
4. パラメータ数

入力

* OpenRouter の `/models` リスト（フィルター `:free`）
* 認証プロファイルまたは `OPENROUTER_API_KEY` からの OpenRouter API キーが必要（[/environment](/ja/environment) を参照）
* 任意のフィルター: `--max-age-days`, `--min-params`, `--provider`, `--max-candidates`
* プローブ制御: `--timeout`, `--concurrency`

TTY 上で実行すると、フォールバックを対話的に選択できます。非対話モードでは
`--yes` を渡してデフォルトをそのまま採用します。

<div id="models-registry-modelsjson">
  ## モデルレジストリ (`models.json`)
</div>

`models.providers` 内のカスタムプロバイダーは、エージェントディレクトリ（デフォルト: `~/.openclaw/agents/<agentId>/models.json`）配下の `models.json` に書き込まれます。`models.mode` が `replace` に設定されていない限り、このファイルはデフォルトで他の定義とマージされます。