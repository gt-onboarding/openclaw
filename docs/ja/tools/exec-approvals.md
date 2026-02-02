---
title: Exec 承認
summary: "Exec 承認、許可リスト、およびサンドボックスエスケーププロンプト"
read_when:
  - Exec 承認または許可リストを設定するとき
  - macOS アプリで Exec 承認の UX を実装するとき
  - サンドボックスエスケーププロンプトとその影響を検討するとき
---

<div id="exec-approvals">
  # Exec approvals
</div>

Exec approvals は、サンドボックス化されたエージェントが実際のホスト（`gateway` または `node`）上で
コマンドを実行できるようにするための、**コンパニオンアプリ／ノードホスト側のガードレール機構** です。
安全装置のようなものだと考えてください。
ポリシー + 許可リスト + （オプションの）ユーザー承認がすべて揃ったときにのみ、コマンドが許可されます。
Exec approvals は、ツールポリシーや昇格ゲーティング（`elevated` が `full` に設定されていて承認をスキップする場合を除く）に
**追加で** 適用されます。
有効なポリシーは `tools.exec.*` と approvals のデフォルト値のうち **より厳しい方** です。
approvals のフィールドが省略された場合、そのフィールドには `tools.exec` の値が使われます。

コンパニオンアプリの UI が **利用できない** 場合、プロンプトを必要とするリクエストはすべて
**ask フォールバック**（デフォルト: deny）によって処理されます。

<div id="where-it-applies">
  ## 適用範囲
</div>

Exec 承認は、各実行ホスト上でローカルに適用されます:

* **gateway ホスト** → Gateway マシン上の `openclaw` プロセス
* **ノードホスト** → ノード ランナー（macOS コンパニオンアプリまたはヘッドレスなノードホスト）

macOS における役割分担:

* **ノードホストサービス** がローカル IPC 経由で **macOS アプリ** に `system.run` を転送します。
* **macOS アプリ** が承認を適用し、UI コンテキストでコマンドを実行します。

<div id="settings-and-storage">
  ## 設定と保存先
</div>

承認は、実行ホスト上のローカル JSON ファイルに保存されます:

`~/.openclaw/exec-approvals.json`

スキーマ例：

```json
{
  "version": 1,
  "socket": {
    "path": "~/.openclaw/exec-approvals.sock",
    "token": "base64url-token"
  },
  "defaults": {
    "security": "deny",
    "ask": "on-miss",
    "askFallback": "deny",
    "autoAllowSkills": false
  },
  "agents": {
    "main": {
      "security": "allowlist",
      "ask": "on-miss",
      "askFallback": "deny",
      "autoAllowSkills": true,
      "allowlist": [
        {
          "id": "B0C8C0B3-2C2D-4F8A-9A3C-5A4B3C2D1E0F",
          "pattern": "~/Projects/**/bin/rg",
          "lastUsedAt": 1737150000000,
          "lastUsedCommand": "rg -n TODO",
          "lastResolvedPath": "/Users/user/Projects/.../bin/rg"
        }
      ]
    }
  }
}
```

<div id="policy-knobs">
  ## ポリシー設定項目
</div>

<div id="security-execsecurity">
  ### セキュリティ (`exec.security`)
</div>

* **deny**: すべてのホスト上の exec リクエストをブロックします。
* **allowlist**: 許可リストに登録されたコマンドのみを許可します。
* **full**: すべてを許可します（`elevated` と同等）。

<div id="ask-execask">
  ### Ask (`exec.ask`)
</div>

* **off**: 一切プロンプトを出しません。
* **on-miss**: 許可リストに一致しない場合にのみプロンプトを出します。
* **always**: すべてのコマンドでプロンプトを出します。

<div id="ask-fallback-askfallback">
  ### Ask fallback (`askFallback`)
</div>

プロンプトが必要だが UI にアクセスできない場合、fallback が次のいずれかを決定します:

* **deny**: ブロックする。
* **allowlist**: 許可リストに一致する場合のみ許可する。
* **full**: 許可する。

<div id="allowlist-per-agent">
  ## 許可リスト（エージェント単位）
</div>

許可リストは **エージェント単位** です。複数のエージェントが存在する場合は、macOS アプリで編集対象のエージェントを切り替えてください。パターンは **大文字・小文字を区別しない glob マッチ** です。
パターンは **バイナリのパス** に解決される必要があります（ベース名のみのエントリは無視されます）。
レガシーな `agents.default` エントリは、ロード時に `agents.main` へ移行されます。

例:

* `~/Projects/**/bin/bird`
* `~/.local/bin/*`
* `/opt/homebrew/bin/rg`

各許可リストエントリでは、以下を追跡します:

* **id** UI 上の識別用に使用される安定した UUID（任意）
* **last used** 最終使用時刻
* **last used command** 最後に使用したコマンド
* **last resolved path** 最後に解決されたパス

<div id="auto-allow-skill-clis">
  ## スキル CLI の自動許可
</div>

**Auto-allow skill CLIs** を有効にすると、既知のスキルから参照される実行ファイルは、
ノード（macOS ノードまたはヘッドレスノードホスト）上で許可リスト登録済みとして扱われます。これは
Gateway の RPC 経由で `skills.bins` を使用してスキル bin の一覧を取得します。許可リストをすべて手動で厳密に管理したい場合は、これを無効にしてください。

<div id="safe-bins-stdin-only">
  ## Safe bins (stdin-only)
</div>

