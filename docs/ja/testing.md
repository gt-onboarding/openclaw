---
title: テスト
summary: "テストキット: unit/e2e/live テストスイート、Docker ランナー、および各テストで何を検証するか"
read_when:
  - ローカルまたは CI でテストを実行するとき
  - モデル／プロバイダーのバグに対する回帰テストを追加するとき
  - Gateway とエージェントの挙動をデバッグするとき
---

<div id="testing">
  # テスト
</div>

OpenClaw には 3 つの Vitest スイート（ユニット/統合テスト、e2e、live）と、いくつかの Docker ランナーがあります。

このドキュメントは「どのようにテストしているか」のガイドです:

* 各スイートが何をカバーし（そして意図的に *カバーしない* ものは何か）
* 一般的なワークフロー（ローカル、プッシュ前、デバッグ）でどのコマンドを実行すべきか
* live テストがどのように認証情報を検出し、モデル/プロバイダーを選択するか
* 実運用のモデル/プロバイダーの不具合に対するリグレッションテストをどのように追加するか

<div id="quick-start">
  ## クイックスタート
</div>

通常の開発フローでは次のとおりです:

* フルゲート（push 前に通過していることが望ましい）: `pnpm lint && pnpm build && pnpm test`

テストに手を入れた場合や、テスト結果により自信を持ちたい場合:

* カバレッジゲート: `pnpm test:coverage`
* E2E スイート: `pnpm test:e2e`

実際のプロバイダー／モデルをデバッグする場合（実際のクレデンシャルが必要）:

* ライブスイート（models + Gateway の tool/image プローブ）: `pnpm test:live`

ヒント: 単一の失敗ケースだけが分かればよい場合は、以下で説明する許可リスト用の環境変数でライブテストの対象を絞り込むことを推奨します。

<div id="test-suites-what-runs-where">
  ## テストスイート（どこで何が動くか）
</div>

スイートは「どれだけ現実の環境に近いか」（それに伴って不安定さやコストがどれだけ増えるか）が段階的に高くなっていくものだと考えてください。

<div id="unit-integration-default">
  ### ユニット / 統合テスト（デフォルト）
</div>

* コマンド: `pnpm test`
* 設定: `vitest.config.ts`
* 対象ファイル: `src/**/*.test.ts`
* スコープ:
  * 純粋なユニットテスト
  * プロセス内の統合テスト（Gateway の認証、ルーティング、ツール、パース、設定）
  * 既知バグに対する決定論的なリグレッションテスト
* 想定:
  * CI で実行する
  * 実際の API キーは不要
  * 高速かつ安定していること

<div id="e2e-gateway-smoke">
  ### E2E (Gateway スモークテスト)
</div>

* Command: `pnpm test:e2e`
* Config: `vitest.e2e.config.ts`
* Files: `src/**/*.e2e.test.ts`
* Scope:
  * 複数インスタンス Gateway のエンドツーエンド動作
  * WebSocket/HTTP インターフェイス、ノードのペアリング、およびより重いネットワーク処理
* Expectations:
  * CI で実行される（パイプラインで有効化されている場合）
  * 実際の API キーは不要
  * ユニットテストより関与するコンポーネントが多い（そのぶん遅くなる可能性がある）

<div id="live-real-providers-real-models">
  ### ライブ（実プロバイダー + 実モデル）
</div>

* コマンド: `pnpm test:live`
* 設定ファイル: `vitest.live.config.ts`
* 対象ファイル: `src/**/*.live.test.ts`
* デフォルト: `pnpm test:live` により **有効**（`OPENCLAW_LIVE_TEST=1` を設定）
* スコープ:
  * 「このプロバイダー / モデルは、実際のクレデンシャルで *今日* ちゃんと動作するか？」
  * プロバイダーのフォーマット変更、ツール呼び出しの癖、認証問題、レートリミットの挙動を検知する
