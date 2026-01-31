---
title: フック
summary: "フック: コマンドおよびライフサイクルイベント向けのイベント駆動型自動化"
read_when:
  - /new、/reset、/stop、およびエージェントのライフサイクルイベントに対してイベント駆動型の自動化を行いたい
  - フックを作成、インストール、またはデバッグしたい
---

<div id="hooks">
  # フック
</div>

フックは、エージェントのコマンドやイベントに応じてアクションを自動化するための、拡張可能なイベント駆動型システムを提供します。フックはディレクトリ内から自動的に検出され、OpenClaw におけるスキルと同様に、CLI コマンド経由で管理できます。

<div id="getting-oriented">
  ## 概要の確認
</div>

Hooks は、何かが起きたときに実行される小さなスクリプトです。種類は 2 つあります:

* **Hooks**（このページ）: `/new`、`/reset`、`/stop` のようなコマンドやライフサイクルイベントなど、エージェントのイベントが発火したときに Gateway 内で実行されます。
* **Webhooks**: 外部向けの HTTP Webhook で、他のシステムから OpenClaw 内の処理をトリガーできます。[Webhook Hooks](/ja/automation/webhook) を参照するか、Gmail 用のヘルパーコマンドには `openclaw webhooks` を使用してください。

Hooks はプラグイン内にバンドルすることもできます。[Plugins](/ja/plugin#plugin-hooks) を参照してください。

主な用途:

* セッションをリセットするときにメモリスナップショットを保存する
* トラブルシューティングやコンプライアンスのためにコマンドの監査ログを保持する
* セッションの開始・終了時にフォローアップの自動化をトリガーする
* イベント発火時にエージェントのワークスペースへファイルを書き込んだり、外部 API を呼び出したりする

小さな TypeScript 関数を書けるなら、Hook も実装できます。Hooks は自動的に検出され、CLI を通じて有効化・無効化できます。

<div id="overview">
  ## 概要
</div>

hooks システムを使用すると、次のことが可能です:

* `/new` が発行されたときにセッションのコンテキストをメモリに保存する
* 監査のためにすべてのコマンドをログとして記録する
* エージェントのライフサイクルイベント発生時に独自の自動化処理をトリガーする
* コアコードを変更せずに OpenClaw の動作を拡張する

<div id="getting-started">
  ## 導入
</div>

<div id="bundled-hooks">
  ### バンドル済みフック
</div>

OpenClaw には、自動検出されるバンドル済みフックが 4 つ含まれています:

* **💾 session-memory**: `/new` を実行したときに、セッションコンテキストをエージェントのワークスペース内（デフォルト `~/.openclaw/workspace/memory/`）に保存します
* **📝 command-logger**: すべてのコマンドイベントを `~/.openclaw/logs/commands.log` に記録します
* **🚀 boot-md**: Gateway 起動時に `BOOT.md` を実行します（内部フックが有効になっている必要があります）
* **😈 soul-evil**: パージウィンドウ期間中、またはランダムなタイミングで、インジェクトされる `SOUL.md` の内容を `SOUL_EVIL.md` と入れ替えます

利用可能なフックを一覧表示:

```bash
openclaw hooks list
```

フックを有効にする：

```bash
openclaw hooks enable session-memory
```

フックの状態を確認する：

```bash
openclaw hooks check
```

詳細情報を確認する：

```bash
openclaw hooks info session-memory
```

<div id="onboarding">
  ### オンボーディング
</div>

オンボーディング（`openclaw onboard`）中に、推奨フックを有効化するかどうか確認されます。ウィザードは自動的に対象となるフックを検出し、選択肢として提示します。

<div id="hook-discovery">
  ## Hook の検出
</div>

Hook は、次の 3 つのディレクトリから自動的に検出されます（上にあるものほど優先度が高い）:

1. **ワークスペース Hook**: `<workspace>/hooks/`（エージェント単位、最優先）
2. **管理 Hook**: `~/.openclaw/hooks/`（ユーザーがインストールし、複数ワークスペースで共有）
3. **バンドル Hook**: `<openclaw>/dist/hooks/bundled/`（OpenClaw に同梱）

管理 Hook のディレクトリは、**単一の Hook** または **Hook パック**（パッケージディレクトリ）のいずれかになります。

各 Hook は、次の内容を含むディレクトリです:

```
my-hook/
├── HOOK.md          # メタデータ + ドキュメント
└── handler.ts       # ハンドラの実装
```

<div id="hook-packs-npmarchives">
  ## フックパック（npm/アーカイブ）
</div>

フックパックは、`package.json` の `openclaw.hooks` によって 1 つ以上のフックをエクスポートする標準的な npm パッケージです。次のコマンドでインストールします：

```bash
openclaw hooks install <path-or-spec>
```

`package.json` の例：

```json
{
  "name": "@acme/my-hooks",
  "version": "0.1.0",
  "openclaw": {
    "hooks": ["./hooks/my-hook", "./hooks/other-hook"]
  }
}
```

各エントリは、`HOOK.md` と `handler.ts`（または `index.ts`）を含むフック用ディレクトリを参照します。
フックパックは依存関係を同梱でき、それらは `~/.openclaw/hooks/<id>` 配下にインストールされます。

<div id="hook-structure">
  ## フックの構造
</div>

<div id="hookmd-format">
  ### HOOK.md フォーマット
</div>

`HOOK.md` ファイルには、YAML フロントマターのメタデータと Markdown ドキュメントが含まれます。

```markdown
---
name: my-hook
description: "Short description of what this hook does"
homepage: https://docs.openclaw.ai/hooks#my-hook
metadata: {"openclaw":{"emoji":"🔗","events":["command:new"],"requires":{"bins":["node"]}}}
---

# My Hook

Detailed documentation goes here...

## What It Does

- Listens for `/new` commands
- Performs some action
- Logs the result

## Requirements

- Node.js must be installed

## Configuration

No configuration needed.
```

<div id="metadata-fields">
  ### メタデータフィールド
</div>

`metadata.openclaw` オブジェクトでは次の項目をサポートします:

* **`emoji`**: CLI で表示する絵文字 (例: `"💾"`)
* **`events`**: リッスンするイベントの配列 (例: `["command:new", "command:reset"]`)
* **`export`**: 使用する名前付きエクスポート (デフォルトは `"default"`)
* **`homepage`**: ドキュメントの URL
* **`requires`**: オプションの要件
  * **`bins`**: PATH 上に必要なバイナリ (例: `["git", "node"]`)
  * **`anyBins`**: このバイナリ群のうち少なくとも 1 つがインストールされている必要がある
  * **`env`**: 必須の環境変数
  * **`config`**: 必須の設定パス (例: `["workspace.dir"]`)
  * **`os`**: 必須のプラットフォーム (例: `["darwin", "linux"]`)
* **`always`**: 適格性チェックをバイパスするかどうかを指定する (boolean)
* **`install`**: インストール方法 (同梱フックの場合: `[{"id":"bundled","kind":"bundled"}]`)

<div id="handler-implementation">
  ### ハンドラーの実装
</div>

`handler.ts` ファイルは `HookHandler` 関数をエクスポートする。

```typescript
import type { HookHandler } from '../../src/hooks/hooks.js';

const myHandler: HookHandler = async (event) => {
  // 'new' コマンドでのみトリガー
  if (event.type !== 'command' || event.action !== 'new') {
    return;
  }

  console.log(`[my-hook] New command triggered`);
  console.log(`  セッション: ${event.sessionKey}`);
  console.log(`  Timestamp: ${event.timestamp.toISOString()}`);

  // ここにカスタムロジックを記述

  // 必要に応じてユーザーにメッセージを送信
  event.messages.push('✨ My hook executed!');
};

export default myHandler;
```

<div id="event-context">
  #### イベントコンテキスト
</div>

各イベントには次の情報が含まれます：

```typescript
{
  type: 'command' | 'session' | 'agent' | 'gateway',
  action: string,              // e.g., 'new', 'reset', 'stop'
  sessionKey: string,          // Session identifier
  timestamp: Date,             // When the event occurred
  messages: string[],          // ユーザーに送信するメッセージをここにプッシュ
  context: {
    sessionEntry?: SessionEntry,
    sessionId?: string,
    sessionFile?: string,
    commandSource?: string,    // e.g., 'whatsapp', 'telegram'
    senderId?: string,
    workspaceDir?: string,
    bootstrapFiles?: WorkspaceBootstrapFile[],
    cfg?: OpenClawConfig
  }
}
```

<div id="event-types">
  ## イベントの種類
</div>

<div id="command-events">
  ### コマンドイベント
</div>

エージェントコマンドが実行されたときにトリガーされます:

* **`command`**: すべてのコマンドイベント（汎用リスナー）
* **`command:new`**: `/new` コマンドが実行されたとき
* **`command:reset`**: `/reset` コマンドが実行されたとき
* **`command:stop`**: `/stop` コマンドが実行されたとき

<div id="agent-events">
  ### エージェントイベント
</div>

* **`agent:bootstrap`**: ワークスペースのブートストラップファイルが投入される前（フックが `context.bootstrapFiles` を変更できる）

<div id="gateway-events">
  ### Gateway イベント
</div>

Gateway の起動時に発生します:

* **`gateway:startup`**: チャネルが起動し、フックが読み込まれた後

<div id="tool-result-hooks-plugin-api">
  ### ツール結果フック（プラグイン API）
</div>

これらのフックはイベントストリームリスナーではありません。OpenClaw がツール結果を永続化する前に、プラグインが同期的にツール結果を調整できるようにします。

* **`tool_result_persist`**: ツール結果がセッショントランスクリプトに書き込まれる前に変換します。同期フックである必要があります。更新済みのツール結果ペイロードを返すか、そのまま維持する場合は `undefined` を返します。詳細は [Agent Loop](/ja/concepts/agent-loop) を参照してください。

<div id="future-events">
  ### 今後のイベント
</div>

予定されているイベント種別:

* **`session:start`**: 新しいセッションが開始されたとき
* **`session:end`**: セッションが終了したとき
* **`agent:error`**: エージェントでエラーが発生したとき
* **`message:sent`**: メッセージが送信されたとき
* **`message:received`**: メッセージが受信されたとき

<div id="creating-custom-hooks">
  ## カスタムフックの作成
</div>

<div id="1-choose-location">
  ### 1. 配置場所を選択する
</div>

* **ワークスペースフック** (`<workspace>/hooks/`): エージェント単位で適用され、優先度が最も高い
* **マネージドフック** (`~/.openclaw/hooks/`): 複数ワークスペース間で共有される

<div id="2-create-directory-structure">
  ### 2. ディレクトリ構造の作成
</div>

```bash
mkdir -p ~/.openclaw/hooks/my-hook
cd ~/.openclaw/hooks/my-hook
```

<div id="3-create-hookmd">
  ### 3. HOOK.md を作成
</div>

```markdown
---
name: my-hook
description: "Does something useful"
metadata: {"openclaw":{"emoji":"🎯","events":["command:new"]}}
---

# My Custom Hook

This hook does something useful when you issue `/new`.
```

<div id="4-create-handlerts">
  ### 4. handler.ts を作成
</div>

```typescript
import type { HookHandler } from '../../src/hooks/hooks.js';

const handler: HookHandler = async (event) => {
  if (event.type !== 'command' || event.action !== 'new') {
    return;
  }

  console.log('[my-hook] Running!');
  // ここにロジックを記述してください
};

export default handler;
```

<div id="5-enable-and-test">
  ### 5. 有効化とテスト
</div>

```bash
# Verify hook is discovered
openclaw hooks list

# Enable it
openclaw hooks enable my-hook

# Gateway プロセスを再起動する（macOS ではメニューバーアプリを再起動、または開発プロセスを再起動）

# Trigger the event
# Send /new via your messaging channel
```

<div id="configuration">
  ## 設定
</div>

<div id="new-config-format-recommended">
  ### 新しい設定形式（推奨）
</div>

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "session-memory": { "enabled": true },
        "command-logger": { "enabled": false }
      }
    }
  }
}
```

<div id="per-hook-configuration">
  ### フックごとの設定
</div>

フックごとに個別の設定を定義できます。

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "my-hook": {
          "enabled": true,
          "env": {
            "MY_CUSTOM_VAR": "value"
          }
        }
      }
    }
  }
}
```

