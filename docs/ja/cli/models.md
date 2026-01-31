---
title: モデル
summary: "`openclaw models` 用 CLI リファレンス（status/list/set/scan、エイリアス、フォールバック、認証）"
read_when:
  - デフォルトモデルを変更したい、またはプロバイダーの認証状態を確認したい場合
  - 利用可能なモデル／プロバイダーをスキャンし、認証プロファイルをデバッグしたい場合
---

<div id="openclaw-models">
  # `openclaw models`
</div>

モデルの検出・スキャンと構成（デフォルトモデル、フォールバック、認証プロファイル）。

関連:

* プロバイダーとモデル: [Models](/ja/providers/models)
* プロバイダー認証のセットアップ: [Getting started](/ja/start/getting-started)

<div id="common-commands">
  ## 主なコマンド
</div>

```bash
openclaw models status
openclaw models list
openclaw models set <model-or-alias>
openclaw models scan
```

`openclaw models status` は、解決済みのデフォルト値/フォールバックに加えて、認証設定の概要を表示します。
プロバイダー使用状況のスナップショットが利用可能な場合、OAuth/トークンステータスセクションに
プロバイダー使用状況ヘッダーが含まれます。
`--probe` を追加すると、設定済みの各プロバイダープロファイルに対してライブな認証プローブを実行します。
これは実際のリクエストであり（トークンを消費し、レート制限を引き起こす可能性があります）。
`--agent <id>` を使用すると、設定済みエージェントのモデル/認証状態を確認できます。省略された場合、
このコマンドは `OPENCLAW_AGENT_DIR`/`PI_CODING_AGENT_DIR` が設定されていればそれを、なければ
設定済みのデフォルトエージェントを使用します。

Notes:

* `models set <model-or-alias>` は `provider/model` またはエイリアスを受け付けます。
* モデル参照は **最初の** `/` で分割してパースされます。モデル ID に `/`（OpenRouter 形式）が含まれる場合は、プロバイダープレフィックスを含めてください（例: `openrouter/moonshotai/kimi-k2`）。
* プロバイダーを省略した場合、OpenClaw は入力をエイリアス、または **デフォルトプロバイダー** 向けのモデルとして扱います（モデル ID に `/` が含まれない場合にのみ動作します）。

<div id="models-status">
  ### `models status`
</div>

オプション:

* `--json`
* `--plain`
* `--check` (終了コード 1=有効期限切れ/存在しない, 2=まもなく有効期限切れ)
* `--probe` (設定済み認証プロファイルへのライブプローブ)
* `--probe-provider <name>` (指定したプロバイダーのみをプローブ)
* `--probe-profile <id>` (プロファイル ID をフラグの繰り返し指定、またはカンマ区切りで指定)
* `--probe-timeout <ms>`
* `--probe-concurrency <n>`
* `--probe-max-tokens <n>`
* `--agent <id>` (設定済みエージェント ID。`OPENCLAW_AGENT_DIR`/`PI_CODING_AGENT_DIR` を上書き)

<div id="aliases-fallbacks">
  ## エイリアスとフォールバック
</div>

```bash
openclaw models aliases list
openclaw models fallbacks list
```

<div id="auth-profiles">
  ## 認証プロファイル
</div>

```bash
openclaw models auth add
openclaw models auth login --provider <id>
openclaw models auth setup-token
openclaw models auth paste-token
```

`models auth login` は、プロバイダープラグインの認可フロー（OAuth / API キー）を実行します。
どのプロバイダーがインストールされているかを確認するには、`openclaw plugins list` を使用します。

注意:

* `setup-token` は setup-token の値の入力を求めます（任意のマシンで `claude setup-token` を実行して生成します）。
* `paste-token` は、別の場所や自動化によって生成されたトークン文字列を受け付けます。