* 想定事項:
  * 設計上、CI での安定動作は前提としていない（実ネットワーク、実プロバイダーのポリシー、クォータ、障害の影響を受ける）
  * コストが発生する / レートリミットを消費する
  * 「全部」を走らせるより、絞り込んだサブセットの実行を推奨
  * ライブ実行時には `~/.profile` を読み込んで、不足している API キーを取得する
  * Anthropic キーのローテーション: `OPENCLAW_LIVE_ANTHROPIC_KEYS="sk-...,sk-..."`（または `OPENCLAW_LIVE_ANTHROPIC_KEY=sk-...`）、もしくは複数の `ANTHROPIC_API_KEY*` 変数を設定する。テストはレートリミット発生時にリトライする

<div id="which-suite-should-i-run">
  ## どのテストスイートを実行すればよいですか？
</div>

次を目安に選んでください:

* ロジック／テストを編集している場合: `pnpm test` を実行（大きく変更した場合は `pnpm test:coverage` も）
* Gateway のネットワーキング／WS プロトコル／ペアリングを変更した場合: `pnpm test:e2e` も実行
* 「自分のボットが落ちている」／プロバイダー固有の失敗／ツール呼び出しの問題をデバッグしている場合: 対象を絞った `pnpm test:live` を実行

<div id="live-model-smoke-profile-keys">
  ## ライブ: モデルスモークテスト（プロファイルキー）
</div>

ライブテストは障害箇所を切り分けられるよう、2つのレイヤーに分かれています。

* 「Direct model」は、指定されたキーでそのプロバイダー／モデルがそもそも応答できるかどうかを確認します。
* 「Gateway smoke」は、そのモデルに対して Gateway＋エージェントのパイプライン全体（セッション、履歴、ツール、サンドボックスのポリシーなど）が正常に動作するかどうかを確認します。

<div id="layer-1-direct-model-completion-no-gateway">
  ### レイヤー 1: モデルへの直接コンプリーション（Gateway を経由しない）
</div>

* テスト: `src/agents/models.profiles.live.test.ts`
* 目的:
  * 検出されたモデルを列挙する
  * `getApiKeyForModel` を使って、認証情報を持っているモデルを選択する
  * モデルごとに小さなコンプリーションを実行する（必要に応じて、対象を絞ったリグレッションも実行する）
* 有効化方法:
  * `pnpm test:live`（または Vitest を直接実行する場合は `OPENCLAW_LIVE_TEST=1`）
* このスイートを実際に実行するには `OPENCLAW_LIVE_MODELS=modern`（または `all`。`modern` のエイリアス）を設定する。それ以外の場合は、`pnpm test:live` を Gateway のスモークテストに集中させるためスキップされる
* モデルの選択方法:
  * `OPENCLAW_LIVE_MODELS=modern` で modern 許可リストを実行（Opus/Sonnet/Haiku 4.5, GPT-5.x + Codex, Gemini 3, GLM 4.7, MiniMax M2.1, Grok 4）
  * `OPENCLAW_LIVE_MODELS=all` は modern 許可リストのエイリアス
  * もしくは `OPENCLAW_LIVE_MODELS="openai/gpt-5.2,anthropic/claude-opus-4-5,..."`（カンマ区切りの許可リスト）
* プロバイダーの選択方法:
  * `OPENCLAW_LIVE_PROVIDERS="google,google-antigravity,google-gemini-cli"`（カンマ区切りの許可リスト）
* キーの取得元:
  * デフォルト: プロファイルストアと環境変数フォールバック
  * `OPENCLAW_LIVE_REQUIRE_PROFILE_KEYS=1` を設定すると **プロファイルストアのみ** を強制する
* このテストが存在する理由:
  * 「プロバイダー API が壊れている／キーが無効」であるケースと「Gateway のエージェントパイプラインが壊れている」ケースを切り分けるため
  * 小さく独立したリグレッションを含めるため（例: OpenAI Responses/Codex Responses の reasoning replay + tool-call フロー）

<div id="layer-2-gateway-dev-agent-smoke-what-openclaw-actually-does">
  ### レイヤー 2: Gateway + dev エージェントのスモークテスト（「@openclaw」が実際に行っていること）
