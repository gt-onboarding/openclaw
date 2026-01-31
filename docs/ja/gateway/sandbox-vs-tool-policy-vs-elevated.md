---
title: サンドボックス vs ツールポリシー vs Elevated
summary: "なぜツールがブロックされるのか: サンドボックスのランタイム、ツールの許可/拒否ポリシー、Elevated 実行ゲート"
read_when: "\"sandbox jail\" に引っかかった、またはツール/Elevated の拒否メッセージが出て、どの設定キーを変更すればよいか正確に知りたいとき。"
status: active
---

<div id="sandbox-vs-tool-policy-vs-elevated">
  # サンドボックス vs ツールポリシー vs Elevated
</div>

OpenClaw には、関連しているが異なる 3 種類の制御があります:

1. **サンドボックス**（`agents.defaults.sandbox.*` / `agents.list[].sandbox.*`）は、ツールを **どこで実行するか**（Docker かホストか）を決定します。
2. **ツールポリシー**（`tools.*`、`tools.sandbox.tools.*`、`agents.list[].tools.*`）は、**どのツールを利用可能/許可とするか** を決定します。
3. **Elevated**（`tools.elevated.*`、`agents.list[].tools.elevated.*`）は、サンドボックス化されているときにホスト上で実行するための、**実行専用のエスケープハッチ** です。

<div id="quick-debug">
  ## クイックデバッグ
</div>

Inspector を使って、OpenClaw が*実際に*何をしているのかを確認してください：

```bash
openclaw sandbox explain
openclaw sandbox explain --session agent:main:main
openclaw sandbox explain --agent work
openclaw sandbox explain --json
```

これは次の内容を出力します:

* 有効なサンドボックスのモード／スコープ／ワークスペースアクセス
* セッションが現在サンドボックス内で実行されているかどうか（main か non-main か）
* 有効なサンドボックスのツール allow/deny（およびそれが agent / global / default のどこ由来か）
* elevated ゲートと fix-it 用キー・パス

<div id="sandbox-where-tools-run">
  ## サンドボックス: ツールが実行される場所
</div>

サンドボックス化は `agents.defaults.sandbox.mode` で制御されます：

* `"off"`: すべてがホスト上で実行されます。
* `"non-main"`: メイン以外のセッションのみサンドボックス化されます（グループ／チャンネルでよくある「意外な挙動」）。
* `"all"`: すべてがサンドボックス化されます。

スコープ、ワークスペースのマウント、イメージなどの全パターンについては [Sandboxing](/ja/gateway/sandboxing) を参照してください。

<div id="bind-mounts-security-quick-check">
  ### バインドマウント（セキュリティ簡易チェック）
</div>

* `docker.binds` はサンドボックスのファイルシステムを *貫通* します。マウントしたものは、指定したモード（`:ro` または `:rw`）でコンテナ内から見えるようになります。
* モードを省略すると、デフォルトは読み書き可能（read-write）です。ソースコードやシークレットには `:ro` を推奨します。
* `scope: "shared"` の場合、エージェントごとのバインドは無視されます（グローバルなバインドのみが適用されます）。
* `/var/run/docker.sock` をバインドすると、実質的にホストの制御権をサンドボックスに渡すことになります。意図して行う場合にのみ実施してください。
* ワークスペースアクセス（`workspaceAccess: "ro"` / `"rw"`）は、バインドのモードとは独立しています。

<div id="tool-policy-which-tools-existare-callable">
  ## ツールポリシー: どのツールが存在し/呼び出し可能か
</div>

重要なのは次のレイヤーです:

* **ツールプロファイル**: `tools.profile` および `agents.list[].tools.profile`（ベースとなる許可リスト）
* **プロバイダーツールプロファイル**: `tools.byProvider[provider].profile` および `agents.list[].tools.byProvider[provider].profile`
* **グローバル/エージェント単位のツールポリシー**: `tools.allow`/`tools.deny` および `agents.list[].tools.allow`/`agents.list[].tools.deny`
* **プロバイダーツールポリシー**: `tools.byProvider[provider].allow/deny` および `agents.list[].tools.byProvider[provider].allow/deny`
* **サンドボックスツールポリシー**（サンドボックス化されている場合のみ適用）: `tools.sandbox.tools.allow`/`tools.sandbox.tools.deny` および `agents.list[].tools.sandbox.tools.*`

