---
summary: "OpenClaw のサンドボックスの仕組み: モード、スコープ、ワークスペースアクセス、イメージ"
title: サンドボックス
read_when: "サンドボックスについての詳しい解説が必要な場合、または agents.defaults.sandbox をチューニングする必要がある場合。"
status: active
---

<div id="sandboxing">
  # サンドボックス化
</div>

OpenClaw は、影響範囲を減らすために **ツールを Docker コンテナ内で実行** できます。
これは **オプション機能** であり、設定（`agents.defaults.sandbox` または
`agents.list[].sandbox`）によって制御されます。サンドボックス化が `off` の場合、ツールはホスト上で実行されます。
Gateway はホスト上に留まり、サンドボックス化が有効な場合はツールの実行だけが分離されたサンドボックス内で行われます。

これは完全なセキュリティ境界ではありませんが、モデルが変なことをしでかした場合でも、ファイルシステムおよびプロセスへのアクセスを実質的に制限します。

<div id="what-gets-sandboxed">
  ## 何がサンドボックス化されるか
</div>

* ツール実行（`exec`、`read`、`write`、`edit`、`apply_patch`、`process` など）。
* オプションのサンドボックス化されたブラウザ（`agents.defaults.sandbox.browser`）。
  * 既定では、ブラウザツールが必要になったときにサンドボックスブラウザが自動起動します（CDP に到達可能であることを保証します）。
    `agents.defaults.sandbox.browser.autoStart` と `agents.defaults.sandbox.browser.autoStartTimeoutMs` で設定できます。
  * `agents.defaults.sandbox.browser.allowHostControl` により、サンドボックス化されたセッションからホスト側のブラウザを明示的に対象にできます。
  * オプションの許可リストにより、`target: "custom"` の利用を制御します: `allowedControlUrls`、`allowedControlHosts`、`allowedControlPorts`。

サンドボックス化されないもの:

* Gateway プロセス自体。
* ホスト上での実行を明示的に許可されたツール（例: `tools.elevated`）。
  * **Elevated exec はホスト上で実行され、サンドボックスをバイパスします。**
  * サンドボックス機能がオフの場合、`tools.elevated` は実行方法を変更しません（すでにホスト上で実行されています）。[Elevated Mode](/ja/tools/elevated) を参照してください。

<div id="modes">
  ## モード
</div>

`agents.defaults.sandbox.mode` は、サンドボックスが **いつ** 使用されるかを制御します:

* `"off"`: サンドボックスを使用しない。
* `"non-main"`: **メイン以外** のセッションのみサンドボックス内で実行する（通常のチャットをホスト上で実行したい場合のデフォルト）。
* `"all"`: すべてのセッションをサンドボックス内で実行する。

注意: `"non-main"` はエージェント ID ではなく、`session.mainKey`（デフォルトは `"main"`）に基づきます。
グループ／チャンネルのセッションはそれぞれ独自のキーを使用するため、メイン以外として扱われ、サンドボックス化されます。

<div id="scope">
  ## スコープ
</div>

`agents.defaults.sandbox.scope` は、**いくつのコンテナ**を作成するかを制御します：

* `"session"`（デフォルト）：セッションごとに 1 つのコンテナ。
* `"agent"`：エージェントごとに 1 つのコンテナ。
* `"shared"`：サンドボックス化されたすべてのセッションで共有される 1 つのコンテナ。

<div id="workspace-access">
  ## ワークスペースへのアクセス
</div>

`agents.defaults.sandbox.workspaceAccess` は、**サンドボックスからアクセスできる範囲** を制御します:

* `"none"` (デフォルト): ツールは `~/.openclaw/sandboxes` 配下のサンドボックス用ワークスペースのみを参照します。
* `"ro"`: エージェントのワークスペースを `/agent` に読み取り専用でマウントします（`write` / `edit` / `apply_patch` を無効化）。
* `"rw"`: エージェントのワークスペースを `/workspace` に読み書き可能でマウントします。

受信メディアは、アクティブなサンドボックスのワークスペース内（`media/inbound/*`）にコピーされます。
スキルに関する注意: `read` ツールはサンドボックスのルートを基準に動作します。`workspaceAccess: "none"` の場合、
OpenClaw は対象となるスキルをサンドボックスのワークスペース（`.../skills`）にミラーリングし、
サンドボックス内から読み取れるようにします。`"rw"` の場合、ワークスペース内のスキルは
`/workspace/skills` から読み取ることができます。

<div id="custom-bind-mounts">
  ## カスタム bind マウント
</div>

`agents.defaults.sandbox.docker.binds` は、追加のホストディレクトリをコンテナにマウントします。
形式: `host:container:mode`（例: `"/home/user/source:/source:rw"`）。

グローバルの bind とエージェントごとの bind は**マージ**されます（置き換えられません）。`scope: "shared"` の場合、エージェントごとの bind は無視されます。

例（読み取り専用のソース + Docker ソケット）:

