---
title: スクリプト
summary: "リポジトリ内のスクリプト：目的、スコープ、安全性に関する注意事項"
read_when:
  - リポジトリ内のスクリプトを実行するとき
  - ./scripts 配下のスクリプトを追加または変更するとき
---

<div id="scripts">
  # スクリプト
</div>

`scripts/` ディレクトリには、ローカルでの開発フローや運用タスク向けの補助スクリプトが含まれています。
タスクが特定のスクリプトに明確に対応している場合はこれらを使用し、それ以外の場合は CLI を優先してください。

<div id="conventions">
  ## 慣例
</div>

* ドキュメントやリリースチェックリストで参照されていない限り、スクリプトは**必須ではありません**。
* 可能な場合は CLI のインターフェースを優先して使用してください（例: 認証モニタリングでは `openclaw models status --check` を使用）。
* スクリプトはホスト依存であると想定し、新しいマシンで実行する前に内容を確認してください。

<div id="git-hooks">
  ## Git フック
</div>

* `scripts/setup-git-hooks.js`: git リポジトリ配下で実行している場合に、`core.hooksPath` を可能な範囲で自動設定します。
* `scripts/format-staged.js`: ステージ済みの `src/` および `test/` ファイルに対する pre-commit 用フォーマッタです。

<div id="auth-monitoring-scripts">
  ## 認証監視スクリプト
</div>

認証監視スクリプトについては、次を参照してください:
[/automation/auth-monitoring](/ja/automation/auth-monitoring)

<div id="when-adding-scripts">
  ## スクリプトを追加する場合
</div>

* スクリプトは用途を明確にし、必ずドキュメント化すること。
* 関連するドキュメントに簡潔な説明を追記する（存在しない場合は新規に作成する）。