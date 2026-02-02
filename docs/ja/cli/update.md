---
title: 更新
summary: "`openclaw update` の CLI リファレンス（比較的安全なソースコード更新 + Gateway の自動再起動）"
read_when:
  - "ソースのチェックアウトを安全に更新したいとき"
  - "`--update` の省略形の挙動を理解する必要があるとき"
---

<div id="openclaw-update">
  # `openclaw update`
</div>

OpenClaw を安全にアップデートし、stable / beta / dev チャンネル間を切り替えます。

**npm/pnpm**（グローバルインストールで git メタデータがない場合）でインストールしている場合、アップデートは [Updating](/ja/install/updating) に記載のパッケージマネージャーフローに従って実行されます。

<div id="usage">
  ## 使用方法
</div>

```bash
openclaw update
openclaw update status
openclaw update wizard
openclaw update --channel beta
openclaw update --channel dev
openclaw update --tag beta
openclaw update --no-restart
openclaw update --json
openclaw --update
```

<div id="options">
  ## オプション
</div>

* `--no-restart`: アップデートが成功しても Gateway サービスの再起動をスキップします。
* `--channel <stable|beta|dev>`: アップデートチャンネルを設定します（git + npm。設定に永続化されます）。
* `--tag <dist-tag|version>`: このアップデートに限り、npm の dist-tag またはバージョンを上書きします。
* `--json`: 機械可読な `UpdateRunResult` の JSON を出力します。
* `--timeout <seconds>`: 各ステップのタイムアウト（デフォルトは 1200 秒）。

注意: ダウングレードでは、古いバージョンによって設定が壊れる可能性があるため、確認が必要です。

<div id="update-status">
  ## `update status`
</div>

アクティブなアップデートチャネルと（ソースチェックアウトの場合の）git タグ／ブランチ／SHA、さらにアップデートの有無を表示します。

```bash
openclaw update status
openclaw update status --json
openclaw update status --timeout 10
```

Options:

* `--json`: 機械可読なステータス JSON を出力します。
* `--timeout <seconds>`: チェックのタイムアウト秒数を指定します（既定値は 3 秒）。

<div id="update-wizard">
  ## `update wizard`
</div>

対話的なフローでアップデートチャンネルを選択し、更新後に Gateway を再起動するかどうかを確認します（デフォルトでは再起動します）。git のチェックアウトがない状態で `dev` を選択した場合は、チェックアウトを作成するかどうかを確認します。

<div id="what-it-does">
  ## 動作概要
</div>

明示的にチャネルを切り替えた場合（`--channel ...`）、OpenClaw は
インストール方法もそれに合わせて揃えます:

* `dev` → Git チェックアウトが存在することを保証し（デフォルト: `~/openclaw`、`OPENCLAW_GIT_DIR` で上書き可能）、
  それを更新し、そのチェックアウトからグローバル CLI をインストールします。
* `stable`/`beta` → 対応する dist-tag を使って npm からインストールします。

<div id="git-checkout-flow">
  ## Git チェックアウトフロー
</div>

チャネル:

* `stable`: 最新の非ベータタグをチェックアウトし、その後 build + doctor を実行します。
* `beta`: 最新の `-beta` タグをチェックアウトし、その後 build + doctor を実行します。
* `dev`: `main` をチェックアウトし、その後 fetch + rebase を実行します。

概要:

1. クリーンな worktree（未コミットの変更がない状態）が必要です。
2. 選択したチャネル（タグまたはブランチ）に切り替えます。
3. upstream を fetch します（dev のみ）。
4. dev のみ: 一時的な worktree で事前チェックとして lint + TypeScript ビルドを実行し、先頭コミットで失敗した場合は最大 10 コミットまで遡って最新のクリーンビルドを探します。
5. 選択したコミットに対して rebase します（dev のみ）。
6. 依存関係をインストールします（pnpm を優先し、npm をフォールバックとして使用）。
7. 本体と Control UI をビルドします。
8. 最終的な「安全なアップデート」チェックとして `openclaw doctor` を実行します。
9. プラグインをアクティブなチャネルに同期し（dev はバンドルされた拡張を使用し、stable/beta は npm を使用）、npm でインストールされたプラグインを更新します。

<div id="update-shorthand">
  ## `--update` の省略形
</div>

`openclaw --update` は `openclaw update` の短縮形です（シェルやランチャースクリプトで便利です）。

<div id="see-also">
  ## 関連項目
</div>

* `openclaw doctor`（Git チェックアウト環境では、まずアップデートの実行を提案します）
* [開発向けチャネル](/ja/install/development-channels)
* [アップデート手順](/ja/install/updating)
* [CLI リファレンス](/ja/cli)