</div>

* テスト: `src/gateway/gateway-models.profiles.live.test.ts`
* 目的:
  * プロセス内 Gateway を起動する
  * `agent:dev:*` セッションを作成/パッチする（実行ごとにモデルを上書き）
  * モデル＋キーの組を順に試し、次をアサートする:
    * 「意味のある」レスポンス（ツール未使用）
    * 実際のツール呼び出しが動作すること（read プローブ）
    * 任意の追加ツールプローブ（exec+read プローブ）が動作すること
    * OpenAI のリグレッション用パス（ツール呼び出しのみ → 後続メッセージ）が壊れていないこと
* プローブの詳細（失敗時にすぐ説明できるように）:
  * `read` プローブ: テストがワークスペース内に nonce ファイルを書き込み、エージェントにそれを `read` して nonce をそのまま返すよう要求する。
  * `exec+read` プローブ: テストがエージェントに `exec` で一時ファイルへ nonce を書き込ませ、その後で `read` させる。
  * 画像プローブ: テストが生成した PNG（猫 + ランダムコード）を添付し、モデルから `cat <CODE>` を返すことを期待する。
  * 実装リファレンス: `src/gateway/gateway-models.profiles.live.test.ts` および `src/gateway/live-image-probe.ts`
* 有効化方法:
  * `pnpm test:live`（または Vitest を直接呼び出す場合は `OPENCLAW_LIVE_TEST=1`）
* モデルの選択方法:
  * デフォルト: モダンな許可リスト（Opus/Sonnet/Haiku 4.5、GPT-5.x + Codex、Gemini 3、GLM 4.7、MiniMax M2.1、Grok 4）
  * `OPENCLAW_LIVE_GATEWAY_MODELS=all` はモダンな許可リストのエイリアス
  * もしくは `OPENCLAW_LIVE_GATEWAY_MODELS="provider/model"`（またはカンマ区切り）を設定して絞り込む
* プロバイダーの選択方法（「OpenRouter で全部流す」状態を回避する）:
  * `OPENCLAW_LIVE_GATEWAY_PROVIDERS="google,google-antigravity,google-gemini-cli,openai,anthropic,zai,minimax"`（カンマ区切りの許可リスト）
* このライブテストではツール＋画像プローブは常に有効:
  * `read` プローブ + `exec+read` プローブ（ツール負荷テスト）
  * モデルが画像入力対応をアドバタイズしている場合に画像プローブを実行
  * フロー（ハイレベル）:
    * テストが「CAT」＋ランダムコード入りの小さな PNG を生成する（`src/gateway/live-image-probe.ts`）
    * それを `agent` 経由で `attachments: [{ mimeType: "image/png", content: "<base64>" }]` として送信する
    * Gateway が添付ファイルを `images[]` にパースする（`src/gateway/server-methods/agent.ts` + `src/gateway/chat-attachments.ts`）
    * 埋め込みエージェントがマルチモーダルのユーザーメッセージをモデルに転送する
    * アサーション: 応答に `cat` とコードの両方が含まれていること（OCR 許容範囲: 軽微な誤りは許容）

Tip: 自分のマシンで何をテストできるか（および正確な `provider/model` ID）を確認するには、次を実行する:

```bash
openclaw models list
openclaw models list --json
```

<div id="live-anthropic-setup-token-smoke">
  ## ライブ: Anthropic setup-token スモークテスト
</div>

* テスト: `src/agents/anthropic.setup-token.live.test.ts`
* 目的: Claude Code CLI の setup-token（または貼り付けた setup-token プロファイル）で Anthropic のプロンプトを完了できることを検証する。
* 有効にするには:
  * `pnpm test:live`（または Vitest を直接実行する場合は `OPENCLAW_LIVE_TEST=1`）
  * `OPENCLAW_LIVE_SETUP_TOKEN=1`
