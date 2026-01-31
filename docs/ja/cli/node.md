---
title: ノード
summary: "`openclaw node` 用 CLI リファレンス（ヘッドレスノードホスト）"
read_when:
  - ヘッドレスノードホストを実行するとき
  - system.run 用に macOS 以外のノードをペアリングするとき
---

<div id="openclaw-node">
  # `openclaw node`
</div>

Gateway の WebSocket に接続し、このマシン上の `system.run` / `system.which` を公開する
**ヘッドレスなノードホスト** を起動します。

<div id="why-use-a-node-host">
  ## なぜノードホストを使うのか？
</div>

ネットワーク内の他のマシン上でエージェントに**コマンドを実行させたい**が、
そこにフル機能の macOS コンパニオンアプリをインストールしたくない場合に、
ノードホストを使用します。

代表的なユースケース:

* リモートの Linux/Windows マシン（ビルドサーバ、ラボ用マシン、NAS）でコマンドを実行する。
* 実行環境自体は Gateway 上で**サンドボックス化**したまま、承認済みの実行だけを他のホストに委譲する。
* オートメーションや CI ノード向けに、軽量なヘッドレス実行ターゲットを提供する。

実行は引き続きノードホスト上の **exec approvals** と、エージェントごとの許可リストによって保護されるため、
コマンドアクセスを明示的かつ限定されたスコープ内に保ったまま運用できます。

<div id="browser-proxy-zero-config">
  ## ブラウザプロキシ（ゼロコンフィグ）
</div>

`browser.enabled` がノードで無効化されていない場合、ノードホストは自動的にブラウザプロキシを公開します。これにより、そのノード上でエージェントが追加の設定なしにブラウザ自動化を利用できるようになります。

必要に応じて、ノード側でこれを無効化してください：

```json5
{
  nodeHost: {
    browserProxy: {
      enabled: false
    }
  }
}
```

<div id="run-foreground">
  ## フォアグラウンドで実行
</div>

```bash
openclaw node run --host <gateway-host> --port 18789
```

Options:

* `--host &lt;host&gt;`: Gateway WebSocket ホスト（デフォルト: `127.0.0.1`）
* `--port &lt;port&gt;`: Gateway WebSocket ポート（デフォルト: `18789`）
* `--tls`: Gateway 接続に TLS を使用する
* `--tls-fingerprint &lt;sha256&gt;`: 想定される TLS 証明書フィンガープリント（sha256）
* `--node-id &lt;id&gt;`: ノード ID を上書きする（ペアリングトークンを消去）
* `--display-name &lt;name&gt;`: ノード表示名を上書きする

<div id="service-background">
  ## サービス（バックグラウンド）
</div>

ヘッドレスのノードホストをユーザーサービスとしてインストールします。

```bash
openclaw node install --host <gateway-host> --port 18789
```

Options:

* `--host <host>`: Gateway の WebSocket ホスト（デフォルト: `127.0.0.1`）
* `--port <port>`: Gateway の WebSocket ポート（デフォルト: `18789`）
* `--tls`: Gateway への接続に TLS を使用する
* `--tls-fingerprint <sha256>`: 想定される TLS 証明書のフィンガープリント（sha256）
* `--node-id <id>`: ノード ID を上書きする（ペアリングトークンを消去）
* `--display-name <name>`: ノードの表示名を上書きする
* `--runtime <runtime>`: サービスのランタイム環境（`node` または `bun`）
* `--force`: すでにインストール済みの場合でも再インストール／上書きする

サービスの管理:

```bash
openclaw node status
openclaw node stop
openclaw node restart
openclaw node uninstall
```

フォアグラウンドでノードホストを実行する場合は（サービスとしてではなく）、`openclaw node run` を使用します。

サービス関連コマンドは、機械可読な出力を得るために `--json` を受け付けます。

<div id="pairing">
  ## ペアリング
</div>

最初の接続時に、Gateway 上に保留中のノードペアリング要求が作成されます。
次の方法で承認します:

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
```

ノードホストは、ノード ID、トークン、表示名、および Gateway への接続情報を
`~/.openclaw/node.json` に保存します。

<div id="exec-approvals">
  ## Exec 承認
</div>

`system.run` はローカルの Exec 承認によって保護されています:

* `~/.openclaw/exec-approvals.json`
* [Exec approvals](/ja/tools/exec-approvals)
* `openclaw approvals --node <id|name|ip>` (Gateway から編集する)