---
title: Openclaw
summary: "OpenClaw を個人アシスタントとして実行するためのエンドツーエンドのガイドと安全上の注意点"
read_when:
  - 新しいアシスタントインスタンスのオンボーディング時
  - 安全性および権限設定の影響を確認するとき
---

<div id="building-a-personal-assistant-with-openclaw">
  # OpenClaw でパーソナルアシスタントを構築する
</div>

OpenClaw は、**Pi** エージェント向けの WhatsApp + Telegram + Discord + iMessage 用 Gateway です。プラグインを追加することで Mattermost にも対応できます。このガイドは「パーソナルアシスタント」用のセットアップガイドで、常時稼働するエージェントとして振る舞う専用の WhatsApp 番号を 1 つ用意する手順を説明します。

<div id="safety-first">
  ## ⚠️ まずは安全第一
</div>

あなたはエージェントに、次のような権限を与えようとしています。

* （Pi ツールのセットアップ次第で）あなたのマシン上でコマンドを実行する
* ワークスペース内のファイルを read/write（読み書き）する
* WhatsApp/Telegram/Discord/Mattermost（プラグイン）経由でメッセージを外部に送信する

最初は慎重な設定から始めてください:

* 必ず `channels.whatsapp.allowFrom` を設定すること（あなたの個人 Mac を「open」＝誰からのメッセージも無制限に受け付ける状態で動かしてはいけません）。
* アシスタント専用の WhatsApp 番号を使用すること。
* ハートビートは現在、デフォルトで 30 分ごとに実行されます。セットアップを信頼できるようになるまでは、`agents.defaults.heartbeat.every: "0m"` を設定して無効化してください。

<div id="prerequisites">
  ## 前提条件
</div>

* Node **22+**
* PATH 上で利用可能な OpenClaw（推奨: グローバルインストール）
* アシスタント用の 2つ目の電話番号（SIM/eSIM/プリペイド）

```bash
npm install -g openclaw@latest
# または: pnpm add -g openclaw@latest
```

送信元（開発用）から：

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # 初回実行時にUI依存関係を自動インストールします
pnpm build
pnpm link --global
```

<div id="the-two-phone-setup-recommended">
  ## 2台のスマートフォン構成（推奨）
</div>

目指すべき構成は次のとおりです：

```
Your Phone (personal)          Second Phone (assistant)
┌─────────────────┐           ┌─────────────────┐
│  Your WhatsApp  │  ──────▶  │  Assistant WA   │
│  +1-555-YOU     │  message  │  +1-555-ASSIST  │
└─────────────────┘           └────────┬────────┘
                                       │ linked via QR
                                       ▼
                              ┌─────────────────┐
                              │  Your Mac       │
                              │  (openclaw)      │
                              │    Pi agent     │
                              └─────────────────┘