* トークンの取得元（いずれか 1 つを選択）:
  * プロファイル: `OPENCLAW_LIVE_SETUP_TOKEN_PROFILE=anthropic:setup-token-test`
  * 生のトークン値: `OPENCLAW_LIVE_SETUP_TOKEN_VALUE=sk-ant-oat01-...`
* モデルの上書き（任意）:
  * `OPENCLAW_LIVE_SETUP_TOKEN_MODEL=anthropic/claude-opus-4-5`

セットアップ例:

```bash
openclaw models auth paste-token --provider anthropic --profile-id anthropic:setup-token-test
OPENCLAW_LIVE_SETUP_TOKEN=1 OPENCLAW_LIVE_SETUP_TOKEN_PROFILE=anthropic:setup-token-test pnpm test:live src/agents/anthropic.setup-token.live.test.ts
```

<div id="live-cli-backend-smoke-claude-code-cli-or-other-local-clis">
  ## Live: CLI backend スモークテスト (Claude Code CLI などのローカル CLI)
</div>

* テスト: `src/gateway/gateway-cli-backend.live.test.ts`
* 目的: デフォルトの設定ファイルには触れずに、ローカル CLI backend を使って Gateway とエージェント間のパイプラインを検証する。
* 有効化:
  * `pnpm test:live` (または Vitest を直接呼び出す場合は `OPENCLAW_LIVE_TEST=1`)
  * `OPENCLAW_LIVE_CLI_BACKEND=1`
* デフォルト:
  * Model: `claude-cli/claude-sonnet-4-5`
  * Command: `claude`
  * Args: `["-p","--output-format","json","--dangerously-skip-permissions"]`
* オーバーライド (任意):
  * `OPENCLAW_LIVE_CLI_BACKEND_MODEL="claude-cli/claude-opus-4-5"`
  * `OPENCLAW_LIVE_CLI_BACKEND_MODEL="codex-cli/gpt-5.2-codex"`
  * `OPENCLAW_LIVE_CLI_BACKEND_COMMAND="/full/path/to/claude"`
  * `OPENCLAW_LIVE_CLI_BACKEND_ARGS='["-p","--output-format","json","--permission-mode","bypassPermissions"]'`
  * `OPENCLAW_LIVE_CLI_BACKEND_CLEAR_ENV='["ANTHROPIC_API_KEY","ANTHROPIC_API_KEY_OLD"]'`
  * `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_PROBE=1` 実際の画像添付を送信する (パスはプロンプトに注入される)。
  * `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_ARG="--image"` 画像ファイルパスを、プロンプト注入ではなく CLI 引数として渡す。
  * `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_MODE="repeat"` (または `"list"`) `IMAGE_ARG` が設定されている場合に、画像引数の渡し方を制御する。
  * `OPENCLAW_LIVE_CLI_BACKEND_RESUME_PROBE=1` 2 ターン目を送信して、再開フローを検証する。
* `OPENCLAW_LIVE_CLI_BACKEND_DISABLE_MCP_CONFIG=0` Claude Code CLI の MCP 設定を有効なまま維持する (デフォルトでは一時的な空ファイルを用いて MCP 設定を無効化する)。

Example:

```bash
OPENCLAW_LIVE_CLI_BACKEND=1 \
  OPENCLAW_LIVE_CLI_BACKEND_MODEL="claude-cli/claude-sonnet-4-5" \
  pnpm test:live src/gateway/gateway-cli-backend.live.test.ts
```

<div id="recommended-live-recipes">
  ### 推奨されるライブ用レシピ
</div>

スコープが狭く明示的な許可リストが、もっとも高速かつ安定しやすいです:

* 単一モデル、ダイレクト（Gateway を経由しない）:
  * `OPENCLAW_LIVE_MODELS="openai/gpt-5.2" pnpm test:live src/agents/models.profiles.live.test.ts`

* 単一モデル、Gateway スモークテスト:
  * `OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

* 複数プロバイダー間でのツール呼び出し:
  * `OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2,anthropic/claude-opus-4-5,google/gemini-3-flash-preview,zai/glm-4.7,minimax/minimax-m2.1" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

