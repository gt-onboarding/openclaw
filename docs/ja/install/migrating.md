---
title: 移行
summary: "OpenClaw のインストール環境を別のマシンへ移行する"
read_when:
  - OpenClaw を新しいノートPCやサーバーに移行するとき
  - セッション、認証情報、およびチャネルのログイン状態（WhatsApp など）を維持したいとき
---

<div id="migrating-openclaw-to-a-new-machine">
  # OpenClaw を新しいマシンに移行する
</div>

このガイドでは、**オンボーディングをやり直すことなく**、1 台のマシンから別のマシンへ OpenClaw Gateway を移行する方法を説明します。

移行の基本的な考え方はシンプルです:

* **state ディレクトリ**（`$OPENCLAW_STATE_DIR`、デフォルト: `~/.openclaw/`）をコピーする — ここには設定、認証情報、セッション、チャネルの状態が含まれます。
* **ワークスペース**（デフォルトでは `~/.openclaw/workspace/`）をコピーする — ここにはエージェントのファイル（メモリ、プロンプトなど）が含まれます。

ただし、**プロファイル**、**権限（パーミッション）**、**一部のみをコピーしてしまうこと**に関して、ありがちな落とし穴がいくつかあります。

<div id="before-you-start-what-you-are-migrating">
  ## 作業を始める前に（移行するもの）
</div>

<div id="1-identify-your-state-directory">
  ### 1) state ディレクトリを特定する
</div>

ほとんどのインストールでは、デフォルトは次のとおりです：

* **State dir:** `~/.openclaw/`

ただし、次のような場合は異なる可能性があります：

* `--profile <name>`（多くの場合 `~/.openclaw-<profile>/` になります）
* `OPENCLAW_STATE_DIR=/some/path`

どれか分からない場合は、**古い**マシン上で次を実行してください：

```bash
openclaw status
```

出力の中から `OPENCLAW_STATE_DIR` やプロファイルに関する記述を探してください。複数の Gateway を実行している場合は、各プロファイルごとに同じ作業を繰り返します。

<div id="2-identify-your-workspace">
  ### 2) 自分のワークスペースを確認する
</div>

一般的なデフォルト:

* `~/.openclaw/workspace/`（推奨ワークスペース）
* 自分で作成した任意のフォルダ

ワークスペースは、`MEMORY.md`、`USER.md`、`memory/*.md` などのファイルが配置されている場所です。

<div id="3-understand-what-you-will-preserve">
  ### 3) 何が引き継がれるかを理解する
</div>

**状態ディレクトリ**とワークスペースの**両方**をコピーした場合、次のものが保持されます:

* Gateway 設定 (`openclaw.json`)
* 認証プロファイル / API キー / OAuth トークン
* セッション履歴 + エージェント状態
* チャンネル状態（例: WhatsApp のログイン / セッション）
* ワークスペース内のファイル（memory、スキル用メモなど）

ワークスペースだけ（例: Git 経由）をコピーした場合、**引き継がれない**もの:

* セッション
* 認証情報
* チャンネルのログイン情報

これらは `$OPENCLAW_STATE_DIR` 以下に保存されています。

<div id="migration-steps-recommended">
  ## 推奨される移行手順
</div>

<div id="step-0-make-a-backup-old-machine">
  ### ステップ 0 — バックアップを作成する（旧マシン）
</div>

**旧**マシン側で、まず Gateway を停止して、コピー中にファイルが変更されないようにします。

```bash
openclaw gateway stop
```

（オプションだが推奨）state ディレクトリとワークスペースをアーカイブする：

```bash
# プロファイルやカスタムロケーションを使用する場合はパスを調整してください
cd ~
tar -czf openclaw-state.tgz .openclaw

tar -czf openclaw-workspace.tgz .openclaw/workspace
```

複数のプロファイルや state ディレクトリ（例: `~/.openclaw-main`, `~/.openclaw-work`）がある場合は、それぞれ個別にアーカイブしてください。

<div id="step-1-install-openclaw-on-the-new-machine">
  ### ステップ 1 — 新しいマシンに OpenClaw をインストールする
</div>

**新しい**マシン上で、CLI（必要であれば Node.js も）をインストールします：

* 参照： [Install](/ja/install)

