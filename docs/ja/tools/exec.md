---
title: Exec
summary: "Exec ツールの使い方、標準入力モードと TTY サポート"
read_when:
  - Exec ツールを使用または変更するとき
  - 標準入力や TTY の挙動をデバッグするとき
---

<div id="exec-tool">
  # Exec ツール
</div>

ワークスペース内でシェルコマンドを実行します。`process` を介して、フォアグラウンドおよびバックグラウンドでの実行をサポートします。
`process` が許可されていない場合、`exec` は同期的に実行され、`yieldMs` / `background` は無視されます。
バックグラウンドでのセッションはエージェントごとにスコープされ、`process` からは同じエージェントのセッションのみが参照できます。

<div id="parameters">
  ## パラメータ
</div>

* `command` (必須)
* `workdir` (省略時は cwd)
* `env` (キー/値の上書き)
* `yieldMs` (デフォルト 10000): 指定時間後に自動でバックグラウンド化
* `background` (bool): 即座にバックグラウンドで実行
* `timeout` (秒, デフォルト 1800): タイムアウト時にプロセスを強制終了
* `pty` (bool): 利用可能な場合は疑似ターミナルで実行 (TTY 専用 CLI、コーディングエージェント、ターミナル UI 向け)
* `host` (`sandbox | gateway | node`): 実行先
* `security` (`deny | allowlist | full`): `gateway`/`node` 用の適用モード
* `ask` (`off | on-miss | always`): `gateway`/`node` 用の承認プロンプト設定
* `node` (string): `host=node` 用のノード ID/名前
* `elevated` (bool): 権限昇格モードを要求 (Gateway ホスト); `security=full` は elevated が `full` に解決される場合にのみ強制される

注意事項:

* `host` のデフォルトは `sandbox`。
* サンドボックスがオフの場合、`elevated` は無視される (exec はすでにホスト上で実行されている)。
* `gateway`/`node` の承認は `~/.openclaw/exec-approvals.json` で制御される。
* `node` を使用するには、ペアリング済みノード (コンパニオンアプリまたはヘッドレスノードホスト) が必要。
* 複数のノードが利用可能な場合は、`exec.node` または `tools.exec.node` を設定して 1 つを選択する。
* Windows 以外のホストでは、`SHELL` が設定されている場合はそれを使用する。ただし `SHELL` が `fish` の場合は、
  fish 非対応のスクリプトを避けるため `PATH` から `bash` (または `sh`) を優先し、どちらも存在しない場合は `SHELL` にフォールバックする。
* 重要: サンドボックスは **デフォルトでオフ**。サンドボックスがオフの場合、`host=sandbox` は
  Gateway ホスト上で直接実行される (コンテナなし) ため、**承認は不要**。承認を必須にするには、
  `host=gateway` で実行し、exec の承認設定を行う (またはサンドボックスを有効化する)。

<div id="config">
  ## 設定
</div>

* `tools.exec.notifyOnExit` (デフォルト: true): true の場合、バックグラウンドで実行された exec セッションが終了したときにシステムイベントをキューに入れ、ハートビートを要求します。
* `tools.exec.approvalRunningNoticeMs` (デフォルト: 10000): 承認が必要な exec がこの値より長く実行されている場合に、一度だけ「running」通知を出します（0 で無効化）。
* `tools.exec.host` (デフォルト: `sandbox`)
* `tools.exec.security` (デフォルト: sandbox の場合は `deny`、未設定時の Gateway + ノードの場合は `allowlist`)
* `tools.exec.ask` (デフォルト: `on-miss`)
* `tools.exec.node` (デフォルト: 未設定)
* `tools.exec.pathPrepend`: exec 実行時に `PATH` に前置するディレクトリの一覧。
* `tools.exec.safeBins`: 明示的な許可リストのエントリなしで実行可能な、標準入力のみを受け付ける安全なバイナリ。

例:

```json5
{
  tools: {
    exec: {
      pathPrepend: ["~/bin", "/opt/oss/bin"]
    }
  }
}
```

<div id="path-handling">
  ### PATH の扱い
</div>

* `host=gateway`: ログインシェルの `PATH` を exec 環境にマージします（ただし、exec 呼び出し側で
  すでに `env.PATH` を設定している場合を除く）。デーモン自体は最小限の `PATH` で動作します：
  * macOS: `/opt/homebrew/bin`, `/usr/local/bin`, `/usr/bin`, `/bin`
  * Linux: `/usr/local/bin`, `/usr/bin`, `/bin`