`tools.exec.safeBins` は、（例えば `jq` のような）**標準入力専用**バイナリの小さなリストを定義します。
これらは明示的な許可リストエントリがなくても、許可リストモードで実行できます。Safe bins は
位置引数として渡されたファイルやパスのようなトークンを拒否するため、入力ストリームに対してのみ操作できます。
許可リストモードでは、シェルのコマンド連結やリダイレクトは自動的には許可されません。

シェル連結（`&&`、`||`、`;`）は、すべてのトップレベルセグメントが許可リスト（Safe bins や skill の自動許可を含む）
を満たしている場合にのみ許可されます。リダイレクトは許可リストモードでは引き続きサポートされません。

デフォルトの Safe bins: `jq`、`grep`、`cut`、`sort`、`uniq`、`head`、`tail`、`tr`、`wc`。

<div id="control-ui-editing">
  ## Control UI での編集
</div>

**Control UI → Nodes → Exec approvals** カードを使って、デフォルト設定、エージェントごとの上書き設定、許可リストを編集します。スコープ（Defaults またはエージェント）を選択し、ポリシーを調整し、許可リストのパターンを追加／削除してから **Save** します。UI にはパターンごとの **last used** メタデータが表示されるため、リストを整理しやすくなります。

ターゲットセレクターでは **Gateway**（ローカル承認）か **Node** を選択します。ノードは `system.execApprovals.get/set` をアドバタイズしている必要があります（macOS アプリまたはヘッドレスのノードホスト）。ノードがまだ exec approvals をアドバタイズしていない場合は、そのローカルの `~/.openclaw/exec-approvals.json` を直接編集してください。

CLI: `openclaw approvals` は Gateway またはノード向けの編集をサポートします（[Approvals CLI](/ja/cli/approvals) を参照）。

<div id="approval-flow">
  ## 承認フロー
</div>

プロンプトが必要な場合、Gateway は `exec.approval.requested` をオペレータークライアントにブロードキャストします。
Control UI と macOS アプリは `exec.approval.resolve` を通じて承認を処理し、その後 Gateway が
承認されたリクエストをノードホストに転送します。

承認が必要な場合、exec ツールは直ちに承認 ID を返します。その ID を使用して、
後から発生するシステムイベント（`Exec finished` / `Exec denied`）と対応付けます。タイムアウトまでに
決定が届かない場合、そのリクエストは承認タイムアウトとして扱われ、拒否理由として扱われます。

確認ダイアログには以下が含まれます:

* command + args
* cwd
* agent id
* resolved executable path
* host + policy metadata

操作:

* **Allow once** → 今回のみ実行
* **Always allow** → 許可リストに追加して実行
* **Deny** → ブロック

<div id="approval-forwarding-to-chat-channels">
  ## チャットチャンネルへの承認転送
</div>

exec の承認プロンプトを任意のチャットチャンネル（プラグインチャンネルを含む）に転送し、`/approve` で承認できます。これは通常のアウトバウンド配信パイプラインを使用します。

設定:

```json5
{
  approvals: {
    exec: {
      enabled: true,
      mode: "session", // "session" | "targets" | "both"
      agentFilter: ["main"],
      sessionFilter: ["discord"], // substring or regex
      targets: [
        { channel: "slack", to: "U12345678" },
        { channel: "telegram", to: "123456789" }
      ]
    }
  }
}
```

チャットで返信

```
/approve <id> allow-once
/approve <id> allow-always
/approve <id> deny
```

<div id="macos-ipc-flow">
  ### macOS における IPC フロー
</div>

```
Gateway -> Node Service (WS)
                 |  IPC (UDS + token + HMAC + TTL)
                 v
             Mac App (UI + approvals + system.run)
```

セキュリティに関する注意事項:

* Unix ソケットモードは `0600`、トークンは `exec-approvals.json` に保存されます。
* 同一 UID のピアチェックを行います。
* チャレンジ／レスポンス方式（nonce + HMAC トークン + リクエストハッシュ）＋短い TTL。

<div id="system-events">
  ## システムイベント
</div>

Exec のライフサイクルはシステムメッセージとして通知されます:

* `Exec running`（コマンドの実行時間が実行通知のしきい値を超えた場合のみ）
* `Exec finished`
* `Exec denied`

これらは、ノードがイベントを報告した後にエージェントのセッションへ送信されます。
Gateway ホスト側の exec 承認も、コマンドが終了したとき（およびオプションで、しきい値を超えて長時間実行されているとき）に同じライフサイクルイベントを発行します。
承認が必須の exec では、これらのメッセージ内で承認 ID を `runId` として再利用し、容易に対応付けできるようにします。

<div id="implications">
  ## 含意
</div>

* **full** は強力なので、可能な限り許可リストを優先してください。
* **ask** は高速な承認を維持しつつ、常に状況を把握し続けられるようにします。
* エージェントごとの許可リストによって、あるエージェントの承認が別のエージェントに波及することを防ぎます。
* 承認は、**認可済み送信者** からのホスト exec リクエストにのみ適用されます。未認可の送信者は `/exec` を発行できません。
* `/exec security=full` は、認可されたオペレーター向けのセッションレベルの利便機能であり、設計上、承認をスキップします。
  ホスト exec を完全にブロックするには、approvals の security を `deny` に設定するか、ツールポリシーで `exec` ツールを拒否してください。

関連:

* [Exec ツール](/ja/tools/exec)
* [Elevated モード](/ja/tools/elevated)
* [スキル](/ja/tools/skills)