---
title: 複数の Gateway
summary: "1台のホスト上で複数の OpenClaw Gateway を実行する（分離、ポート、プロファイル）"
read_when:
  - 同じマシン上で複数の Gateway を実行している場合
  - Gateway ごとに config/state/ポートを分離したい場合
---

<div id="multiple-gateways-same-host">
  # 複数 Gateway（同一ホスト）
</div>

ほとんどの構成では Gateway は 1 つで十分です。単一の Gateway で複数のメッセージング接続やエージェントを扱えるためです。より強力な分離や冗長性（例: レスキューボット）が必要な場合は、プロファイルやポートを分離した複数の Gateway を個別に実行してください。

<div id="isolation-checklist-required">
  ## 分離チェックリスト（必須）
</div>

- `OPENCLAW_CONFIG_PATH` — インスタンスごとの設定ファイル
- `OPENCLAW_STATE_DIR` — インスタンスごとのセッション／認証情報／キャッシュ
- `agents.defaults.workspace` — インスタンスごとのワークスペースのルート
- `gateway.port`（または `--port`） — インスタンスごとに固有
- 派生ポート（ブラウザ／キャンバス用）はインスタンス間で重複してはいけません

これらを共有していると、設定競合やポート競合が発生します。

<div id="recommended-profiles-profile">
  ## 推奨: プロファイル（`--profile`）
</div>

プロファイルは `OPENCLAW_STATE_DIR` と `OPENCLAW_CONFIG_PATH` を自動的にスコープ付けし、サービス名にサフィックスを付与します。

```bash
# main
openclaw --profile main setup
openclaw --profile main gateway --port 18789

# rescue
openclaw --profile rescue setup
openclaw --profile rescue gateway --port 19001
```

プロファイル別のサービス：

```bash
openclaw --profile main gateway install
openclaw --profile rescue gateway install
```


<div id="rescue-bot-guide">
  ## Rescue-bot ガイド
</div>

同じホスト上で、次の要素をそれぞれ個別に持つ 2 つ目の Gateway を起動します:

- プロファイル/設定
- state ディレクトリ
- ワークスペース
- ベースポート（およびそこから派生するポート）

これにより、レスキューボットはメインボットから分離され、プライマリボットがダウンしている場合でも、デバッグや設定変更の適用を行えるようになります。

ポート間隔: ベースポート同士の間に少なくとも 20 個のポート番号の余裕を持たせ、派生するブラウザ/キャンバス/CDP ポートが衝突しないようにします。

<div id="how-to-install-rescue-bot">
  ### インストール方法（Rescue Bot）
</div>

```bash
# Main bot (existing or fresh, without --profile param)
# Runs on port 18789 + Chrome CDC/Canvas/... Ports 
openclaw onboard
openclaw gateway install

# Rescue bot (isolated profile + ports)
openclaw --profile rescue onboard
# 注意: 
# - ワークスペース名はデフォルトで末尾に -rescue が付加されます
# - ポートは最低でも 18789 + 20 ポート必要です。
#   19789 のように完全に異なるベースポートを選択することを推奨します。
# - オンボーディングの残りの手順は通常と同じです

# To install the service (if not happened automatically during onboarding)
openclaw --profile rescue gateway install
```


<div id="port-mapping-derived">
  ## ポートマッピング（導出）
</div>

ベースポート = `gateway.port`（または `OPENCLAW_GATEWAY_PORT` / `--port`）。

- ブラウザ制御サービスのポート = ベース + 2（ループバックのみ）
- `canvasHost.port = ベース + 4`
- ブラウザプロファイルの CDP ポートは `browser.controlPort + 9 .. + 108` の範囲から自動的に割り当てられる

これらのいずれかを設定や環境変数で上書きする場合は、必ずインスタンスごとに一意になるようにしてください。

<div id="browsercdp-notes-common-footgun">
  ## ブラウザ/CDP に関する注意（よくあるハマりどころ）
</div>

- 複数のインスタンスで `browser.cdpUrl` を同じ値に固定しないでください。
- 各インスタンスには、それぞれ専用のブラウザ制御ポートと CDP ポートレンジ（対応する Gateway ポートから導出）が必要です。
- 明示的な CDP ポートが必要な場合は、インスタンスごとに `browser.profiles.<name>.cdpPort` を設定してください。
- リモート Chrome を使う場合は、`browser.profiles.<name>.cdpUrl` を使用してください（プロファイルごと、インスタンスごと）。

<div id="manual-env-example">
  ## 手動での環境変数設定例
</div>

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/main.json \
OPENCLAW_STATE_DIR=~/.openclaw-main \
openclaw gateway --port 18789

OPENCLAW_CONFIG_PATH=~/.openclaw/rescue.json \
OPENCLAW_STATE_DIR=~/.openclaw-rescue \
openclaw gateway --port 19001
```


<div id="quick-checks">
  ## 簡易確認
</div>

```bash
openclaw --profile main status
openclaw --profile rescue status
openclaw --profile rescue browser status
```
