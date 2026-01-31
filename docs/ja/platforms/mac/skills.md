---
title: スキル
summary: "macOS スキル設定 UI と Gateway によって管理されるステータス"
read_when:
  - macOS スキル設定 UI を更新する場合
  - スキルの制限やインストール動作を変更する場合
---

<div id="skills-macos">
  # スキル (macOS)
</div>

macOS アプリは Gateway 経由で OpenClaw のスキルを利用可能にしますが、スキルをローカルでパースすることはありません。

<div id="data-source">
  ## データソース
</div>

- `skills.status`（Gateway）は、すべてのスキルと、それぞれの利用可否および不足している要件
  （バンドル済みスキルに対する許可リストによるブロックを含む）を返します。
- 要件は、各 `SKILL.md` 内の `metadata.openclaw.requires` から決定されます。

<div id="install-actions">
  ## インストールアクション
</div>

- `metadata.openclaw.install` はインストールオプション（brew/node/go/uv）を定義します。
- アプリは `skills.install` を呼び出し、Gateway ホスト上でインストーラーを実行します。
- 複数のインストーラーが指定されている場合、Gateway は 1 つの優先インストーラーのみを提示します
  （利用可能であれば brew、それ以外は `skills.install` からの node マネージャー、デフォルトは npm）。

<div id="envapi-keys">
  ## 環境変数 / API キー
</div>

- アプリはキーを `~/.openclaw/openclaw.json` の `skills.entries.<skillKey>` 配下に保存します。
- `skills.update` は `enabled`、`apiKey`、`env` を更新します。

<div id="remote-mode">
  ## リモートモード
</div>

- インストールや設定の更新は、ローカルの Mac ではなく Gateway が動作しているホスト側で行われます。