---
title: プラグイン
summary: "`openclaw plugins` の CLI リファレンス（一覧表示、インストール、有効化/無効化、doctor）"
read_when:
  - Gateway プロセス内で動作するプラグインをインストールまたは管理したい場合
  - プラグインの読み込みエラーをデバッグしたい場合
---

<div id="openclaw-plugins">
  # `openclaw plugins`
</div>

Gateway のプラグイン／拡張機能を管理します（インプロセスでロードされます）。

関連:

* プラグインシステム: [Plugins](/ja/plugin)
* プラグインマニフェストおよびスキーマ: [Plugin manifest](/ja/plugins/manifest)
* セキュリティ強化: [Security](/ja/gateway/security)

<div id="commands">
  ## コマンド
</div>

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins doctor
openclaw plugins update <id>
openclaw plugins update --all
```

同梱されているプラグインは OpenClaw にバンドルされていますが、初期状態では無効になっています。`plugins enable` を使用して有効化してください。

すべてのプラグインは、インラインの JSON Schema（空でもよい `configSchema`）を含む `openclaw.plugin.json` ファイルを同梱している必要があります。マニフェストまたはスキーマが存在しない、もしくは不正な場合、プラグインはロードされず、設定の検証も失敗します。

<div id="install">
  ### インストール
</div>

```bash
openclaw plugins install <path-or-spec>
```

セキュリティ上の注意: プラグインのインストールはコード実行と同様に扱ってください。バージョンを固定して利用することを推奨します。

サポートされているアーカイブ形式: `.zip`, `.tgz`, `.tar.gz`, `.tar`。

ローカルディレクトリのコピーを避けるには `--link` を使用します（`plugins.load.paths` にパスが追加されます）:

```bash
openclaw plugins install -l ./my-plugin
```

<div id="update">
  ### 更新
</div>

```bash
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins update <id> --dry-run
```

アップデートは、npm からインストールされ、`plugins.installs` で追跡されているプラグインにのみ適用されます。