<div id="extra-directories">
  ### 追加ディレクトリ
</div>

フックを追加のディレクトリから読み込みます:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "load": {
        "extraDirs": ["/path/to/more/hooks"]
      }
    }
  }
}
```

<div id="legacy-config-format-still-supported">
  ### レガシー設定形式（引き続きサポート）
</div>

古い設定形式は、後方互換性のために現在も利用できます。

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "handlers": [
        {
          "event": "command:new",
          "module": "./hooks/handlers/my-handler.ts",
          "export": "default"
        }
      ]
    }
  }
}
```

**移行**: 新規のフックには、新しいディスカバリーベースのシステムを使用してください。レガシーハンドラーは、ディレクトリベースのフックの後にロードされます。

<div id="cli-commands">
  ## CLI コマンド
</div>

<div id="list-hooks">
  ### フックの一覧
</div>

```bash
# List all hooks
openclaw hooks list

# Show only eligible hooks
openclaw hooks list --eligible

# 詳細出力 (不足している要件を表示)
openclaw hooks list --verbose

# JSON output
openclaw hooks list --json
```

<div id="hook-information">
  ### フック情報
</div>

```bash
# フックに関する詳細情報を表示
openclaw hooks info session-memory

# JSON出力
openclaw hooks info session-memory --json
```

