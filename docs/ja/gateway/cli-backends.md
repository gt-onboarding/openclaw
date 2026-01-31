---
title: CLI バックエンド
summary: "CLI バックエンド: ローカル AI CLI を使ったテキスト専用フォールバック"
read_when:
  - API プロバイダーが失敗したときの信頼できるフォールバックが必要な場合
  - Claude Code CLI やその他のローカル AI CLI を実行しており、それらを再利用したい場合
  - セッションと画像には対応しつつ、テキストのみ・ツール未使用のパスが必要な場合
---

<div id="cli-backends-fallback-runtime">
  # CLI backends (fallback runtime)
</div>

OpenClaw は、API プロバイダーがダウンしているときや
レート制限されているとき、一時的に不安定な挙動をしているときに、**ローカル AI CLI** を **テキスト専用のフォールバック** として実行できます。これは意図的に保守的な設計です:

- **ツールは無効** です（ツール呼び出しなし）。
- **テキスト入力 → テキスト出力**（信頼性が高い）。
- **セッションに対応**（追加入力のターンでも一貫性が保たれる）。
- CLI が画像パスを受け付ける場合は、**画像をパススルー** できます。

これはプライマリな経路ではなく、**安全装置（セーフティネット）** として設計されています。外部 API に依存せず、「常に動作する」テキスト応答が欲しい場合に使用してください。

<div id="beginner-friendly-quick-start">
  ## 初心者向けクイックスタート
</div>

Claude Code CLI は **設定なし** でもそのまま使用できます（OpenClaw には組み込みのデフォルト設定が用意されています）:

```bash
openclaw agent --message "hi" --model claude-cli/opus-4.5
```

Codex CLI は、追加の設定なしでそのまま利用できます。

```bash
openclaw agent --message "hi" --model codex-cli/gpt-5.2-codex
```

Gateway が launchd/systemd 配下で動作していて PATH が最小限にしか通っていない場合は、コマンドのパスだけを追加します：

```json5
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude"
        }
      }
    }
  }
}
```

以上です。CLI 以外には、API キーも追加の認証設定も不要です。


<div id="using-it-as-a-fallback">
  ## フォールバックとして使用する
</div>