原則:

* 常に `deny` が優先されます。
* `allow` が空でない場合、それ以外はすべてブロックとして扱われます。
* ツールポリシーが最終的な制限ポイントになります。`/exec` は `exec` ツールに対する `deny` を上書きできません。
* `/exec` が変更するのは、認可された送信者に対するセッションのデフォルト設定のみであり、ツールアクセス権を付与するものではありません。
  プロバイダーのツールキーには、`provider`（例: `google-antigravity`）または `provider/model`（例: `openai/gpt-5.2`）のいずれかを指定できます。

<div id="tool-groups-shorthands">
  ### ツールグループ（省略記法）
</div>

ツールポリシー（グローバル、エージェント、サンドボックス）は、`group:*` というエントリを指定すると、それが複数のツールに展開されるようになっています。

```json5
{
  tools: {
    sandbox: {
      tools: {
        allow: ["group:runtime", "group:fs", "group:sessions", "group:memory"]
      }
    }
  }
}
```

利用可能なグループ:

* `group:runtime`: `exec`, `bash`, `process`
* `group:fs`: `read`, `write`, `edit`, `apply_patch`
* `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
* `group:memory`: `memory_search`, `memory_get`
* `group:ui`: `browser`, `canvas`
* `group:automation`: `cron`, `gateway`
* `group:messaging`: `message`
* `group:nodes`: `nodes`
* `group:openclaw`: すべての組み込み OpenClaw ツール（プロバイダーのプラグインを除く）

<div id="elevated-exec-only-run-on-host">
  ## Elevated: exec専用の「ホスト上で実行」
</div>

Elevated は追加のツールを付与**しません**。影響するのは `exec` のみです。

* サンドボックス内であれば、`/elevated on`（または `elevated: true` 付きの `exec`）はホスト上で実行します（承認が必要な場合があります）。
* セッション内の exec 承認をスキップするには `/elevated full` を使用します。
* すでにホスト上で直接実行している場合、elevated は実質的に何もしません（依然としてゲートされています）。
* Elevated は**スキルスコープではなく**、ツールの allow/deny を上書きもしません。
* `/exec` は elevated とは別物です。認可された送信者ごとに、セッション単位の exec デフォルト値だけを調整します。

ゲート:

* 有効化: `tools.elevated.enabled`（および必要に応じて `agents.list[].tools.elevated.enabled`）
* 送信者の許可リスト: `tools.elevated.allowFrom.<provider>`（および必要に応じて `agents.list[].tools.elevated.allowFrom.<provider>`）

[Elevated モード](/ja/tools/elevated) を参照してください。

<div id="common-sandbox-jail-fixes">
  ## よくある「サンドボックス jail」の解消方法
</div>

<div id="tool-x-blocked-by-sandbox-tool-policy">
  ### 「ツール X がサンドボックスのツールポリシーでブロックされています」
</div>

修正用キー（いずれかを選択）:

* サンドボックスを無効化する: `agents.defaults.sandbox.mode=off`（またはエージェント単位で `agents.list[].sandbox.mode=off`）
* サンドボックス内でツールを許可する:
  * `tools.sandbox.tools.deny` から削除（またはエージェント単位で `agents.list[].tools.sandbox.tools.deny` から削除）
  * もしくは `tools.sandbox.tools.allow` に追加（またはエージェント単位の allow に追加）

<div id="i-thought-this-was-main-why-is-it-sandboxed">
  ### 「ここはmainのはずなのに、なぜサンドボックスで動いているのですか？」
</div>

`"non-main"` モードでは、グループ／チャンネルキーは *mainではありません*。`sandbox explain` に表示される main セッションキーを使用するか、モードを `"off"` に切り替えてください。