<div id="check-eligibility">
  ### 利用可否の確認
</div>

```bash
# 適格性サマリーを表示
openclaw hooks check

# JSON出力
openclaw hooks check --json
```

<div id="enabledisable">
  ### 有効化/無効化
</div>

```bash
# フックを有効化する
openclaw hooks enable session-memory

# フックを無効化する
openclaw hooks disable command-logger
```

## 同梱フック

<div id="session-memory">
  ### session-memory
</div>

`/new` を実行したときに、セッションのコンテキストをメモリに保存します。

**Events**: `command:new`

**Requirements**: `workspace.dir` が設定されている必要があります

**Output**: `<workspace>/memory/YYYY-MM-DD-slug.md`（デフォルトは `~/.openclaw/workspace`）

**動作**:

1. リセット前のセッションエントリを使って、対象の会話ログを特定する
2. 直近の会話 15 行を抽出する
3. LLM を使って、説明的なファイル名用スラッグを生成する
4. セッションメタデータを日付付きのメモリファイルに保存する

**出力例**:

```markdown
# Session: 2026-01-16 14:30:00 UTC

- **Session Key**: agent:main:main
- **Session ID**: abc123def456
- **Source**: telegram
```

**ファイル名の例**:

* `2026-01-16-vendor-pitch.md`
* `2026-01-16-api-design.md`
* `2026-01-16-1430.md` (スラッグ生成に失敗した場合のフォールバックとして用いるタイムスタンプ)

