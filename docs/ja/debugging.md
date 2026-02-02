---
title: デバッグ
summary: "デバッグツール: ウォッチモード、生のモデルストリーム、推論リークのトレース"
read_when:
  - 推論リークを確認するために生のモデル出力を検査する必要がある
  - 反復的に開発しながら Gateway をウォッチモードで実行したい
  - 再現可能なデバッグワークフローが必要である
---

<div id="debugging">
  # デバッグ
</div>

このページでは、特にプロバイダーが通常のテキストに推論内容を混在させる場合の、ストリーミング出力をデバッグするための補助機能について説明します。

<div id="runtime-debug-overrides">
  ## ランタイムデバッグ用オーバーライド
</div>

チャット内で `/debug` を使用して、**ランタイム時のみ有効** な設定オーバーライド（メモリ上のみで、ディスクには保存されない）を行えます。
`/debug` はデフォルトでは無効です。`commands.debug: true` に設定して有効化してください。
`openclaw.json` を編集せずに、あまり表に出ない細かい設定を一時的に切り替えたいときに便利です。

例:

```
/debug show
/debug set messages.responsePrefix="[openclaw]"
/debug unset messages.responsePrefix
/debug reset
```

`/debug reset` はすべてのオーバーライド設定をクリアし、ディスク上の設定ファイルの内容に戻します。


<div id="gateway-watch-mode">
  ## Gateway ウォッチモード
</div>

素早く反復開発するには、ファイルウォッチャーを有効にして Gateway を実行します。

```bash
pnpm gateway:watch --force
```

これは次のようにマッピングされます：

```bash
tsx watch src/entry.ts gateway --force
```

`gateway:watch` の後ろに任意の Gateway CLI フラグを追加すると、再起動のたびにそのまま渡されます。


<div id="dev-profile-dev-gateway-dev">
  ## Devプロファイル + dev Gateway（--dev）
</div>

devプロファイルを使うと、状態を分離した、安全で使い捨て可能なデバッグ用セットアップを
立ち上げられます。`--dev` フラグは **2つ** あります:

* **グローバル `--dev`（プロファイル）:** 状態を `~/.openclaw-dev` 配下に分離し、
  Gateway のデフォルトポートを `19001` に設定します（派生ポートもこれに合わせてシフトします）。
* **`gateway --dev`: Gateway に、デフォルトの config と
  ワークスペースを（存在しない場合に）自動作成させ**、BOOTSTRAP.md をスキップします。

推奨フロー（devプロファイル + devブートストラップ）:

```bash
pnpm gateway:dev
OPENCLAW_PROFILE=dev openclaw tui
```

まだグローバルインストールをしていない場合は、`pnpm openclaw ...` で CLI を実行してください。

これにより、次の処理が行われます:

1. **プロファイルの分離**（グローバル `--dev`）
   * `OPENCLAW_PROFILE=dev`
   * `OPENCLAW_STATE_DIR=~/.openclaw-dev`
   * `OPENCLAW_CONFIG_PATH=~/.openclaw-dev/openclaw.json`
   * `OPENCLAW_GATEWAY_PORT=19001`（ブラウザ／キャンバス側もそれに応じて切り替わります）

2. **開発ブートストラップ**（`gateway --dev`）
   * 最小構成ファイルが存在しない場合に書き込みます（`gateway.mode=local`、ループバックにバインド）。
   * `agent.workspace` を開発用ワークスペースに設定します。
   * `agent.skipBootstrap=true` を設定します（BOOTSTRAP.md は使用しない）。
   * ワークスペース内に対象ファイルがない場合は初期ファイルとして作成します:
     `AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`。
   * デフォルトのアイデンティティ: **C3‑PO**（プロトコルドロイド）。
   * 開発モードではチャネルプロバイダーをスキップします（`OPENCLAW_SKIP_CHANNELS=1`）。

リセットフロー（クリーンスタート）:

```bash
pnpm gateway:dev:reset
```

注意: `--dev` は**グローバル**なプロファイル用フラグであり、一部のランナー側で消費されてしまうことがあります。
明示的に指定する必要がある場合は、環境変数の形式を使用してください:

```bash
OPENCLAW_PROFILE=dev openclaw gateway --dev --reset
```

`--reset` は、設定、認証情報、セッション、および開発用ワークスペースを（`rm` ではなく `trash` を使って）削除し、その後デフォルトの開発用セットアップを再作成します。

ヒント：非開発用の Gateway がすでに起動している場合（launchd/systemd）、先にその Gateway を停止してください：

```bash
openclaw gateway stop
```


<div id="raw-stream-logging-openclaw">
  ## 生ストリームのログ記録 (OpenClaw)
</div>

OpenClaw では、フィルタ処理や整形を行う前の **生のアシスタントストリーム** をログに記録できます。
これは、推論がプレーンテキストのデルタとして届いているのか、
それとも独立した思考ブロックとして届いているのかを確認する最も確実な方法です。

CLI から有効化します:

```bash
pnpm gateway:watch --force --raw-stream
```

パスの上書き（任意）:

```bash
pnpm gateway:watch --force --raw-stream --raw-stream-path ~/.openclaw/logs/raw-stream.jsonl
```

対応する環境変数：

```bash
OPENCLAW_RAW_STREAM=1
OPENCLAW_RAW_STREAM_PATH=~/.openclaw/logs/raw-stream.jsonl
```

デフォルトのファイルパス:

`~/.openclaw/logs/raw-stream.jsonl`


<div id="raw-chunk-logging-pi-mono">
  ## 生チャンクのログ記録（pi-mono）
</div>

**生の OpenAI 互換チャンク** をブロックにパースする前に取得するには、
pi-mono は別のロガーを提供しています。

```bash
PI_RAW_STREAM=1
```

オプションのパス:

```bash
PI_RAW_STREAM_PATH=~/.pi-mono/logs/raw-openai-completions.jsonl
```

デフォルトのファイル:

`~/.pi-mono/logs/raw-openai-completions.jsonl`

> 注記: これは pi-mono の `openai-completions` プロバイダーを使用しているプロセスでのみ出力されます。


<div id="safety-notes">
  ## セキュリティに関する注意事項
</div>

- 生のストリームログには、プロンプト全体、ツールの出力、ユーザーデータが含まれる場合があります。
- ログはローカルに保持し、デバッグが終わったら削除してください。
- ログを共有する必要がある場合は、事前に秘密情報や個人情報（PII）を確実にマスクしてください。