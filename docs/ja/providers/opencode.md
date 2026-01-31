---
title: Opencode
summary: "OpenClaw で OpenCode Zen（キュレーション済みモデル）を使う"
read_when:
  - モデルへのアクセスに OpenCode Zen を使いたい
  - コーディングに適したモデルのキュレーション済みリストが欲しい
---

<div id="opencode-zen">
  # OpenCode Zen
</div>

OpenCode Zen は、コーディング用エージェント向けに OpenCode チームが推奨する、**厳選されたモデルの一覧**です。
api キーと `opencode` プロバイダーを利用する、任意利用のホスト型モデルアクセスパスです。
Zen は現在ベータ版です。

<div id="cli-setup">
  ## CLI のセットアップ
</div>

```bash
openclaw onboard --auth-choice opencode-zen
# または非対話モード
openclaw onboard --opencode-zen-api-key "$OPENCODE_API_KEY"
```


<div id="config-snippet">
  ## 設定スニペット
</div>

```json5
{
  env: { OPENCODE_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "opencode/claude-opus-4-5" } } }
}
```


<div id="notes">
  ## 注意事項
</div>

- `OPENCODE_ZEN_API_KEY` も利用できます。
- Zen にサインインし、請求情報を追加してから API キーをコピーします。
- OpenCode Zen はリクエストごとに課金されます。詳細は OpenCode ダッシュボードで確認してください。