**有効化**:

```bash
openclaw hooks enable session-memory
```

<div id="command-logger">
  ### command-logger
</div>

すべてのコマンドイベントを中央の監査ファイルに記録します。

**Events**: `command`

**Requirements**: なし

**Output**: `~/.openclaw/logs/commands.log`

**What it does**:

1. イベントの詳細（コマンドアクション、タイムスタンプ、セッションキー、送信者ID、ソース）を取得します
2. JSONL 形式でログファイルに追記します
3. バックグラウンドで静かに動作します

**Example log entries**:

```jsonl
{"timestamp":"2026-01-16T14:30:00.000Z","action":"new","sessionKey":"agent:main:main","senderId":"+1234567890","source":"telegram"}
{"timestamp":"2026-01-16T15:45:22.000Z","action":"stop","sessionKey":"agent:main:main","senderId":"user@example.com","source":"whatsapp"}
```

**ログを確認**:

```bash
# View recent commands
tail -n 20 ~/.openclaw/logs/commands.log

# Pretty-print with jq
cat ~/.openclaw/logs/commands.log | jq .

# アクションでフィルタリング
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**有効にする**:

```bash
openclaw hooks enable command-logger
```

<div id="soul-evil">
  ### soul-evil
</div>

パージウィンドウ期間中、またはランダムに、インジェクトされる `SOUL.md` の内容を `SOUL_EVIL.md` の内容に入れ替えます。

**Events**: `agent:bootstrap`

**Docs**: [SOUL Evil Hook](/ja/hooks/soul-evil)

**Output**: ファイルは書き込まれません。入れ替えはメモリ上のみで行われます。

**Enable**:

```bash
openclaw hooks enable soul-evil
```

**構成**：

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "soul-evil": {
          "enabled": true,
          "file": "SOUL_EVIL.md",
          "chance": 0.1,
          "purge": { "at": "21:00", "duration": "15m" }
        }
      }
    }
  }
}
```

<div id="boot-md">
  ### boot-md
</div>

