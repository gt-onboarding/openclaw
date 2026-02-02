---
title: 子プロセス
summary: "macOS における Gateway のライフサイクル (launchd)"
read_when:
  - macOS アプリを Gateway のライフサイクルと統合する場合
---

<div id="gateway-lifecycle-on-macos">
  # macOS における Gateway のライフサイクル
</div>

macOS アプリは、デフォルトでは **launchd 経由で Gateway を管理** し、Gateway を子プロセスとして起動しません。まず、設定されたポート上で既に動作中の Gateway への接続を試み、到達可能なものがなければ、外部の `openclaw` CLI（組み込みランタイムなし）を通じて launchd サービスを有効化します。これにより、ログイン時の自動起動とクラッシュ時の自動再起動を確実に行えます。

子プロセスモード（Gateway がアプリから直接起動されるモード）は、現在は **使用されていません**。UI との連携をより密にしたい場合は、ターミナルで Gateway を手動実行してください。

<div id="default-behavior-launchd">
  ## デフォルトの動作（launchd）
</div>

* アプリはユーザー単位の LaunchAgent を `bot.molt.gateway` というラベルでインストールします
  （`--profile`/`OPENCLAW_PROFILE` を使用している場合は `bot.molt.<profile>`、従来の `com.openclaw.*` もサポートされます）。
* Local モードが有効な場合、アプリは LaunchAgent がロードされていることを確認し、
  必要に応じて Gateway を起動します。
* ログは launchd の Gateway ログパスに書き込まれます（Debug Settings で確認可能）。

よく使うコマンド：

```bash
launchctl kickstart -k gui/$UID/bot.molt.gateway
launchctl bootout gui/$UID/bot.molt.gateway
```

名前付きプロファイルで実行する場合は、ラベルを `bot.molt.<profile>` に置き換えてください。


<div id="unsigned-dev-builds">
  ## 署名なしの開発ビルド
</div>

`scripts/restart-mac.sh --no-sign` は、署名鍵がない場合のローカルでの高速ビルド用です。未署名の relay バイナリを launchd が参照しないようにするため、次を実行します:

* `~/.openclaw/disable-launchagent` を書き込みます。

`scripts/restart-mac.sh` を署名付きで実行した場合、このマーカーが存在していれば、この上書き設定をクリアします。手動でリセットするには:

```bash
rm ~/.openclaw/disable-launchagent
```


<div id="attach-only-mode">
  ## アタッチ専用モード
</div>

macOS アプリに **一切 launchd をインストールまたは管理させない** ようにするには、
`--attach-only`（または `--no-launchd`）を指定して起動します。これにより
`~/.openclaw/disable-launchagent` が設定され、アプリは既に起動している Gateway にのみアタッチする
動作になります。同じ挙動は Debug Settings からも切り替えられます。

<div id="remote-mode">
  ## リモートモード
</div>

リモートモードではローカルの Gateway は起動しません。アプリはリモートホストへの SSH トンネルを張り、そのトンネル経由で接続します。

<div id="why-we-prefer-launchd">
  ## なぜ launchd を優先するのか
</div>

- ログイン時に自動起動できる。
- 再起動／KeepAlive の動作仕様が標準で備わっている。
- ログ出力とプロセス監視の挙動が予測しやすい。

もし真の子プロセスとして動作するモードが再び必要になった場合は、
開発用途専用の明示的な別モードとして文書化すべきである。