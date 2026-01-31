---
title: スキル設定
summary: "スキル設定スキーマと例"
read_when:
  - スキル設定を追加または変更するとき
  - バンドルされている許可リストまたはインストール時の挙動を調整するとき
---

<div id="skills-config">
  # スキル設定
</div>

すべてのスキル関連の設定は、`~/.openclaw/openclaw.json` の `skills` 配下で定義されます。

```json5
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: [
        "~/Projects/agent-scripts/skills",
        "~/Projects/oss/some-skill-pack/skills"
      ],
      watch: true,
      watchDebounceMs: 250
    },
    install: {
      preferBrew: true,
      nodeManager: "npm" // npm | pnpm | yarn | bun (Gateway ランタイムは引き続き Node を使用; bun は非推奨)
    },
    entries: {
      "nano-banana-pro": {
        enabled: true,
        apiKey: "GEMINI_KEY_HERE",
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE"
        }
      },
      peekaboo: { enabled: true },
      sag: { enabled: false }
    }
  }
}
```


<div id="fields">
  ## フィールド
</div>

- `allowBundled`: **バンドル済み**スキル専用の任意の許可リスト。設定すると、
  リスト内のバンドル済みスキルのみが対象になる（managed/ワークスペーススキルには影響しない）。
- `load.extraDirs`: 追加でスキャンするスキルディレクトリ（優先度は最も低い）。
- `load.watch`: スキルフォルダを監視し、スキルのスナップショットを更新する（デフォルト: true）。
- `load.watchDebounceMs`: スキルウォッチャーのイベントに対するデバウンス時間（ミリ秒単位）（デフォルト: 250）。
- `install.preferBrew`: 利用可能な場合は brew インストーラーを優先する（デフォルト: true）。
- `install.nodeManager`: Node インストーラーの優先設定（`npm` | `pnpm` | `yarn` | `bun`, デフォルト: npm）。
  これは **スキルのインストール** のみを対象とする。Gateway ランタイム自体は引き続き Node
  を使用する想定（WhatsApp/Telegram には Bun は非推奨）。
- `entries.<skillKey>`: スキルごとのオーバーライド。

スキルごとのフィールド:

- `enabled`: バンドル済み/インストール済みであっても、スキルを無効化するには `false` を設定する。
- `env`: エージェント実行時に注入される環境変数（すでに設定済みの場合は上書きしない）。
- `apiKey`: プライマリ env 変数を宣言しているスキル向けの任意指定用の簡易フィールド。

<div id="notes">
  ## 注意事項
</div>

- `entries` 配下のキーは、デフォルトではスキル名にマップされます。スキル側で
  `metadata.openclaw.skillKey` を定義している場合は、そのキーが代わりに使用されます。
- ウォッチャーが有効になっている場合、スキルの変更は次のエージェントターンで反映されます。

<div id="sandboxed-skills-env-vars">
  ### サンドボックス化されたスキルと環境変数
</div>

セッションが**サンドボックス化**されている場合、スキルプロセスは Docker 内で実行されます。サンドボックス環境は
ホスト側の `process.env` を**引き継ぎません**。

次のいずれかを使用してください:

- `agents.defaults.sandbox.docker.env`（またはエージェントごとの `agents.list[].sandbox.docker.env`）
- 環境変数をカスタムサンドボックスイメージに焼き込む

グローバルな `env` および `skills.entries.<skill>.env/apiKey` は、**ホスト**上で直接実行する場合にのみ適用されます。