Gateway の起動時（チャネル起動後）に `BOOT.md` を実行します。
これを動作させるには内部フックを有効化する必要があります。

**Events**: `gateway:startup`

**Requirements**: `workspace.dir` が設定されている必要があります

**What it does**:

1. ワークスペースから `BOOT.md` を読み込みます
2. エージェントランナー経由でその手順を実行します
3. message ツールを使って要求された外向きメッセージを送信します

**Enable**:

```bash
openclaw hooks enable boot-md
```

<div id="best-practices">
  ## ベストプラクティス
</div>

<div id="keep-handlers-fast">
  ### ハンドラーを高速に保つ
</div>

フックはコマンド処理中に実行されます。できるだけ処理を軽く保ってください。

```typescript
// ✓ Good - async work, returns immediately
const handler: HookHandler = async (event) => {
  void processInBackground(event); // 非同期実行して待たない
};

// ✗ Bad - blocks command processing
const handler: HookHandler = async (event) => {
  await slowDatabaseQuery(event);
  await evenSlowerAPICall(event);
};
```

<div id="handle-errors-gracefully">
  ### エラーを適切に処理する
</div>

リスクのある処理は必ずラップしてください：

```typescript
const handler: HookHandler = async (event) => {
  try {
    await riskyOperation(event);
  } catch (err) {
    console.error('[my-handler] Failed:', err instanceof Error ? err.message : String(err));
    // throw しない - 他のハンドラーを実行させる
  }
};
```

<div id="filter-events-early">
  ### イベントは早期にフィルタリングする
</div>

イベントが対象外であれば早期に return する:

```typescript
const handler: HookHandler = async (event) => {
  // 'new' コマンドのみ処理する
  if (event.type !== 'command' || event.action !== 'new') {
    return;
  }

  // ここに処理ロジックを記述
};
```

<div id="use-specific-event-keys">
  ### 特定のイベントキーを使用する
</div>

可能な場合は、メタデータに具体的なイベントキーを指定します。

```yaml
metadata: {"openclaw":{"events":["command:new"]}}  # 具体的なイベント指定
```

次のようにするのではなく：

```yaml
metadata: {"openclaw":{"events":["command"]}}      # 汎用的 - オーバーヘッドが大きい
```

<div id="debugging">
  ## デバッグ
</div>

<div id="enable-hook-logging">
  ### Hook ログを有効化する
</div>

Gateway は起動時にフックの読み込みをログに記録します:

```
Registered hook: session-memory -> command:new
Registered hook: command-logger -> command
Registered hook: boot-md -> gateway:startup
```

<div id="check-discovery">
  ### 検出状況の確認
</div>

検出されたすべてのフックを一覧表示します。

```bash
openclaw hooks list --verbose
```

<div id="check-registration">
  ### 登録を確認する
</div>

ハンドラー内で、呼び出されたタイミングをログに出力してください:

```typescript
const handler: HookHandler = async (event) => {
  console.log('[my-handler] Triggered:', event.type, event.action);
  // ロジックを記述
};
```

<div id="verify-eligibility">
  ### 対象可否の確認
</div>

フックが実行対象外になっている理由を確認します：

```bash
openclaw hooks info my-hook
```

出力に不足している要件がないか確認してください。

<div id="testing">
  ## テスト
</div>

<div id="gateway-logs">
  ### Gateway ログ
</div>

フックの実行を確認するために Gateway のログを監視します。

```bash
# macOS
./scripts/clawlog.sh -f

# その他のプラットフォーム
tail -f ~/.openclaw/gateway.log
```

<div id="test-hooks-directly">
  ### フックを直接テストする
</div>

ハンドラーを単体でテストする：

```typescript
import { test } from 'vitest';
import { createHookEvent } from './src/hooks/hooks.js';
import myHandler from './hooks/my-hook/handler.js';

test('my handler works', async () => {
  const event = createHookEvent('command', 'new', 'test-session', {
    foo: 'bar'
  });

  await myHandler(event);

  // Assert side effects
});
```

<div id="architecture">
  ## アーキテクチャ
</div>

<div id="core-components">
  ### コアコンポーネント
</div>

