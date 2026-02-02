---
title: フック
summary: "`openclaw hooks`（エージェントフック）の CLI リファレンス"
read_when:
  - エージェントフックを管理したいとき
  - フックをインストールまたは更新したいとき
---

<div id="openclaw-hooks">
  # `openclaw hooks`
</div>

エージェントフック（`/new`、`/reset`、Gateway の起動時などのコマンドに対するイベント駆動型の自動化処理）を管理します。

関連:

* フック: [Hooks](/ja/hooks)
* プラグインフック: [Plugins](/ja/plugin#plugin-hooks)

<div id="list-all-hooks">
  ## フックを一覧表示する
</div>

```bash
openclaw hooks list
```

ワークスペース、マネージド、およびバンドル済みディレクトリから検出されたすべてのフックを一覧表示します。

**オプション:**

* `--eligible`: 利用可能なフックのみを表示する（要件を満たしているもの）
* `--json`: JSON 形式で出力する
* `-v, --verbose`: 不足している要件も含めた詳細情報を表示する

**出力例:**

```
Hooks (4/4 ready)

Ready:
  🚀 boot-md ✓ - Run BOOT.md on gateway startup
  📝 command-logger ✓ - Log all command events to a centralized audit file
  💾 session-memory ✓ - Save session context to memory when /new command is issued
  😈 soul-evil ✓ - Swap injected SOUL content during a purge window or by random chance
```

**例（詳細モード）:**

```bash
openclaw hooks list --verbose
```

利用不可のフックについて、不足している要件を表示します。

**例（JSON）:**

```bash
openclaw hooks list --json
```

プログラムから利用できるよう、構造化された JSON を返します。

<div id="get-hook-information">
  ## フックの情報を取得する
</div>

```bash
openclaw hooks info <name>
```

特定のフックの詳細情報を表示します。

**引数:**

* `<name>`: フック名（例: `session-memory`）

**オプション:**

* `--json`: JSON 形式で出力します

**例:**

```bash
openclaw hooks info session-memory
```

**出力：**

```
💾 session-memory ✓ Ready

Save session context to memory when /new command is issued

Details:
  Source: openclaw-bundled
  Path: /path/to/openclaw/hooks/bundled/session-memory/HOOK.md
  Handler: /path/to/openclaw/hooks/bundled/session-memory/handler.ts
  Homepage: https://docs.openclaw.ai/hooks#session-memory
  Events: command:new

Requirements:
  Config: ✓ workspace.dir
```

<div id="check-hooks-eligibility">
  ## フックの適用条件を確認する
</div>

```bash
openclaw hooks check
```

フックの有効化可否の概要（準備完了の数と未準備の数）を表示します。

**オプション:**

* `--json`: JSON として出力

**出力例:**

```
Hooks Status

Total hooks: 4
Ready: 4
Not ready: 0
```

<div id="enable-a-hook">
  ## フックを有効にする
</div>

```bash
openclaw hooks enable <name>
```

特定のフックを有効にするには、設定ファイル（`~/.openclaw/config.json`）に追加します。

**注意:** プラグインによって管理されるフックは、`openclaw hooks list` で `plugin:<id>` と表示され、
ここから有効化／無効化することはできません。代わりにプラグイン自体を有効化／無効化してください。

**引数:**

* `<name>`: フック名（例: `session-memory`）

**例:**

```bash
openclaw hooks enable session-memory
```

**出力結果:**

```
✓ フックを有効化しました: 💾 session-memory
```

**何を行うか:**

* フックが存在し、利用可能かどうかを確認します
* 設定ファイルの `hooks.internal.entries.<name>.enabled = true` を更新します
* 設定をディスクに保存します

**有効化後:**

* フックを再読み込みするために Gateway を再起動します（macOS ではメニューバーアプリを再起動し、開発環境では Gateway プロセスを再起動します）。

<div id="disable-a-hook">
  ## フックを無効にする
</div>

```bash
openclaw hooks disable <name>
```

特定のフックを無効化するには、設定ファイルを更新します。

**引数:**

* `<name>`: フック名（例: `command-logger`）

**例:**

```bash
openclaw hooks disable command-logger
```

**出力：**

```
⏸ Disabled hook: 📝 command-logger
```

**無効化後:**

* hook を再読み込みさせるために Gateway を再起動する

<div id="install-hooks">
  ## フックをインストールする
</div>

```bash
openclaw hooks install <path-or-spec>
```

ローカルフォルダー／アーカイブまたは npm からフックパックをインストールします。

**処理内容:**

* フックパックを `~/.openclaw/hooks/<id>` にコピーします
* インストールされたフックを `hooks.internal.entries.*` で有効化します
* インストール情報を `hooks.internal.installs` に記録します

**オプション:**

* `-l, --link`: コピーの代わりにローカルディレクトリをリンクします（`hooks.internal.load.extraDirs` に追加されます）

**サポートされているアーカイブ形式:** `.zip`, `.tgz`, `.tar.gz`, `.tar`

**例:**

```bash
# Local directory
openclaw hooks install ./my-hook-pack

# Local archive
openclaw hooks install ./my-hook-pack.zip

# NPM package
openclaw hooks install @openclaw/my-hook-pack

# コピーせずにローカルディレクトリをリンク
openclaw hooks install -l ./my-hook-pack
```

<div id="update-hooks">
  ## アップデート用フック
</div>

```bash
openclaw hooks update <id>
openclaw hooks update --all
```

インストール済みのフックパックを更新します（npm インストールの場合のみ）。

**オプション:**

* `--all`: 追跡対象のすべてのフックパックを更新
* `--dry-run`: 実際には書き込まずに、変更内容のみを表示

<div id="bundled-hooks">
  ## 同梱フック
</div>

<div id="session-memory">
  ### session-memory
</div>

`/new` を実行したときに、セッションコンテキストをメモリ上に保存します。

**有効化:**

```bash
openclaw hooks enable session-memory
```

**出力:** `~/.openclaw/workspace/memory/YYYY-MM-DD-slug.md`

**参照:** [セッションメモリに関するドキュメント](/ja/hooks#session-memory)

<div id="command-logger">
  ### command-logger
</div>

すべてのコマンドイベントを中央集約された監査ファイルに記録します。

**有効化:**

```bash
openclaw hooks enable command-logger
```

**出力先:** `~/.openclaw/logs/commands.log`

**ログの確認:**

```bash
# 最近のコマンド
tail -n 20 ~/.openclaw/logs/commands.log

# 整形表示
cat ~/.openclaw/logs/commands.log | jq .

# アクションでフィルタリング
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**参照:** [command-logger のドキュメント](/ja/hooks#command-logger)

<div id="soul-evil">
  ### soul-evil
</div>

パージ期間中、またはランダムなタイミングで、注入される `SOUL.md` の内容を `SOUL_EVIL.md` に入れ替えます。

**有効化:**

```bash
openclaw hooks enable soul-evil
```

**参照:** [SOUL Evil Hook](/ja/hooks/soul-evil)

<div id="boot-md">
  ### boot-md
</div>

Gateway の起動時（チャネルの起動後）に `BOOT.md` を実行します。

**イベント**: `gateway:startup`

**有効化**:

```bash
openclaw hooks enable boot-md
```

**参照:** [boot-md のドキュメント](/ja/hooks#boot-md)