プライマリモデルが失敗した場合にのみ実行されるように、CLI バックエンドをフォールバックリストに追加します。

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-opus-4-5",
        fallbacks: [
          "claude-cli/opus-4.5"
        ]
      },
      models: {
        "anthropic/claude-opus-4-5": { alias: "Opus" },
        "claude-cli/opus-4.5": {}
      }
    }
  }
}
```

注意:

* `agents.defaults.models`（許可リスト）を使用する場合は、`claude-cli/...` を必ず含めてください。
* プライマリプロバイダーが利用できなくなった場合（認証、レート制限、タイムアウトなど）、OpenClaw は
  次に CLI バックエンドを試行します。


<div id="configuration-overview">
  ## 構成の概要
</div>

すべての CLI バックエンドは次のパス配下に存在します：

```
agents.defaults.cliBackends
```

各エントリは **プロバイダー ID**（例: `claude-cli`、`my-cli`）をキーとして持ちます。
このプロバイダー ID が、モデル参照の左側になります。

```
<provider>/<model>
```


<div id="example-configuration">
  ### 設定例
</div>

```json5
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude"
        },
        "my-cli": {
          command: "my-cli",
          args: ["--json"],
          output: "json",
          input: "arg",
          modelArg: "--model",
          modelAliases: {
            "claude-opus-4-5": "opus",
            "claude-sonnet-4-5": "sonnet"
          },
          sessionArg: "--session",
          sessionMode: "existing",
          sessionIdFields: ["session_id", "conversation_id"],
          systemPromptArg: "--system",
          systemPromptWhen: "first",
          imageArg: "--image",
          imageMode: "repeat",
          serialize: true
        }
      }
    }
  }
}
```


<div id="how-it-works">
  ## 仕組み
</div>

1) プロバイダーのプレフィックス（`claude-cli/...`）に基づいて**バックエンドを選択**します。
2) 同じ OpenClaw のプロンプトとワークスペースのコンテキストを使って**システムプロンプトを構築**します。
3) 履歴が一貫して保持されるように、（サポートされていれば）セッション ID を付与して**CLI を実行**します。
4) **出力をパース**し（JSON またはプレーンテキスト）、最終的なテキストを返します。
5) フォローアップで同じ CLI セッションを再利用できるように、バックエンドごとに**セッション ID を永続化**します。

<div id="sessions">
  ## セッション
</div>

- CLI がセッションをサポートしている場合は、`sessionArg`（例: `--session-id`）または
  `sessionArgs`（プレースホルダー `{sessionId}`。複数のフラグに ID を挿入する必要がある場合）を設定します。
- CLI が、別のフラグを持つ **resume サブコマンド** を使用する場合は、
  `resumeArgs`（再開時に `args` を置き換える）と、必要に応じて `resumeOutput`
  （JSON 以外の形式で再開する場合に使用）を設定します。
- `sessionMode`:
  - `always`: 常にセッション ID を送信します（保存されている ID がない場合は新しい UUID を使用）。
  - `existing`: 以前に保存されている場合にのみセッション ID を送信します。
  - `none`: セッション ID を送信しません。

<div id="images-pass-through">
  ## 画像（パススルー）
</div>

CLI が画像パスを受け付ける場合は、`imageArg` を設定します。

```json5
imageArg: "--image",
imageMode: "repeat"
```

OpenClaw は base64 画像を一時ファイルとして書き出します。`imageArg` が設定されている場合、
それらのパスは CLI の引数として渡されます。`imageArg` が指定されていない場合、OpenClaw はファイルパスを
プロンプトに付加します（パスインジェクション）。これは、プレーンなパスからローカルファイルを
自動的にロードする CLI（Claude Code CLI の挙動）には十分です。


<div id="inputs-outputs">
  ## 入力 / 出力
</div>

- `output: "json"`（デフォルト）は JSON を解析し、テキストとセッション ID を抽出します。
- `output: "jsonl"` は JSONL ストリーム（Codex CLI の `--json`）を解析し、
  最後のエージェントからのメッセージと、存在する場合は `thread_id` を抽出します。
- `output: "text"` は stdout を最終的なレスポンスとして扱います。

入力モード:

- `input: "arg"`（デフォルト）はプロンプトを最後の CLI 引数として渡します。
- `input: "stdin"` はプロンプトを stdin 経由で送信します。
- プロンプトが非常に長く、`maxPromptArgChars` が設定されている場合は stdin が使用されます。

<div id="defaults-built-in">
  ## デフォルト（組み込み）
</div>

OpenClaw には `claude-cli` 用のデフォルト設定が用意されています:

- `command: "claude"`
- `args: ["-p", "--output-format", "json", "--dangerously-skip-permissions"]`
- `resumeArgs: ["-p", "--output-format", "json", "--dangerously-skip-permissions", "--resume", "{sessionId}"]`
- `modelArg: "--model"`
- `systemPromptArg: "--append-system-prompt"`
- `sessionArg: "--session-id"`
- `systemPromptWhen: "first"`
- `sessionMode: "always"`

OpenClaw には `codex-cli` 用のデフォルト設定も用意されています:

- `command: "codex"`
- `args: ["exec","--json","--color","never","--sandbox","read-only","--skip-git-repo-check"]`
- `resumeArgs: ["exec","resume","{sessionId}","--color","never","--sandbox","read-only","--skip-git-repo-check"]`
- `output: "jsonl"`
- `resumeOutput: "text"`
- `modelArg: "--model"`
- `imageArg: "--image"`
- `sessionMode: "existing"`

必要な場合のみ上書きしてください（よくあるのは `command` に絶対パスを指定するケースです）。

<div id="limitations">
  ## 制約事項
</div>

- **OpenClaw ツールは利用不可**（CLI バックエンドはツール呼び出しを一切受け付けません）。一部の CLI は独自のエージェント用ツールを実行する場合があります。
- **ストリーミングなし**（CLI の出力は一度バッファリングされてからまとめて返されます）。
- **構造化された出力** は CLI 側の JSON フォーマットに依存します。
- **Codex CLI セッション** はテキスト出力（JSONL ではない）経由で再開されます。このため、初回の `--json` 実行時ほど構造化されません。OpenClaw のセッションは引き続き通常どおり動作します。

<div id="troubleshooting">
  ## トラブルシューティング
</div>

- **CLI が見つからない**: `command` をフルパスに設定してください。
- **モデル名が誤っている**: `modelAliases` を使って `provider/model` を CLI モデルにマッピングしてください。
- **セッションが継続しない**: `sessionArg` が設定されていて、`sessionMode` が
  `none` になっていないことを確認してください（Codex CLI は現在、JSON 出力では再開できません）。
- **画像が無視される**: `imageArg` を設定し、CLI がファイルパスをサポートしていることを確認してください。