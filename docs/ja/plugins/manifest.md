---
title: マニフェスト
summary: "プラグインマニフェストおよび JSON スキーマ要件（設定の厳密なバリデーション）"
read_when:
  - OpenClaw 用のプラグインを開発している
  - プラグインの設定スキーマを提供する必要がある、またはプラグインのバリデーションエラーをデバッグする必要がある
---

<div id="plugin-manifest-openclawpluginjson">
  # プラグインマニフェスト (openclaw.plugin.json)
</div>

すべてのプラグインは、**プラグインのルート** に `openclaw.plugin.json` ファイルを必ず配置する **必要があります**。
OpenClaw はこのマニフェストを使って、**プラグインコードを実行せずに** 設定を検証します。
マニフェストが存在しない、または不正な場合はプラグインエラーとして扱われ、設定検証は実行されません。

プラグインシステム全体のガイドについては [Plugins](/ja/plugin) を参照してください。

<div id="required-fields">
  ## 必須項目
</div>

```json
{
  "id": "voice-call",
  "configSchema": {
    "type": "object",
    "additionalProperties": false,
    "properties": {}
  }
}
```

必須キー:

* `id` (string): 正規のプラグイン ID。
* `configSchema` (object): プラグイン設定用の JSON Schema（インライン）。

任意キー:

* `kind` (string): プラグインの種類（例: `"memory"`）。
* `channels` (array): このプラグインによって登録されるチャネル ID（例: `["matrix"]`）。
* `providers` (array): このプラグインによって登録されるプロバイダー ID。
* `skills` (array): 読み込むスキル用ディレクトリ（プラグインルートからの相対パス）。
* `name` (string): プラグインの表示名。
* `description` (string): プラグインの簡潔な概要。
* `uiHints` (object): UI レンダリング用の設定フィールドのラベル／プレースホルダー／機微情報用フラグ。
* `version` (string): プラグインのバージョン（情報目的）。

<div id="json-schema-requirements">
  ## JSON Schema の要件
</div>

* **すべてのプラグインは、たとえ設定を受け付けない場合でも JSON Schema を含める必要があります。**
* 空のスキーマも許容されます（例: `{ "type": "object", "additionalProperties": false }`）。
* スキーマは、実行時ではなく設定の読み取り/書き込み時に検証されます。

<div id="validation-behavior">
  ## 検証時の挙動
</div>

* 未知の `channels.*` キーは、そのチャネル ID がプラグインマニフェストで宣言されている場合を除き、**エラー** となります。
* `plugins.entries.<id>`、`plugins.allow`、`plugins.deny`、および `plugins.slots.*` は、**検出可能な** プラグイン ID を参照している必要があります。未知の ID は **エラー** となります。
* プラグインがインストール済みでも、マニフェストまたはスキーマが壊れている、もしくは存在しない場合は、検証が失敗し、Doctor によってプラグインエラーが報告されます。
* プラグインの設定が存在していても、そのプラグインが**無効化**されている場合、設定は保持されますが、Doctor およびログに **警告** が出力されます。

<div id="notes">
  ## 注意事項
</div>

* マニフェストは**すべてのプラグインに必須**であり、ローカルファイルシステムからのロードも含みます。
* ランタイムはプラグインモジュールを別途ロードします。マニフェストは検出 + 検証のみを目的としています。
* プラグインがネイティブモジュールに依存する場合、ビルド手順とパッケージマネージャーの許可リスト要件（例: pnpm `allow-build-scripts`
  * `pnpm rebuild <package>`）を文書化してください。