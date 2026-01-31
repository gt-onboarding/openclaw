---
title: Cron Add の強化
summary: "cron.add の入力処理を強化し、スキーマを整合させ、cron UI とエージェント向けツールを改善する"
owner: "openclaw"
status: "完了"
last_updated: "2026-01-05"
---

<div id="cron-add-hardening-schema-alignment">
  # Cron 追加処理のセキュリティ強化とスキーマ整合
</div>

<div id="context">
  ## コンテキスト
</div>

最近の Gateway のログで、無効なパラメーター (`sessionTarget`、`wakeMode`、`payload` の欠落や、不正な形式の `schedule`) による `cron.add` の失敗が繰り返し発生していることが確認されています。これは、少なくとも 1 つのクライアント (おそらくエージェントのツール呼び出しパス) が、ラップされた、または部分的にしか指定されていないジョブのペイロードを送信していることを示しています。別途、TypeScript の cron プロバイダー enum、Gateway スキーマ、CLI フラグ、UI フォーム型の間で不整合が発生しており、さらに `cron.status` についても、UI は `jobCount` を期待している一方で Gateway は `jobs` を返すという UI 上の不一致があります。

<div id="goals">
  ## 目標
</div>

* 一般的なラッパーペイロードを正規化し、不足している `kind` フィールドを推論することで、`cron.add` の INVALID&#95;REQUEST スパムを抑止する。
* Gateway スキーマ、cron 種別、CLI ドキュメント、UI フォーム間で cron プロバイダーリストを整合させる。
* LLM が正しいジョブペイロードを生成できるよう、エージェント用の cron ツールスキーマを明示的にする。
* Control UI の cron ステータスにおけるジョブ数の表示を修正する。
* 正規化とツールの挙動をカバーするテストを追加する。

<div id="non-goals">
  ## 非目標
</div>

* cron のスケジュールのセマンティクスやジョブ実行時の挙動を変更すること。
* 新しいスケジュール種別を追加したり、cron 式のパースを拡張したりすること。
* 必要なフィールド修正の範囲を超えて、cron の UI/UX を抜本的に刷新すること。

<div id="findings-current-gaps">
  ## 調査結果（現在のギャップ）
</div>

* Gateway の `CronPayloadSchema` では `signal` と `imessage` が除外されている一方で、TypeScript の型定義には含まれている。
* Control UI の CronStatus は `jobCount` を期待しているが、Gateway は `jobs` を返している。
* エージェントの cron ツールスキーマは任意の `job` オブジェクトを許容しており、不正な入力を受け付けてしまう。
* Gateway は `cron.add` を正規化なしで厳密に検証するため、ラップされたペイロードは検証に通らない。

<div id="what-changed">
  ## 変更点
</div>

* `cron.add` と `cron.update` は、一般的なラッパー形式を正規化し、不足している `kind` フィールドを補完するようになりました。
* エージェントの cron ツールのスキーマが Gateway のスキーマと一致するようになり、無効なペイロードが減りました。
* プロバイダーの列挙型が Gateway、CLI、UI、macOS ピッカー間で整合するようになりました。
* Control UI はステータス表示に Gateway の `jobs` カウントフィールドを使用します。

<div id="current-behavior">
  ## 現在の動作
</div>

* **正規化:** ラップされた `data`/`job` ペイロードは展開され、安全な場合は `schedule.kind` と `payload.kind` が推論されます。
* **デフォルト:** `wakeMode` と `sessionTarget` が指定されていない場合、安全なデフォルト値が適用されます。
* **プロバイダー:** Discord/Slack/Signal/iMessage が CLI/UI の両方で一貫して表示されるようになりました。

正規化された構造と例については、[Cron jobs](/ja/automation/cron-jobs) を参照してください。

<div id="verification">
  ## 検証
</div>

* Gateway のログを監視し、`cron.add` の INVALID&#95;REQUEST エラーが減少していることを確認します。
* Control UI の cron ステータスで、再読み込み後にジョブ数が表示されていることを確認します。

<div id="optional-follow-ups">
  ## 任意のフォローアップ項目
</div>

* Control UI の手動スモークテスト: プロバイダーごとに cron ジョブを追加し、status ジョブ数を検証する。

<div id="open-questions">
  ## 未解決の課題
</div>

* `cron.add` はクライアントから明示的な `state` を受け付けるべきか（現在はスキーマ上許可されていない）？
* 明示的な配信プロバイダーとして `webchat` を許可すべきか（現在は配信解決時にフィルタリングされている）？