この段階では、オンボーディング処理によって新しい `~/.openclaw/` が作成されても問題ありません — 次のステップで上書きします。

<div id="step-2-copy-the-state-dir-workspace-to-the-new-machine">
  ### Step 2 — state ディレクトリとワークスペースを新しいマシンへコピーする
</div>

**両方**をコピーします:

* `$OPENCLAW_STATE_DIR`（デフォルト `~/.openclaw/`）
* あなたのワークスペース（デフォルト `~/.openclaw/workspace/`）

よくある方法:

* `scp` で tarball を転送して展開する
* SSH 経由で `rsync -a` を使う
* 外部ドライブを利用する

コピー後は次を確認してください:

* 隠しディレクトリが含まれていること（例: `.openclaw/`）
* Gateway を実行するユーザーに対して、ファイル所有権が正しく設定されていること

<div id="step-3-run-doctor-migrations-service-repair">
  ### ステップ 3 — doctor の実行（マイグレーション + サービス修復）
</div>

**新しい**マシン上で:

```bash
openclaw doctor
```

`doctor` は「安全でつまらない」コマンドです。サービスを修復し、設定マイグレーションを適用し、不整合を検出して警告します。

次に:

```bash
openclaw gateway restart
openclaw status
```

<div id="common-footguns-and-how-to-avoid-them">
  ## よくあるハマりどころ（とその回避方法）
</div>

<div id="footgun-profile-state-dir-mismatch">
  ### フットガン: profile / state-dir の不一致
</div>

古い Gateway を profile（または `OPENCLAW_STATE_DIR`）指定で動かしていて、新しい Gateway が別のものを使っている場合、次のような症状が発生します:

* 設定変更が反映されない
* チャンネルが消えている / ログアウト状態になっている
* セッション履歴が空になる

対処法: 移行に使った **同じ** profile/state dir で Gateway/サービスを起動し、そのうえで次を再実行します:

```bash
openclaw doctor
```

<div id="footgun-copying-only-openclawjson">
  ### 危険ポイント: `openclaw.json` だけをコピーすること
</div>

`openclaw.json` だけでは不十分です。多くのプロバイダーは、次の場所配下に状態を保存します:

* `$OPENCLAW_STATE_DIR/credentials/`
* `$OPENCLAW_STATE_DIR/agents/<agentId>/...`

必ず `$OPENCLAW_STATE_DIR` ディレクトリ全体を移行してください。

<div id="footgun-permissions-ownership">
  ### Footgun: パーミッション / 所有権
</div>

root でコピーしたりユーザーを変更した場合、Gateway が認証情報やセッションを読み取れないことがあります。

対処: state ディレクトリとワークスペースの所有者が、Gateway を実行しているユーザーになっていることを確認してください。

<div id="footgun-migrating-between-remotelocal-modes">
  ### やらかしポイント: リモート/ローカルモード間の移行
</div>

* UI (WebUI/TUI) が **リモート** Gateway に接続されている場合、セッションストアとワークスペースはリモートホスト側にあります。
* ノートPCを移行しても、リモート Gateway の状態は移行されません。

リモートモードを使っている場合は、**Gateway ホスト**側を移行してください。

<div id="footgun-secrets-in-backups">
  ### フットガン：バックアップ内のシークレット
</div>

`$OPENCLAW_STATE_DIR` にはシークレット（API キー、OAuth トークン、WhatsApp のクレデンシャル）が含まれています。バックアップも本番環境のシークレットと同様に扱ってください:

* 暗号化して保存する
* 安全でないチャネルで共有しない
* 漏えいが疑われる場合はキーをローテーションする

<div id="verification-checklist">
  ## 検証チェックリスト
</div>

新しいマシン上で、次を確認してください:

* `openclaw status` で Gateway が稼働していること
* チャネルが引き続き接続されていること（例: WhatsApp で再ペアリングが不要であること）
* ダッシュボードが開き、既存のセッションが表示されること
* ワークスペース内のファイル（メモリ、設定）が存在すること

<div id="related">
  ## 関連項目
</div>

* [Doctor](/ja/gateway/doctor)
* [Gateway のトラブルシューティング](/ja/gateway/troubleshooting)
* [OpenClaw はどこにデータを保存しますか？](/ja/help/faq#where-does-openclaw-store-its-data)