```

個人の WhatsApp を OpenClaw にリンクすると、あなた宛てのすべてのメッセージが「エージェント入力」として扱われます。これは多くの場合、望む動作ではありません。

<div id="5-minute-quick-start">
  ## 5分クイックスタート
</div>

1. WhatsApp Web をペアリングします（QRコードが表示されるので、アシスタント用のスマートフォンでスキャンします）:

```bash
openclaw channels login
```

2. Gateway を起動します（このまま起動した状態にしておきます）:

```bash
openclaw gateway --port 18789
```

3. `~/.openclaw/openclaw.json` に最小限の設定を記述します：

```json5
{
  channels: { whatsapp: { allowFrom: ["+15555550123"] } }
}
```

次に、許可リストに登録した自分の電話番号からアシスタントの番号にメッセージを送信します。

オンボーディングが完了すると、Gateway トークン付きのダッシュボードを自動的に開き、トークンを埋め込んだリンクも出力します。あとで再度開くには `openclaw dashboard` を実行します。

<div id="give-the-agent-a-workspace-agents">
  ## エージェントにワークスペースを与える (AGENTS)
</div>

OpenClaw は、ワークスペースディレクトリから運用手順や「メモリ」を読み込みます。

デフォルトでは、OpenClaw は `~/.openclaw/workspace` をエージェントのワークスペースとして使用し、セットアップ／最初のエージェント実行時に、このディレクトリと初期ファイル（`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`）を自動的に作成します。`BOOTSTRAP.md` はワークスペースが新規作成されたときにのみ生成されます（削除した後に再び作成されることはありません）。

ヒント: このディレクトリは OpenClaw の「メモリ」として扱い、git リポジトリ（できれば非公開）にしておくと、`AGENTS.md` とメモリ関連ファイルのバックアップになります。git がインストールされている場合、新しく作成されたワークスペースは自動的に初期化されます。

```bash
openclaw setup
```

ワークスペースレイアウト全体とバックアップガイド: [エージェントのワークスペース](/ja/concepts/agent-workspace)
メモリのワークフロー: [メモリ](/ja/concepts/memory)

任意: `agents.defaults.workspace` で別のワークスペースを選択できます（`~` に対応）。

```json5
{
  agent: {
    workspace: "~/.openclaw/workspace"
  }
}
```

すでにリポジトリ側で独自のワークスペースファイル一式を同梱している場合は、ブートストラップ用ファイルの自動作成を完全に無効にできます。

```json5
{
  agent: {
    skipBootstrap: true
  }
}
```

<div id="the-config-that-turns-it-into-an-assistant">
  ## 「アシスタント」に仕立てるための設定
</div>

OpenClaw はデフォルトですでに妥当なアシスタント構成になっていますが、通常は次の点を調整します:

* `SOUL.md` 内のペルソナ／指示
* 思考系のデフォルト設定（必要であれば）
* ハートビート（信頼できるようになってから）

例:

```json5
{
  logging: { level: "info" },
  agent: {
    model: "anthropic/claude-opus-4-5",
    workspace: "~/.openclaw/workspace",
    thinkingDefault: "high",
    timeoutSeconds: 1800,
    // 最初は 0 で開始し、後で有効化する
    heartbeat: { every: "0m" }
  },
  channels: {
    whatsapp: {
      allowFrom: ["+15555550123"],
      groups: {
        "*": { requireMention: true }
      }
    }
  },
  routing: {
    groupChat: {
      mentionPatterns: ["@openclaw", "openclaw"]
    }
  },
  session: {
    scope: "per-sender",
    resetTriggers: ["/new", "/reset"],
    reset: {
      mode: "daily",
      atHour: 4,
      idleMinutes: 10080
    }
  }
}
```

<div id="sessions-and-memory">
  ## セッションとメモリ
</div>

* セッションファイル: `~/.openclaw/agents/<agentId>/sessions/{{SessionId}}.jsonl`
* セッションメタデータ（トークン使用量、直近のルートなど）: `~/.openclaw/agents/<agentId>/sessions/sessions.json`（レガシー: `~/.openclaw/sessions/sessions.json`）
* `/new` または `/reset` は、そのチャット用に新しいセッションを開始します（`resetTriggers` で設定可能）。単独で送信された場合、エージェントはリセット確認のために短い挨拶を返答します。
* `/compact [instructions]` はセッションコンテキストをコンパクト化し、残りのコンテキスト予算を報告します。

<div id="heartbeats-proactive-mode">
  ## ハートビート（プロアクティブモード）
</div>

デフォルトでは、OpenClaw は 30 分ごとに次のプロンプトを使ってハートビートを実行します:
`Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`
無効化するには、`agents.defaults.heartbeat.every: "0m"` を設定します。

* `HEARTBEAT.md` が存在するものの、実質的に空（空行と `# Heading` のような Markdown 見出しだけが含まれる）である場合、OpenClaw は API 呼び出しを節約するためにそのハートビート実行をスキップします。
* ファイルが存在しない場合でも、ハートビートは実行され、モデルが何を行うかを決定します。
* エージェントが `HEARTBEAT_OK`（短い追記（パディング）付きでも可。`agents.defaults.heartbeat.ackMaxChars` を参照）を返した場合、OpenClaw はそのハートビートに対する外部への配信を抑制します。
* ハートビートはエージェントのフルターンとして実行されるため、間隔を短くするとトークン消費が増加します。

```json5
{
  agent: {
    heartbeat: { every: "30m" }
  }
}
```

<div id="media-in-and-out">
  ## メディアの入出力
</div>

受信した添付ファイル（画像／音声／ドキュメント）は、テンプレート経由でコマンドから利用できます：

* `{{MediaPath}}`（ローカルの一時ファイルパス）
* `{{MediaUrl}}`（疑似 URL）
* `{{Transcript}}`（音声文字起こしが有効な場合）

エージェントから送信する添付ファイル：`MEDIA:<path-or-url>` を単独の行（スペースなし）として含めます。例:

```
スクリーンショットは以下の通りです。
MEDIA:/tmp/screenshot.png
```

OpenClaw はこれらを抽出し、テキストと一緒にメディアとして送信します。

<div id="operations-checklist">
  ## 運用チェックリスト
</div>

```bash
openclaw status          # ローカルステータス (認証情報、セッション、キューに入ったイベント)
openclaw status --all    # 完全診断 (読み取り専用、貼り付け可能)
openclaw status --deep   # Gatewayヘルスプローブを追加 (Telegram + Discord)
openclaw health --json   # Gatewayヘルススナップショット (WS)
```

ログファイルは `/tmp/openclaw/` に保存されます（デフォルトでは `openclaw-YYYY-MM-DD.log`）。

<div id="next-steps">
  ## 次のステップ
</div>

* WebChat: [WebChat](/ja/web/webchat)
* Gateway の運用: [Gateway runbook](/ja/gateway)
* Cron とウェイクアップ処理: [Cron jobs](/ja/automation/cron-jobs)
* macOS メニューバー常駐コンパニオン: [OpenClaw macOS app](/ja/platforms/macos)
* iOS ノードアプリ: [iOS app](/ja/platforms/ios)
* Android ノードアプリ: [Android app](/ja/platforms/android)
* Windows の対応状況: [Windows (WSL2)](/ja/platforms/windows)
* Linux の対応状況: [Linux app](/ja/platforms/linux)
* セキュリティ: [Security](/ja/gateway/security)