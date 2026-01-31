---
title: エージェント
summary: "エージェント実行環境（組み込み pi-mono）、ワークスペース・コントラクト、およびセッションのブートストラップ"
read_when:
  - エージェント実行環境、ワークスペースのブートストラップ、またはセッションの動作を変更するとき
---

<div id="agent-runtime">
  # エージェントランタイム 🤖
</div>

OpenClaw は、**pi-mono** を基盤とした単一の組み込みのエージェントランタイムを実行します。

<div id="workspace-required">
  ## ワークスペース（必須）
</div>

OpenClaw は単一のエージェント用ワークスペースディレクトリ（`agents.defaults.workspace`）を、ツールとコンテキストのためのエージェントの**唯一の**作業ディレクトリ（`cwd`）として使用します。

推奨: `openclaw setup` を使用して、存在しない場合は `~/.openclaw/openclaw.json` を作成し、ワークスペース用のファイルを初期化してください。

ワークスペース全体のレイアウトとバックアップに関するガイド: [Agent workspace](/ja/concepts/agent-workspace)

`agents.defaults.sandbox` が有効な場合、メイン以外のセッションは
`agents.defaults.sandbox.workspaceRoot` 配下にあるセッションごとのワークスペースでこれを上書き（オーバーライド）できます（[Gateway configuration](/ja/gateway/configuration) を参照）。

<div id="bootstrap-files-injected">
  ## ブートストラップファイル（インジェクトされるもの）
</div>

`agents.defaults.workspace` の中に、OpenClaw は次のユーザー編集可能ファイルが存在することを想定しています:

* `AGENTS.md` — 運用手順 + 「メモリ」
* `SOUL.md` — ペルソナ、境界、トーン
* `TOOLS.md` — ユーザー管理のツールメモ（例: `imsg`, `sag`, 各種ルール・慣習）
* `BOOTSTRAP.md` — 初回起動時に一度だけ実行する儀式（完了後に削除される）
* `IDENTITY.md` — エージェントの名前 / 雰囲気 / 絵文字
* `USER.md` — ユーザープロファイル + 好みの呼びかけ方

新しいセッションの最初のターンで、OpenClaw はこれらのファイルの内容をエージェントコンテキストに直接インジェクトします。

空のファイルはスキップされます。大きなファイルは、プロンプトを軽量に保つために、マーカー付きでトリミング・切り詰められます（全文はファイル自体を参照してください）。

ファイルが存在しない場合、OpenClaw は 1 行だけの「missing file」マーカーをインジェクトします（また、`openclaw setup` によって安全なデフォルトテンプレートが作成されます）。

`BOOTSTRAP.md` は、**まったく新しいワークスペース**（他のブートストラップファイルが存在しない状態）の場合にのみ作成されます。儀式を完了した後にこれを削除した場合、後続の再起動時に再作成されることはありません。

ブートストラップファイルの作成を完全に無効化したい場合（事前に内容を用意したワークスペース向け）は、次の設定を行ってください:

```json5
{ agent: { skipBootstrap: true } }
```

<div id="built-in-tools">
  ## 組み込みツール
</div>

コアツール（read/exec/edit/write および関連するシステムツール）は、ツールポリシーの範囲内で常に利用可能です。`apply_patch` はオプションであり、`tools.exec.applyPatch` によって有効化が制御されます。`TOOLS.md` は、どのツールが存在するかを制御する **ものではありません**。*あなた* がそれらをどう使わせたいかを示すためのガイドラインです。

<div id="skills">
  ## スキル
</div>

OpenClaw はスキルを次の 3 つの場所から読み込みます（名前が衝突した場合はワークスペース側が優先されます）:

* バンドル済み（インストール時に同梱されているもの）
* 管理/ローカル: `~/.openclaw/skills`
* ワークスペース: `<workspace>/skills`

スキルは設定ファイルや環境変数（config/env）で制御できます（[Gateway configuration](/ja/gateway/configuration) の `skills` を参照）。

<div id="pi-mono-integration">
  ## pi-mono との統合
</div>

OpenClaw は pi-mono のコードベース（models/tools）の一部を再利用しますが、**セッション管理、ディスカバリ、およびツール接続は OpenClaw が管理します**。

* pi-coding エージェントのランタイムはありません。
* `~/.pi/agent` や `<workspace>/.pi` の設定は参照されません。

<div id="sessions">
  ## セッション
</div>

セッションの記録は、次のパスの JSONL ファイルとして保存されます:

* `~/.openclaw/agents/<agentId>/sessions/<SessionId>.jsonl`

セッション ID は固定されており、OpenClaw によって決定されます。
旧 Pi/Tau セッションフォルダは **読み込まれません**。

<div id="steering-while-streaming">
  ## ストリーミング中のステアリング
</div>

キューモードが `steer` の場合、受信メッセージは現在進行中の実行に注入されます。
キューは**各ツール呼び出しの後に**確認され、キューされたメッセージが存在する場合は、
現在のアシスタントメッセージからの残りのツール呼び出しはスキップされます（ツール結果は `"Skipped due to queued user message."` というエラーになります）。その後、次のアシスタント応答の前にキューされたユーザーメッセージが注入されます。

キューモードが `followup` または `collect` の場合、受信メッセージは現在のターンが終了するまで保持され、
その後、キューされたペイロードで新しいエージェントのターンが開始します。モードとデバウンス／キャップ動作については
[Queue](/ja/concepts/queue) を参照してください。

ブロックストリーミングは、完了したアシスタントブロックを完了し次第送信しますが、
**デフォルトではオフ** です（`agents.defaults.blockStreamingDefault: "off"`）。
境界は `agents.defaults.blockStreamingBreak`（`text_end` と `message_end` のいずれか。デフォルトは `text_end`）で調整します。
ソフトブロックのチャンク分割は `agents.defaults.blockStreamingChunk`（デフォルトは 800–1200 文字。段落区切りを優先し、その次に改行、最後に文）で制御します。
`agents.defaults.blockStreamingCoalesce` によってストリーミングされたチャンクをまとめることで、
1 行ごとのスパム的なメッセージを削減します（送信前にアイドル時間ベースでマージ）。Telegram 以外のチャネルでは、
ブロック単位の返信を有効にするために明示的に `*.blockStreaming: true` が必要です。
詳細なツールサマリーはツール開始時に出力されます（デバウンスなし）。Control UI は、
利用可能な場合にはエージェントイベント経由でツール出力をストリーミングします。
詳細は [Streaming + chunking](/ja/concepts/streaming) を参照してください。

<div id="model-refs">
  ## モデル参照
</div>

設定内のモデル参照（例: `agents.defaults.model` や `agents.defaults.models`）は、**最初に現れる** `/` で分割して解析されます。

* モデルを設定するときは `provider/model` 形式を使用します。
* モデル ID 自体に `/` が含まれている場合（OpenRouter スタイルなど）、プロバイダーのプレフィックスを含めて指定します（例: `openrouter/moonshotai/kimi-k2`）。
* プロバイダーを省略した場合、OpenClaw はその文字列をエイリアス、または**デフォルトプロバイダー**向けのモデルとして扱います（モデル ID に `/` が含まれていない場合にのみ機能します）。

<div id="configuration-minimal">
  ## 設定（最小限）
</div>

最低限、以下を設定してください：

* `agents.defaults.workspace`
* `channels.whatsapp.allowFrom`（強く推奨）

***

*次へ: [グループチャット](/ja/concepts/group-messages)* 🦞