* Google にフォーカス（Gemini API キー + Antigravity）:
  * Gemini（API キー）: `OPENCLAW_LIVE_GATEWAY_MODELS="google/gemini-3-flash-preview" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`
  * Antigravity（OAuth）: `OPENCLAW_LIVE_GATEWAY_MODELS="google-antigravity/claude-opus-4-5-thinking,google-antigravity/gemini-3-pro-high" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

補足:

* `google/...` は Gemini API（API キー）を使用します。
* `google-antigravity/...` は Antigravity OAuth ブリッジ（Cloud Code Assist 風のエージェントエンドポイント）を使用します。
* `google-gemini-cli/...` はローカルマシン上の Gemini CLI（ローカルの `gemini` バイナリ。独立した認証 + ツール周りの独自仕様）を使用します。
* Gemini API と Gemini CLI の違い:
  * API: OpenClaw が HTTP 経由で Google のホストされた Gemini API を呼び出します（API キー / プロファイル認証）。一般的に多くのユーザーが「Gemini」と言うときは、これを指します。
  * CLI: OpenClaw がローカルの `gemini` バイナリをサブプロセスとして実行します。独自の認証を持ち、挙動が異なる場合があります（ストリーミング / ツール対応 / バージョン差異など）。

<div id="live-model-matrix-what-we-cover">
  ## ライブ: モデルマトリクス（カバー範囲）
</div>

固定の「CI 用モデルリスト」はありません（ライブはオプトイン）ですが、以下はキーを設定した開発マシンで継続的にテスト対象に含めておくことを推奨するモデルです。

<div id="modern-smoke-set-tool-calling-image">
  ### 最新スモークテストセット（ツール呼び出し + 画像）
</div>

これは今後も継続して動作することを想定している「共通モデル」の実行セットです:

* OpenAI（非 Codex）: `openai/gpt-5.2`（オプション: `openai/gpt-5.1`）
* OpenAI Codex: `openai-codex/gpt-5.2`（オプション: `openai-codex/gpt-5.2-codex`）
* Anthropic: `anthropic/claude-opus-4-5`（または `anthropic/claude-sonnet-4-5`）
* Google（Gemini API）: `google/gemini-3-pro-preview` および `google/gemini-3-flash-preview`（古い Gemini 2.x モデルは使用しないこと）
* Google（Antigravity）: `google-antigravity/claude-opus-4-5-thinking` および `google-antigravity/gemini-3-flash`
* Z.AI（GLM）: `zai/glm-4.7`
* MiniMax: `minimax/minimax-m2.1`

Gateway のスモークテスト（ツール + 画像対応）を実行します:
`OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2,openai-codex/gpt-5.2,anthropic/claude-opus-4-5,google/gemini-3-pro-preview,google/gemini-3-flash-preview,google-antigravity/claude-opus-4-5-thinking,google-antigravity/gemini-3-flash,zai/glm-4.7,minimax/minimax-m2.1" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

<div id="baseline-tool-calling-read-optional-exec">
  ### ベースライン: ツール呼び出し (`read` + 任意の `Exec`)
</div>

プロバイダー系統ごとに少なくとも 1 つ選択してください:

* OpenAI: `openai/gpt-5.2`（または `openai/gpt-5-mini`）
* Anthropic: `anthropic/claude-opus-4-5`（または `anthropic/claude-sonnet-4-5`）
* Google: `google/gemini-3-flash-preview`（または `google/gemini-3-pro-preview`）
* Z.AI (GLM): `zai/glm-4.7`
* MiniMax: `minimax/minimax-m2.1`

任意の追加カバー範囲（あると望ましい）:

* xAI: `xai/grok-4`（または利用可能な最新のもの）
* Mistral: `mistral/`…（有効化しているモデルのうち「tools」対応のものを 1 つ選択）
* Cerebras: `cerebras/`…（アクセス権がある場合）
* LM Studio: `lmstudio/`…（ローカル；ツール呼び出しは API モードに依存）