```json5
{
  agents: {
    defaults: {
      sandbox: {
        docker: {
          binds: [
            "/home/user/source:/source:ro",
            "/var/run/docker.sock:/var/run/docker.sock"
          ]
        }
      }
    },
    list: [
      {
        id: "build",
        sandbox: {
          docker: {
            binds: ["/mnt/cache:/cache:rw"]
          }
        }
      }
    ]
  }
}
```

セキュリティ上の注意:

* バインドはサンドボックスのファイルシステムをバイパスします。ホスト側のパスを、設定したモード（`:ro` または `:rw`）でそのまま公開します。
* 機密性の高いマウント（例: `docker.sock`、シークレット、SSH キー）は、絶対に必要な場合を除き `:ro` にするべきです。
* ワークスペースへの読み取り専用アクセスだけが必要な場合は、`workspaceAccess: "ro"` と組み合わせてください。バインドのモードはそれとは独立しています。
* バインドがツールポリシーおよび昇格実行とどのように連携するかについては、[Sandbox vs Tool Policy vs Elevated](/ja/gateway/sandbox-vs-tool-policy-vs-elevated) を参照してください。

<div id="images-setup">
  ## イメージとセットアップ
</div>

デフォルトイメージ: `openclaw-sandbox:bookworm-slim`

最初に一度だけビルドします:

```bash
scripts/sandbox-setup.sh
```

注記: デフォルトイメージには Node は**含まれていません**。スキルで Node（または
他のランタイム）が必要な場合は、カスタムイメージを作成するか、
`sandbox.docker.setupCommand` を使ってインストールします（外向きネットワーク通信 + 書き込み可能な root ファイルシステム +
root ユーザーが必要）。

サンドボックス化されたブラウザーイメージ:

```bash
scripts/sandbox-browser-setup.sh
```

デフォルトでは、サンドボックスコンテナは**ネットワーク機能なし**で実行されます。
`agents.defaults.sandbox.docker.network` で変更できます。

Docker のインストール手順およびコンテナ化された Gateway については、次を参照してください:
[Docker](/ja/install/docker)

<div id="setupcommand-one-time-container-setup">
  ## setupCommand（一度きりのコンテナセットアップ）
</div>

`setupCommand` はサンドボックスコンテナが作成されたあとに **一度だけ** 実行されます（毎回の実行時ではありません）。
コンテナ内で `sh -lc` を通して実行されます。

パス:

* グローバル: `agents.defaults.sandbox.docker.setupCommand`
* エージェント単位: `agents.list[].sandbox.docker.setupCommand`

よくある落とし穴:

* デフォルトの `docker.network` は `"none"`（外部への通信なし）なので、パッケージのインストールは失敗します。
* `readOnlyRoot: true` は書き込みを禁止します。`readOnlyRoot: false` にするか、カスタムイメージを作成してください。
* パッケージのインストールには `user` が root である必要があります（`user` を省略するか、`user: "0:0"` を設定してください）。
* サンドボックス内での実行はホストの `process.env` を **継承しません**。スキル用の API キーには
  `agents.defaults.sandbox.docker.env`（またはカスタムイメージ）を使用してください。

<div id="tool-policy-escape-hatches">
  ## ツールポリシー + エスケープハッチ
</div>

ツールの allow/deny ポリシーは、サンドボックスのルールよりも先に適用されます。ツールが
グローバルまたはエージェント単位で deny されている場合、サンドボックス化しても再び利用可能にはなりません。

`tools.elevated` はホスト上で `exec` を実行するための明示的なエスケープハッチです。
`/exec` ディレクティブは認可された送信者にのみ適用され、セッション単位で維持されます。`exec` を
強制的に無効化するには、ツールポリシーで deny を使用します（[Sandbox vs Tool Policy vs Elevated](/ja/gateway/sandbox-vs-tool-policy-vs-elevated) を参照）。

デバッグ:

* `openclaw sandbox explain` を使って、有効なサンドボックスモード、ツールポリシー、および問題解消用の設定キーを確認します。
* 「なぜこれはブロックされているのか？」という思考モデルについては [Sandbox vs Tool Policy vs Elevated](/ja/gateway/sandbox-vs-tool-policy-vs-elevated) を参照してください。
  厳格にロックダウンした状態を維持してください。

<div id="multi-agent-overrides">
  ## マルチエージェントでのオーバーライド設定
</div>

各エージェントは、サンドボックスとツールを個別にオーバーライドできます：
`agents.list[].sandbox` と `agents.list[].tools`（およびサンドボックスのツールポリシーを指定する `agents.list[].tools.sandbox.tools`）。
優先順位については [Multi-Agent Sandbox &amp; Tools](/ja/multi-agent-sandbox-tools) を参照してください。

<div id="minimal-enable-example">
  ## 最小構成で有効化する例
</div>

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        scope: "session",
        workspaceAccess: "none"
      }
    }
  }
}
```

<div id="related-docs">
  ## 関連ドキュメント
</div>

* [サンドボックスの設定](/ja/gateway/configuration#agentsdefaults-sandbox)
* [マルチエージェント用サンドボックスとツール](/ja/multi-agent-sandbox-tools)
* [セキュリティ](/ja/gateway/security)