---
title: 開発チャンネル
summary: "stable・beta・dev チャンネルの位置づけ、切り替え、タグ付け"
read_when:
  - stable/beta/dev チャンネルを切り替えたいとき
  - プレリリースにタグ付けしたり公開したりするとき
---

<div id="development-channels">
  # 開発チャネル
</div>

最終更新日: 2026-01-21

OpenClaw には 3 つのアップデートチャネルがあります:

- **stable**: npm dist-tag `latest`。
- **beta**: npm dist-tag `beta`（テスト中のビルド）。
- **dev**: `main`（git）の常に更新され続ける先頭。npm dist-tag: `dev`（公開されている場合）。

ビルドはまず **beta** にリリースしてテストし、その後、**検証済みビルドを `latest` にプロモート**します。
この際、バージョン番号は変更せず、npm インストール時の唯一の信頼できる情報源は dist-tag です。

<div id="switching-channels">
  ## チャンネルの切り替え
</div>

Git でのチェックアウト:

```bash
openclaw update --channel stable
openclaw update --channel beta
openclaw update --channel dev
```

* `stable`/`beta` は、最新の対応するタグをチェックアウトします（多くの場合は同じタグです）。
* `dev` は `main` に切り替え、upstream の最新にリベースします。

npm/pnpm のグローバルインストール:

```bash
openclaw update --channel stable
openclaw update --channel beta
openclaw update --channel dev
```

これは対応する npm の dist-tag（`latest`、`beta`、`dev`）経由で更新されます。

`--channel` で **明示的に** チャンネルを切り替えた場合、OpenClaw は
インストール方法もそれに合わせます。

* `dev` は git チェックアウト（デフォルトは `~/openclaw`。`OPENCLAW_GIT_DIR` で上書き可能）を保証し、
  それを更新して、そのチェックアウトからグローバル CLI をインストールします。
* `stable`/`beta` は、対応する dist-tag を使って npm からインストールします。

ヒント: stable と dev を並行して使いたい場合は、リポジトリを 2 つクローンし、Gateway は stable 側のクローンを指すようにしておきます。


<div id="plugins-and-channels">
  ## プラグインとチャネル
</div>

`openclaw update` でチャネルを切り替えると、OpenClaw はプラグインのソースも同期します：

- `dev` は git チェックアウトに含まれているバンドル済みプラグインを優先して使用します。
- `stable` および `beta` は npm でインストールされたプラグインパッケージを復元します。

<div id="tagging-best-practices">
  ## タグ付けのベストプラクティス
</div>

- git のチェックアウト先にしたいリリースにはタグを付ける（`vYYYY.M.D` または `vYYYY.M.D-<patch>`）。
- タグは不変に保つ：タグを移動したり再利用したりしない。
- npm の dist-tag を、npm インストール時の唯一の信頼できる情報源とする：
  - `latest` → 安定版
  - `beta` → 候補ビルド
  - `dev` → main ブランチのスナップショット（任意）

<div id="macos-app-availability">
  ## macOS アプリの提供状況
</div>

Beta ビルドおよび dev ビルドには、macOS アプリのリリースが **含まれない場合があります**。その場合でも問題ありません。

- git タグや npm dist-tag は引き続き公開できます。
- リリースノートまたは changelog で「この beta には macOS ビルドはありません」と明記します。