<div id="vision-image-send-attachment-multimodal-message">
  ### ビジョン: 画像送信（添付ファイル → マルチモーダルメッセージ）
</div>

`OPENCLAW_LIVE_GATEWAY_MODELS` に、少なくとも 1 つは画像対応モデル（Claude/Gemini/OpenAI の Vision 対応バリアントなど）を含め、`image probe` を実行できるようにしてください。

<div id="aggregators-alternate-gateways">
  ### アグリゲーター / 代替 Gateway
</div>

キーが有効になっている場合、以下を介したテストにも対応しています:

* OpenRouter: `openrouter/...`（数百のモデルが利用可能。`openclaw models scan` を使って、ツール＋画像対応の候補モデルを見つけてください）
* OpenCode Zen: `opencode/...`（`OPENCODE_API_KEY` / `OPENCODE_ZEN_API_KEY` による認証）

ライブマトリクスに追加できるプロバイダー（クレデンシャル/設定がある場合）:

* 組み込み: `openai`, `openai-codex`, `anthropic`, `google`, `google-vertex`, `google-antigravity`, `google-gemini-cli`, `zai`, `openrouter`, `opencode`, `xai`, `groq`, `cerebras`, `mistral`, `github-copilot`
* `models.providers`（カスタムエンドポイント）経由: `minimax`（クラウド/API）に加えて、OpenAI/Anthropic 互換プロキシ（LM Studio、vLLM、LiteLLM など）

Tip: ドキュメント内で「すべてのモデル」をハードコードしようとしないでください。正となるリストは、ローカル環境で `discoverModels(...)` が返すもの＋利用可能なキーに依存します。

<div id="credentials-never-commit">
  ## 認証情報（絶対にコミットしないこと）
</div>

ライブテストは、CLI と同様の方法で認証情報を検出します。実務上の意味は次のとおりです。

* CLI が動作しているなら、ライブテストでも同じキーが見つかるはずです。

* ライブテストが「no creds」と表示する場合は、`openclaw models list` やモデル選択をデバッグするときと同じ要領でデバッグしてください。

* プロファイルストア: `~/.openclaw/credentials/`（推奨。テストで「profile keys」と呼んでいるもの）

* 設定: `~/.openclaw/openclaw.json`（または `OPENCLAW_CONFIG_PATH`）

環境変数経由のキー（例: `~/.profile` で export しているもの）に依存したい場合は、`source ~/.profile` を実行してからローカルテストを実行するか、以下の Docker ランナーを使用してください（`~/.profile` をコンテナ内にマウントできます）。

<div id="deepgram-live-audio-transcription">
  ## Deepgram ライブ（音声文字起こし）
</div>

* テスト: `src/media-understanding/providers/deepgram/audio.live.test.ts`
* 有効化: `DEEPGRAM_API_KEY=... DEEPGRAM_LIVE_TEST=1 pnpm test:live src/media-understanding/providers/deepgram/audio.live.test.ts`

<div id="docker-runners-optional-works-in-linux-checks">
  ## Docker ランナー（任意の「Linux で動くか」チェック）
</div>

これらはこのリポジトリの Docker イメージ内で `pnpm test:live` を実行し、ローカルの設定ディレクトリとワークスペースをマウントします（マウントされていれば `~/.profile` も読み込みます）:

* 直接モデル: `pnpm test:docker:live-models`（スクリプト: `scripts/test-live-models-docker.sh`）
* Gateway + 開発用エージェント: `pnpm test:docker:live-gateway`（スクリプト: `scripts/test-live-gateway-models-docker.sh`）
* Onboarding wizard（TTY、完全なスキャフォールディング）: `pnpm test:docker:onboard`（スクリプト: `scripts/e2e/onboard-docker.sh`）
* Gateway のネットワーク（2 つのコンテナ、WS 認証 + ヘルス）: `pnpm test:docker:gateway-network`（スクリプト: `scripts/e2e/gateway-network-docker.sh`）
* プラグイン（カスタム拡張ロード + レジストリ疎通確認）: `pnpm test:docker:plugins`（スクリプト: `scripts/e2e/plugins-docker.sh`）

