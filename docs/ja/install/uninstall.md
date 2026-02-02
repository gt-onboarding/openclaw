---
title: アンインストール
summary: "OpenClaw を完全にアンインストールする（CLI、サービス、状態データ、ワークスペース）"
read_when:
  - マシンから OpenClaw を削除したいとき
  - アンインストールしたのに Gateway サービスがまだ動作しているとき
---

<div id="uninstall">
  # アンインストール
</div>

次の2通りの方法があります:

- `openclaw` がまだインストールされている場合の **簡単な方法**。
- CLI は削除されたがサービスはまだ動作している場合の **サービスの手動削除**。

<div id="easy-path-cli-still-installed">
  ## 簡単な手順（CLI がまだインストールされている場合）
</div>

推奨: 組み込みのアンインストーラーを使用してください。

```bash
openclaw uninstall
```

非対話（自動化 / npx）:

```bash
openclaw uninstall --all --yes --non-interactive
npx -y openclaw uninstall --all --yes --non-interactive
```

手動で実行する場合（結果は同じ）:

1. Gateway サービスを停止します:

```bash
openclaw gateway stop
```

2. Gateway サービスをアンインストールします（launchd/systemd/schtasks）：

```bash
openclaw gateway uninstall
```

3. 状態と設定を削除:

```bash
rm -rf "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}"
```

`OPENCLAW_CONFIG_PATH` を state ディレクトリ外のカスタムパスに設定している場合は、そのファイルも削除してください。

4. ワークスペースを削除する（任意。エージェントのファイルを削除します）:

```bash
rm -rf ~/.openclaw/workspace
```

5. CLI のインストールを削除します（使用したインストール方法を選択）:

```bash
npm rm -g openclaw
pnpm remove -g openclaw
bun remove -g openclaw
```

6. macOS アプリをインストール済みの場合：

```bash
rm -rf /Applications/OpenClaw.app
```

注意:

* プロファイル（`--profile` / `OPENCLAW_PROFILE`）を使用している場合は、各 state ディレクトリ（デフォルトは `~/.openclaw-<profile>`）ごとに手順 3 を繰り返してください。
* リモートモードでは state ディレクトリは **Gateway ホスト** 上にあるため、手順 1〜4 も同様にそのホスト上で実行してください。


<div id="manual-service-removal-cli-not-installed">
  ## 手動でのサービス削除（CLI がインストールされていない場合）
</div>

Gateway サービスが動作し続けているのに `openclaw` が存在しない場合にこの手順を使用します。

<div id="macos-launchd">
  ### macOS (launchd)
</div>

デフォルトのラベルは `bot.molt.gateway`（または `bot.molt.<profile>`。レガシーな `com.openclaw.*` ラベルが残っている場合があります）です。

```bash
launchctl bootout gui/$UID/bot.molt.gateway
rm -f ~/Library/LaunchAgents/bot.molt.gateway.plist
```

プロファイルを使用している場合は、ラベルと plist 名を `bot.molt.<profile>` に置き換えてください。存在する場合は、従来の `com.openclaw.*` の plist をすべて削除してください。


<div id="linux-systemd-user-unit">
  ### Linux（systemd ユーザーユニット）
</div>

デフォルトのユニット名は `openclaw-gateway.service`（または `openclaw-gateway-<profile>.service`）です。

```bash
systemctl --user disable --now openclaw-gateway.service
rm -f ~/.config/systemd/user/openclaw-gateway.service
systemctl --user daemon-reload
```


<div id="windows-scheduled-task">
  ### Windows（スケジュールタスク）
</div>

デフォルトのタスク名は `OpenClaw Gateway`（または `OpenClaw Gateway (<profile>)`）です。
タスク用スクリプトは、state ディレクトリ配下に配置されます。

```powershell
schtasks /Delete /F /TN "OpenClaw Gateway"
Remove-Item -Force "$env:USERPROFILE\.openclaw\gateway.cmd"
```

プロファイルを使用した場合は、対応するタスク名と `~\.openclaw-<profile>\gateway.cmd` を削除してください。


<div id="normal-install-vs-source-checkout">
  ## 通常インストール vs ソースチェックアウト
</div>

<div id="normal-install-installsh-npm-pnpm-bun">
  ### 通常のインストール（install.sh / npm / pnpm / bun）
</div>

`https://openclaw.bot/install.sh` または `install.ps1` を使用した場合、CLI は `npm install -g openclaw@latest` によってインストールされています。
アンインストールするには `npm rm -g openclaw` を実行します（pnpm / bun でインストールした場合は、代わりに `pnpm remove -g` または `bun remove -g` を実行します）。

<div id="source-checkout-git-clone">
  ### ソースチェックアウト（git clone）
</div>

リポジトリのチェックアウト（`git clone` + `openclaw ...` / `bun run openclaw ...`）から実行している場合:

1) リポジトリを削除する**前に** Gateway サービスをアンインストールします（上記の簡単な手順を使うか、サービスを手動で削除してください）。
2) リポジトリディレクトリを削除します。
3) 上記のとおり、state とワークスペースを削除します。