* **`src/hooks/types.ts`**: 型定義
* **`src/hooks/workspace.ts`**: ディレクトリのスキャンと読み込み
* **`src/hooks/frontmatter.ts`**: HOOK.md メタデータのパース
* **`src/hooks/config.ts`**: 適用可否チェック
* **`src/hooks/hooks-status.ts`**: ステータス報告
* **`src/hooks/loader.ts`**: 動的モジュールローダー
* **`src/cli/hooks-cli.ts`**: CLI コマンド
* **`src/gateway/server-startup.ts`**: Gateway 起動時にフックをロード
* **`src/auto-reply/reply/commands-core.ts`**: コマンドイベントをトリガーする

<div id="discovery-flow">
  ### ディスカバリーフロー
</div>

```
Gateway startup
    ↓
Scan directories (workspace → managed → bundled)
    ↓
Parse HOOK.md files
    ↓
Check eligibility (bins, env, config, os)
    ↓
Load handlers from eligible hooks
    ↓
Register handlers for events
```

<div id="event-flow">
  ### イベントフロー
</div>

```
ユーザーが /new を送信
    ↓
コマンド検証
    ↓
フックイベント作成
    ↓
フックトリガー（登録済みハンドラー全て）
    ↓
コマンド処理継続
    ↓
セッションリセット
```

<div id="troubleshooting">
  ## トラブルシューティング
</div>

<div id="hook-not-discovered">
  ### フックが検出されない
</div>

1. ディレクトリ構造を確認する:
   ```bash
   ls -la ~/.openclaw/hooks/my-hook/
   # HOOK.md と handler.ts が表示されているはず
   ```

2. HOOK.md の形式を確認する:
   ```bash
   cat ~/.openclaw/hooks/my-hook/HOOK.md
   # name とメタデータを含む YAML フロントマターがあるはず
   ```

3. 検出されたすべてのフックを一覧表示する:
   ```bash
   openclaw hooks list
   ```

<div id="hook-not-eligible">
  ### Hook が適用条件を満たしていない
</div>

要件を確認してください：

```bash
openclaw hooks info my-hook
```

次の項目に不足や問題がないか確認してください:

* バイナリ（`PATH` を確認）
* 環境変数
* 設定値
* OS との互換性

<div id="hook-not-executing">
  ### Hook が実行されない
</div>

1. Hook が有効になっていることを確認します:
   ```bash
   openclaw hooks list
   # 有効な Hook には ✓ が表示されているはずです
   ```

2. Gateway プロセスを再起動して Hook を再読み込みします。

3. エラーがないか Gateway のログを確認します:
   ```bash
   ./scripts/clawlog.sh | grep hook
   ```

<div id="handler-errors">
  ### ハンドラーエラー
</div>

TypeScript や import のエラーがないか確認してください。

```bash
# 直接インポートをテスト
node -e "import('./path/to/handler.ts').then(console.log)"
```

<div id="migration-guide">
  ## 移行ガイド
</div>

<div id="from-legacy-config-to-discovery">
  ### 従来の設定から Discovery への移行
</div>

**変更前**：

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "handlers": [
        {
          "event": "command:new",
          "module": "./hooks/handlers/my-handler.ts"
        }
      ]
    }
  }
}
```

**移行後**:

1. フック用ディレクトリを作成します:
   ```bash
   mkdir -p ~/.openclaw/hooks/my-hook
   mv ./hooks/handlers/my-handler.ts ~/.openclaw/hooks/my-hook/handler.ts
   ```

2. HOOK.md を作成します:
   ```markdown
   ---
   name: my-hook
   description: "My custom hook"
   metadata: {"openclaw":{"emoji":"🎯","events":["command:new"]}}
   ---

   # My Hook

   有用な処理を行います。
   ```

3. 設定を更新します:
   ```json
   {
     "hooks": {
       "internal": {
         "enabled": true,
         "entries": {
           "my-hook": { "enabled": true }
         }
       }
     }
   }
   ```

4. Gateway プロセスを確認し、再起動します:
   ```bash
   openclaw hooks list
   # 次のように表示されていれば OK: 🎯 my-hook ✓
   ```

**移行のメリット**:

* 自動検出
* CLI による管理
* 利用可否チェック
* ドキュメントの改善
* 一貫した構造

<div id="see-also">
  ## 関連項目
</div>

* [CLI リファレンス: Hooks](/ja/cli/hooks)
* [同梱 Hooks の README](https://github.com/openclaw/openclaw/tree/main/src/hooks/bundled)
* [Webhook フック](/ja/automation/webhook)
* [設定](/ja/gateway/configuration#hooks)