* `host=sandbox`: コンテナ内で `sh -lc`（ログインシェル）を実行するため、`/etc/profile` によって `PATH` がリセットされる場合があります。
  OpenClaw は、プロファイル読み込み後に内部環境変数経由で `env.PATH` を先頭に付加します（シェル展開なし）；
  `tools.exec.pathPrepend` もここで適用されます。
* `host=node`: 渡した環境変数の上書きのみがノードに送信されます。`tools.exec.pathPrepend` は、
  exec 呼び出し側ですでに `env.PATH` を設定している場合にのみ適用されます。ヘッドレスノードホストは、
  ノードホスト側の PATH に前置される場合にのみ `PATH` を受け付けます（置換は不可）。macOS ノードは
  `PATH` の上書きを完全に破棄します。

エージェント単位でのノードバインディング（設定内でエージェントリストのインデックスを使用）:

```bash
openclaw config get agents.list
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
```

Control UI の Nodes タブには、同じ設定を行うための小さな「Exec ノードバインディング」パネルもあります。

<div id="session-overrides-exec">
  ## セッションのオーバーライド (`/exec`)
</div>

`/exec` を使用して、`host`、`security`、`ask`、`node` の **セッション単位の** デフォルト値を設定します。
現在の値を表示するには、引数なしで `/exec` を送信します。

例:

```
/exec host=gateway security=allowlist ask=on-miss node=mac-1
```

<div id="authorization-model">
  ## 認可モデル
</div>

`/exec` が有効になるのは、**認可された送信者**（チャネルの許可リスト／ペアリング設定と `commands.useAccessGroups`）に対してのみです。
これは **セッション状態のみ** を更新し、コンフィグには反映されません。`exec` をハードに無効化したい場合は、
ツールポリシー（`tools.deny: ["exec"]` またはエージェント単位）で拒否してください。ホスト側の承認は、
`security=full` かつ `ask=off` を明示的に設定しない限り引き続き適用されます。

<div id="exec-approvals-companion-app-node-host">
  ## Exec の承認（コンパニオンアプリ / ノードホスト）
</div>

サンドボックス化されたエージェントでは、Gateway またはノードホスト上で `exec` を実行する前に、リクエスト単位で承認を必須にできます。
ポリシー、許可リスト、UI フローについては [Exec approvals](/ja/tools/exec-approvals) を参照してください。

承認が必要な場合、exec ツールはすぐに `status: "approval-pending"` と承認 ID を返します。承認（または拒否 / タイムアウト）されると、Gateway はシステムイベント（`Exec finished` / `Exec denied`）を発行します。コマンドが `tools.exec.approvalRunningNoticeMs` 経過後もまだ実行中の場合は、`Exec running` 通知が 1 回だけ発行されます。

<div id="allowlist-safe-bins">
  ## 許可リスト + セーフバイナリ
</div>

許可リストの適用は、**解決済みバイナリパスに対してのみ**行われます（basename だけでのマッチは行いません）。\
`security=allowlist` の場合、パイプライン内のすべてのセグメントが許可リスト対象かセーフバイナリである場合にのみ、シェルコマンドは自動的に許可されます。`;`、`&&`、`||` によるコマンドの連結やリダイレクトは、allowlist モードでは拒否されます。

<div id="examples">
  ## 例
</div>

フォアグラウンド：

```json
{"tool":"exec","command":"ls -la"}
```

背景＋アンケート:

```json
{"tool":"exec","command":"npm run build","yieldMs":1000}
{"tool":"process","action":"poll","sessionId":"<id>"}
```

キー送信 (tmuxスタイル):

```json
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["Enter"]}
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["C-c"]}
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["Up","Up","Enter"]}
```

送信（CR のみ）:

```json
{"tool":"process","action":"submit","sessionId":"<id>"}
```

貼り付け内容（デフォルトでは角括弧で囲まれます）:

```json
{"tool":"process","action":"paste","sessionId":"<id>","text":"line1\nline2\n"}
```

<div id="apply_patch-experimental">
  ## apply_patch（実験的）
</div>

`apply_patch` は、構造化された複数ファイル編集のための `exec` のサブツールです。
明示的に有効化する必要があります：

```json5
{
  tools: {
    exec: {
      applyPatch: { enabled: true, allowModels: ["gpt-5.2"] }
    }
  }
}
```

Notes:

* OpenAI／OpenAI Codex モデルでのみ利用可能です。
* ツールポリシーは引き続き有効です。`allow: ["exec"]` を指定すると、暗黙的に `apply_patch` も許可されます。
* 設定は `tools.exec.applyPatch` の下にあります。
