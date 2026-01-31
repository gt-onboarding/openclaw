---
title: スキル
summary: "スキル: マネージド型とワークスペース型、ゲート条件ルール、および設定／環境変数（config/env）の連携方法"
read_when:
  - スキルを追加または変更するとき
  - スキルのゲート条件やロードルールを変更するとき
---

<div id="skills-openclaw">
  # スキル (OpenClaw)
</div>

OpenClaw は、エージェントにツールの使い方を教えるために **[AgentSkills](https://agentskills.io) 互換**のスキルフォルダーを使用します。各スキルは、YAML フロントマターと指示を書いた `SKILL.md` を格納したディレクトリです。OpenClaw は **バンドル済みスキル** と任意のローカルでのオーバーライドを読み込み、環境・設定・バイナリの有無に基づいて読み込み時にそれらをフィルタリングします。

<div id="locations-and-precedence">
  ## 位置と優先順位
</div>

スキルは**3つ**の場所から読み込まれます:

1. **バンドル済みスキル**: インストールに含まれているもの (npm パッケージまたは OpenClaw.app)
2. **管理/ローカルスキル**: `~/.openclaw/skills`
3. **ワークスペーススキル**: `<workspace>/skills`

スキル名が重複した場合の優先順位は次のとおりです:

`<workspace>/skills` (最優先) → `~/.openclaw/skills` → バンドル済みスキル (最下位)

さらに、`~/.openclaw/openclaw.json` の `skills.load.extraDirs` で、追加のスキル用ディレクトリ (最下位の優先順位) を設定できます。

<div id="per-agent-vs-shared-skills">
  ## エージェントごとのスキルと共有スキル
</div>

**マルチエージェント**構成では、各エージェントはそれぞれ専用のワークスペースを持ちます。これは次のことを意味します。

* **エージェントごとのスキル**は、そのエージェント専用で `<workspace>/skills` に配置されます。
* **共有スキル**は `~/.openclaw/skills`（managed/local）に配置され、同じマシン上の\
  **すべてのエージェント**から参照できます。
* **共有フォルダ**は、複数のエージェントで共通のスキルパックを使いたい場合に、
  `skills.load.extraDirs`（優先度は最も低い）経由で追加することもできます。

同じスキル名が複数の場所に存在する場合は、通常の優先順位が適用されます。
ワークスペースが最優先、その次に managed/local、最後にバンドル済みのものが使用されます。

<div id="plugins-skills">
  ## プラグイン + スキル
</div>

プラグインは、`openclaw.plugin.json` 内で `skills` ディレクトリを列挙することで、独自のスキルを含めることができます（パスはプラグインのルートからの相対パス）。プラグインのスキルは、プラグインが有効化されたタイミングでロードされ、通常のスキルの優先順位ルールに従って動作します。プラグインの設定エントリ上の `metadata.openclaw.requires.config` を使って、それらの有効化条件を制御できます。ディスカバリ／設定については [Plugins](/ja/plugin)、それらのスキルが公開するツール群については [Tools](/ja/tools) を参照してください。

<div id="clawhub-install-sync">
  ## ClawHub（インストール + 同期）
</div>

ClawHub は OpenClaw 向けの公開スキルレジストリです。https://clawhub.com
で閲覧できます。スキルの検索・発見、インストール、更新、バックアップに使用します。
詳細ガイド: [ClawHub](/ja/tools/clawhub)。

代表的な利用フロー:

* スキルをワークスペースにインストールする:
  * `clawhub install <skill-slug>`
* インストール済みのすべてのスキルを更新する:
  * `clawhub update --all`
* 同期（スキャン + 更新の公開）:
  * `clawhub sync --all`

デフォルトでは、`clawhub` は現在の作業ディレクトリ直下の `./skills`
にインストールします（または、設定済みの OpenClaw ワークスペースにフォールバックします）。OpenClaw は次回のセッション開始時にそれを `<workspace>/skills` として検出します。

<div id="security-notes">
  ## セキュリティに関する注意事項
</div>

* サードパーティ製スキルは **信頼されたコード** として扱われます。有効化する前に必ず内容を確認してください。
* 信頼できない入力や危険なツールには、サンドボックス化された実行を優先して使用してください。[Sandboxing](/ja/gateway/sandboxing) を参照してください。
* `skills.entries.*.env` と `skills.entries.*.apiKey` は、そのエージェントのターンの間、**ホスト** プロセスにシークレットを注入します（サンドボックス内ではありません）。プロンプトやログにシークレットを含めないでください。
* より包括的な脅威モデルとチェックリストについては、[Security](/ja/gateway/security) を参照してください。

<div id="format-agentskills-pi-compatible">
  ## フォーマット（AgentSkills + Pi 互換）
</div>

`SKILL.md` には少なくとも次の内容を含める必要があります：

```markdown
---
name: nano-banana-pro
description: Gemini 3 Pro Imageで画像を生成または編集
---
```

Notes:

* レイアウト／意図については AgentSkills 仕様に従います。
* 埋め込みエージェントで使用されるパーサは、**1 行のみ**の frontmatter キーにしか対応していません。
* `metadata` は **1 行の JSON オブジェクト**である必要があります。
* スキルフォルダーのパスを参照する場合は、手順内で `{baseDir}` を使用します。
* オプションの frontmatter キー:
  * `homepage` — macOS Skills UI で “Website” として表示される URL（`metadata.openclaw.homepage` 経由もサポート）。
  * `user-invocable` — `true|false`（デフォルト: `true`）。`true` の場合、スキルはユーザーのスラッシュコマンドとして公開されます。
  * `disable-model-invocation` — `true|false`（デフォルト: `false`）。`true` の場合、そのスキルはモデルプロンプトから除外されます（ユーザーによる呼び出しでは引き続き利用可能）。
  * `command-dispatch` — `tool`（任意）。`tool` に設定すると、スラッシュコマンドはモデルをバイパスし、直接ツールにディスパッチされます。
  * `command-tool` — `command-dispatch: tool` が設定されているときに呼び出すツール名。
  * `command-arg-mode` — `raw`（デフォルト）。ツールディスパッチ時に、生の引数文字列をコアでパースせず、そのままツールへ渡します。

    ツールは次のパラメータで呼び出されます:
    `{ command: "<raw args>", commandName: "<slash command>", skillName: "<skill name>" }`.

<div id="gating-load-time-filters">
  ## ゲーティング（読み込み時フィルタ）
</div>

OpenClaw は、`metadata`（1 行の JSON）を使って**読み込み時にスキルをフィルタリング**します。

```markdown
---
name: nano-banana-pro
description: Gemini 3 Pro Imageを介して画像を生成または編集
metadata: {"openclaw":{"requires":{"bins":["uv"],"env":["GEMINI_API_KEY"],"config":["browser.enabled"]},"primaryEnv":"GEMINI_API_KEY"}}
---
```

`metadata.openclaw` 配下のフィールド:

* `always: true` — 常にこのスキルを有効にする（他のゲート判定をスキップ）。
* `emoji` — macOS Skills UI で使用される任意の絵文字。
* `homepage` — macOS Skills UI で “Website” として表示される任意の URL。
* `os` — 任意のプラットフォーム一覧（`darwin`, `linux`, `win32`）。設定されている場合、その OS 上でのみスキルが有効になる。
* `requires.bins` — リスト；それぞれが `PATH` 上に存在している必要がある。
* `requires.anyBins` — リスト；少なくとも 1 つが `PATH` 上に存在している必要がある。
* `requires.env` — リスト；環境変数が存在している **か** もしくは設定ファイルで与えられている必要がある。
* `requires.config` — `openclaw.json` 内の、truthy（真と評価される）でなければならないパスのリスト。
* `primaryEnv` — `skills.entries.<name>.apiKey` に紐づく環境変数名。
* `install` — macOS Skills UI で使用される任意のインストーラ仕様配列（brew/node/go/uv/download）。

サンドボックス化に関する注意:

* `requires.bins` はスキルのロード時に **ホスト** 上でチェックされる。
* エージェントがサンドボックス化されている場合、バイナリは **コンテナ内** にも存在している必要がある。
  `agents.defaults.sandbox.docker.setupCommand`（またはカスタムイメージ）を使ってインストールする。
  `setupCommand` はコンテナ作成後に 1 度だけ実行される。
  パッケージのインストールには、外部へのネットワーク疎通、書き込み可能な root ファイルシステム、およびサンドボックス内での root ユーザーが必要となる。
  例: `summarize` スキル（`skills/summarize/SKILL.md`）は、サンドボックスコンテナ内で動作させるために
  そこに `summarize` CLI が必要になる。

インストーラの例:

```markdown
---
name: gemini
description: Use Gemini CLI for coding assistance and Google search lookups.
metadata: {"openclaw":{"emoji":"♊️","requires":{"bins":["gemini"]},"install":[{"id":"brew","kind":"brew","formula":"gemini-cli","bins":["gemini"],"label":"Gemini CLIをインストール (brew)"}]}}
---
```

Notes:

* 複数のインストーラーが列挙されている場合、Gateway は **1つだけ** 優先オプションを選択します（`brew` が利用可能ならそれを使用し、そうでなければ `node` を使用します）。
* すべてのインストーラーが `download` の場合、OpenClaw は各エントリを列挙し、利用可能なアーティファクトを確認できるようにします。
* インストーラー定義には `os: ["darwin"|"linux"|"win32"]` を含めることができ、プラットフォーム別に選択肢を絞り込めます。
* Node インストールは `openclaw.json` 内の `skills.install.nodeManager` を参照します（デフォルト: npm、指定可能: npm/pnpm/yarn/bun）。
  これは **スキルのインストール** のみに影響します。Gateway ランタイム自体は引き続き Node である必要があります
  （WhatsApp/Telegram 用として Bun は推奨されません）。
* Go インストール: `go` が存在せず `brew` が利用可能な場合、Gateway はまず Homebrew 経由で Go をインストールし、可能な場合は `GOBIN` を Homebrew の `bin` に設定します。
* Download インストール: `url`（必須）、`archive`（`tar.gz` | `tar.bz2` | `zip`）、`extract`（デフォルト: archive 検出時は自動）、`stripComponents`、`targetDir`（デフォルト: `~/.openclaw/tools/<skillKey>`）。

`metadata.openclaw` が存在しない場合、そのスキルは（設定で無効化されていない限り、またはバンドル済みスキルに対する `skills.allowBundled` によってブロックされていない限り）常にインストール候補として扱われます。

<div id="config-overrides-openclawopenclawjson">
  ## 設定のオーバーライド（`~/.openclaw/openclaw.json`）
</div>

バンドル／管理対象のスキルは、有効化・無効化を切り替えたり、環境変数の値を設定したりできます。

```json5
{
  skills: {
    entries: {
      "nano-banana-pro": {
        enabled: true,
        apiKey: "GEMINI_KEY_HERE",
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE"
        },
        config: {
          endpoint: "https://example.invalid",
          model: "nano-pro"
        }
      },
      peekaboo: { enabled: true },
      sag: { enabled: false }
    }
  }
}
```

注意: スキル名にハイフンが含まれている場合は、キーを引用符で囲んでください（JSON5 ではキーを引用符で囲めます）。

config キーはデフォルトで **skill name** と一致します。スキルが
`metadata.openclaw.skillKey` を定義している場合は、そのキーを `skills.entries` の下で使用します。

ルール:

* `enabled: false` は、そのスキルがバンドル済み/インストール済みであっても無効化します。
* `env`: プロセス内でその変数がまだ設定されていない場合に **のみ** 注入されます。
* `apiKey`: `metadata.openclaw.primaryEnv` を宣言しているスキル向けの簡便指定です。
* `config`: スキルごとのカスタムフィールド用の任意のコンテナ。カスタムキーは必ずここに置きます。
* `allowBundled`: **bundled** スキル専用の任意の許可リスト。設定されている場合、
  リストに含まれるバンドル済みスキルのみが有効になります（managed スキルおよびワークスペース内スキルには影響しません）。

<div id="environment-injection-per-agent-run">
  ## 環境インジェクション（エージェント実行ごと）
</div>

エージェントの実行が開始されると、OpenClaw は次を行います:

1. スキルのメタデータを読み込みます。
2. `skills.entries.<key>.env` または `skills.entries.<key>.apiKey` に指定された値を
   `process.env` に適用します。
3. **対象となる**スキルを使って system プロンプトを構築します。
4. 実行終了後に元の環境を復元します。

これは**エージェント実行にスコープされており**、グローバルなシェル環境ではありません。

<div id="session-snapshot-performance">
  ## セッションスナップショット（パフォーマンス）
</div>

OpenClaw は、**セッション開始時に** 対象となるスキルのスナップショットを作成し、同じセッション内の後続のターンではそのリストを再利用します。スキルや設定の変更は、次に新しいセッションが開始されたときに反映されます。

スキルウォッチャーが有効な場合や、新しい対象のリモートノードが出現した場合（後述）には、セッションの途中でもスキルを更新できます。これはいわゆる **ホットリロード** のような動作で、更新されたリストは次のエージェントターンで反映されます。

<div id="remote-macos-nodes-linux-gateway">
  ## リモート macOS ノード（Linux Gateway）
</div>

Gateway が Linux 上で動作していても、**macOS ノード** が接続されていて **`system.run` が許可されている場合**（実行承認セキュリティが `deny` に設定されていない場合）、そのノード上に必要なバイナリが存在していれば、OpenClaw は macOS 限定のスキルを利用可能なスキルとして扱います。エージェントはそれらのスキルを `nodes` ツール（通常は `nodes.run`）経由で実行する必要があります。

これは、ノードが自身のコマンド対応状況を報告することと、`system.run` によるバイナリ検出（bin probe）に依存します。後から macOS ノードがオフラインになった場合でも、スキルは引き続き UI 上に表示されますが、ノードが再接続するまでは呼び出しが失敗する可能性があります。

<div id="skills-watcher-auto-refresh">
  ## スキルウォッチャー（自動更新）
</div>

デフォルトでは、OpenClaw はスキル用フォルダを監視し、`SKILL.md` ファイルに変更があるとスキルのスナップショットを更新します。これは `skills.load` で設定します。

```json5
{
  skills: {
    load: {
      watch: true,
      watchDebounceMs: 250
    }
  }
}
```

<div id="token-impact-skills-list">
  ## トークンへの影響（スキル一覧）
</div>

スキルが利用可能な状態になると、OpenClaw は利用可能なスキルのコンパクトな XML 一覧をシステムプロンプト内に挿入します（`pi-coding-agent` 内の `formatSkillsForPrompt` 経由）。このコストは完全に決まっています:

* **ベースオーバーヘッド（スキルが1つ以上ある場合のみ）:** 195文字
* **スキルごと:** 97文字 + XML エスケープされた `<name>`, `<description>`, `<location>` 各値の文字数

計算式（文字数）:

```
total = 195 + Σ (97 + len(name_escaped) + len(description_escaped) + len(location_escaped))
```

注記:

* XML エスケープでは、`& < > " '` がエンティティ（`&amp;`、`&lt;` など）に展開され、その分文字数が増加します。
* トークン数はモデルごとのトークナイザーによって異なります。大まかな OpenAI 方式の目安では **約 4 文字/トークン** なので、**97 文字 ≈ 24 トークン** がスキル 1 つあたりの目安となり、これに実際のフィールドの文字数分がさらに加わります。

<div id="managed-skills-lifecycle">
  ## 管理されたスキルのライフサイクル
</div>

OpenClaw には、インストール時（npm パッケージや OpenClaw.app）に含まれる **バンドル済みスキル** として、ベースラインのスキルセットが含まれています。`~/.openclaw/skills` はローカルでの上書き用ディレクトリとして存在し（たとえば、バンドルされているコピーを変更せずに特定のスキルをバージョン固定したりパッチを適用したりする場合など）、ワークスペースのスキルはユーザー所有であり、名前が衝突した場合にはバンドル済みスキルおよびローカル上書きよりも優先されます。

<div id="config-reference">
  ## 設定リファレンス
</div>

詳細な設定スキーマについては、[スキル設定](/ja/tools/skills-config) を参照してください。

<div id="looking-for-more-skills">
  ## ほかのスキルをお探しですか？
</div>

https://clawhub.com をご覧ください。

***