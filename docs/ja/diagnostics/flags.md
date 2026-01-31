---
title: フラグ
summary: "特定のデバッグログ取得のための診断用フラグ"
read_when:
  - グローバルなログレベルを引き上げずに、特定のデバッグログが必要な場合
  - サポート対応用にサブシステム固有のログを取得する必要がある場合
---

<div id="diagnostics-flags">
  # 診断フラグ
</div>

診断フラグを使うと、システム全体で詳細ログを有効にすることなく、特定の領域や機能のデバッグログだけを有効化できます。フラグはオプトイン方式であり、サブシステム側でチェックされない限り効果はありません。

<div id="how-it-works">
  ## 動作概要
</div>

* フラグは文字列です（大文字・小文字は区別されません）。
* フラグは設定ファイル、または環境変数による上書きで有効化できます。
* ワイルドカードも利用できます：
  * `telegram.*` は `telegram.http` にマッチします
  * `*` はすべてのフラグを有効にします

<div id="enable-via-config">
  ## 設定で有効化する
</div>

```json
{
  "diagnostics": {
    "flags": ["telegram.http"]
  }
}
```

複数のフラグ:

```json
{
  "diagnostics": {
    "flags": ["telegram.http", "gateway.*"]
  }
}
```

フラグを変更後は Gateway を再起動してください。

<div id="env-override-one-off">
  ## 環境変数による一時的な上書き
</div>

```bash
OPENCLAW_DIAGNOSTICS=telegram.http,telegram.payload
```

すべてのフラグを無効にする：

```bash
OPENCLAW_DIAGNOSTICS=0
```

<div id="where-logs-go">
  ## ログの保存先
</div>

フラグは標準の診断ログファイルにログを出力します。既定では、次のファイルになります：

```
/tmp/openclaw/openclaw-YYYY-MM-DD.log
```

`logging.file` を設定している場合は、そのパスを代わりに使用してください。ログは JSONL 形式（1 行ごとに 1 つの JSON オブジェクト）です。`logging.redactSensitive` に基づき、マスキングは引き続き適用されます。

<div id="extract-logs">
  ## ログを抽出する
</div>

最新のログファイルを選択します:

```bash
ls -t /tmp/openclaw/openclaw-*.log | head -n 1
```

Telegram HTTP 診断用フィルター：

```bash
rg "telegram http error" /tmp/openclaw/openclaw-*.log
```

または、再現させながら tail する:

```bash
tail -f /tmp/openclaw/openclaw-$(date +%F).log | rg "telegram http error"
```

リモートの Gateway でも `openclaw logs --follow` を使用できます（[/cli/logs](/ja/cli/logs) を参照）。

<div id="notes">
  ## 注意事項
</div>

* `logging.level` が `warn` より高く設定されている場合、これらのログは出力されない可能性があります。デフォルト値の `info` のままで問題ありません。
* フラグは有効のままにしておいても安全です。対象のサブシステムに関するログの出力量にのみ影響します。
* ログの出力先、レベル、秘匿化設定を変更するには [/logging](/ja/logging) を使用します。