有用な環境変数:

* `OPENCLAW_CONFIG_DIR=...`（デフォルト: `~/.openclaw`）を `/home/node/.openclaw` にマウント
* `OPENCLAW_WORKSPACE_DIR=...`（デフォルト: `~/.openclaw/workspace`）を `/home/node/.openclaw/workspace` にマウント
* `OPENCLAW_PROFILE_FILE=...`（デフォルト: `~/.profile`）を `/home/node/.profile` にマウントし、テスト実行前に読み込み
* `OPENCLAW_LIVE_GATEWAY_MODELS=...` / `OPENCLAW_LIVE_MODELS=...` で実行対象を絞り込み
* `OPENCLAW_LIVE_REQUIRE_PROFILE_KEYS=1` を設定して、認証情報が必ずプロファイルストアから（環境変数ではなく）取得されるようにする

<div id="docs-sanity">
  ## ドキュメントの健全性チェック
</div>

ドキュメントを編集したら、`pnpm docs:list` を実行してチェックを行ってください。

<div id="offline-regression-ci-safe">
  ## オフライン回帰テスト（CI セーフ）
</div>

これは、実際のプロバイダーを使わない「実パイプライン」の回帰テストです:

* Gateway ツール呼び出し（OpenAI をモックし、実際の Gateway + エージェントループを使用）：`src/gateway/gateway.tool-calling.mock-openai.test.ts`
* Gateway ウィザード（WS `wizard.start`/`wizard.next` を用い、設定を書き込み + 認証を必須化）：`src/gateway/gateway.wizard.e2e.test.ts`

<div id="agent-reliability-evals-skills">
  ## エージェント信頼性評価（スキル）
</div>

すでに、「エージェント信頼性評価」に相当する、CI 環境で安全に実行できるテストがいくつかあります:

* 実際の Gateway + エージェントループを通したモックのツール呼び出し (`src/gateway/gateway.tool-calling.mock-openai.test.ts`)。
* セッションのワイヤリングと設定の効果を検証するエンドツーエンドのウィザードフロー (`src/gateway/gateway.wizard.e2e.test.ts`)。

スキル向けに、まだ不足しているもの（[Skills](/ja/tools/skills) を参照）:

* **意思決定:** プロンプト内にスキルが列挙されたとき、エージェントが正しいスキルを選択するか（あるいは不要なスキルを避けられるか）？
* **コンプライアンス:** エージェントは使用前に `SKILL.md` を read し、要求されている手順・引数に従うか？
* **ワークフロー契約:** ツールの実行順序、セッション履歴の引き継ぎ、サンドボックスの境界を検証するマルチターンシナリオ。

将来追加する評価も、まずは決定論的であることを優先する:

* モックのプロバイダーを用いて、ツール呼び出し + 順序、スキルファイルの read、セッションのワイヤリングをアサートするシナリオランナー。
* スキルにフォーカスした小さなシナリオスイート（使用 vs 回避、ゲーティング、プロンプトインジェクション）。
* CI で安全なスイートが整備された後にのみ有効化する、オプションのライブ評価（オプトイン・環境変数で制御可能）。

<div id="adding-regressions-guidance">
  ## 回帰テストの追加（ガイダンス）
</div>

本番環境で発見したプロバイダー／モデルの問題を修正したときは、次を行うこと：

* 可能であれば CI セーフな回帰テストを追加する（プロバイダーをモック／スタブするか、リクエストの形状（shape）変換をそのままキャプチャする）
* それ自体が本番環境でしか再現できない場合（レート制限、認可ポリシーなど）は、本番テストの範囲を絞り、環境変数によるオプトイン制にする
* バグを捕捉できる最小のレイヤーを優先してターゲットにする：
  * プロバイダーのリクエスト変換／リプレイのバグ → models の直接テスト
  * Gateway のセッション／履歴／tool パイプラインのバグ → Gateway のライブスモークテスト、または CI セーフな Gateway モックテスト