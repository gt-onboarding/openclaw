---
title: プラグイン
summary: "OpenClaw プラグイン/拡張機能: 検出、構成、セキュリティ"
read_when:
  - プラグイン/拡張機能を追加・変更するとき
  - プラグインのインストールや読み込みルールをドキュメント化するとき
---

<div id="plugins-extensions">
  # プラグイン（拡張機能）
</div>

<div id="quick-start-new-to-plugins">
  ## クイックスタート（プラグインが初めての方へ）
</div>

プラグインは、OpenClaw に追加機能（コマンド、ツール、Gateway RPC）を提供して拡張するための **小さなコードモジュール** です。

多くの場合、まだ OpenClaw コアに組み込まれていない機能が欲しいとき（またはオプション機能をメインのインストールから切り離しておきたいとき）にプラグインを利用します。

簡単な手順:

1. すでにロードされているものを確認する:

```bash
openclaw plugins list
```

2. 公式プラグイン（例: Voice Call）をインストールします。

```bash
openclaw plugins install @openclaw/voice-call
```

3. Gateway を再起動し、その後 `plugins.entries.<id>.config` で設定します。

具体的なプラグインの例については [Voice Call](/ja/plugins/voice-call) を参照してください。

<div id="available-plugins-official">
  ## 利用可能なプラグイン（公式）
</div>

* Microsoft Teams は 2026.1.15 時点でプラグインとしてのみ提供されています。Teams を使用する場合は `@openclaw/msteams` をインストールしてください。
* Memory (Core) — バンドルされているメモリ検索プラグイン（`plugins.slots.memory` でデフォルト有効）
* Memory (LanceDB) — バンドルされている長期メモリプラグイン（自動リコール／キャプチャ；`plugins.slots.memory = "memory-lancedb"` を設定）
* [Voice Call](/ja/plugins/voice-call) — `@openclaw/voice-call`
* [Zalo Personal](/ja/plugins/zalouser) — `@openclaw/zalouser`
* [Matrix](/ja/channels/matrix) — `@openclaw/matrix`
* [Nostr](/ja/channels/nostr) — `@openclaw/nostr`
* [Zalo](/ja/channels/zalo) — `@openclaw/zalo`
* [Microsoft Teams](/ja/channels/msteams) — `@openclaw/msteams`
* Google Antigravity OAuth（プロバイダー認証）— `google-antigravity-auth` としてバンドル（デフォルトは無効）
* Gemini CLI OAuth（プロバイダー認証）— `google-gemini-cli-auth` としてバンドル（デフォルトは無効）
* Qwen OAuth（プロバイダー認証）— `qwen-portal-auth` としてバンドル（デフォルトは無効）
* Copilot Proxy（プロバイダー認証）— ローカル VS Code Copilot Proxy ブリッジ。組み込みの `github-copilot` デバイスログインとは別物（バンドル済みだがデフォルトは無効）

OpenClaw プラグインは、実行時に jiti 経由でロードされる **TypeScript モジュール** です。**設定の
バリデーションはプラグインコードを実行しません**。代わりにプラグインマニフェストと JSON
Schema を使用します。詳細は [Plugin manifest](/ja/plugins/manifest) を参照してください。

プラグインは次のものを登録できます:

* Gateway の RPC メソッド
* Gateway の HTTP ハンドラー
* エージェントツール
* CLI コマンド
* バックグラウンドサービス
* 任意の設定バリデーション
* **スキル**（プラグインマニフェスト内で `skills` ディレクトリを列挙）
* **自動返信コマンド**（AI エージェントを呼び出さずに実行される）

プラグインは Gateway と **同一プロセス内** で動作するため、信頼済みコードとして扱ってください。
ツール作成ガイド: [Plugin agent tools](/ja/plugins/agent-tools)。

<div id="runtime-helpers">
  ## ランタイムヘルパー
</div>

プラグインは `api.runtime` を通じて、特定のコアヘルパーにアクセスできます。音声通話向けの TTS の場合:

```ts
const result = await api.runtime.tts.textToSpeechTelephony({
  text: "Hello from OpenClaw",
  cfg: api.config,
});
```

Notes:

* コアの `messages.tts` 設定（OpenAI または ElevenLabs）を使用します。
* PCM オーディオバッファとサンプルレートを返します。プラグイン側でプロバイダー向けにリサンプリング／エンコードする必要があります。
* Edge TTS は電話（音声通話）用途ではサポートされていません。

<div id="discovery-precedence">
  ## 検出順序と優先順位
</div>

OpenClaw は次の順序でスキャンします。

1. 設定パス

* `plugins.load.paths`（ファイルまたはディレクトリ）

2. ワークスペース拡張機能

* `<workspace>/.openclaw/extensions/*.ts`
* `<workspace>/.openclaw/extensions/*/index.ts`

3. グローバル拡張機能

* `~/.openclaw/extensions/*.ts`
* `~/.openclaw/extensions/*/index.ts`

4. バンドル拡張機能（OpenClaw に同梱、**デフォルトでは無効**）

* `<openclaw>/extensions/*`

バンドル済みプラグインは、`plugins.entries.<id>.enabled`\
または `openclaw plugins enable <id>` で明示的に有効化する必要があります。\
インストール済みプラグインはデフォルトで有効ですが、同じ方法で無効化できます。

各プラグインは、そのルートに `openclaw.plugin.json` ファイルを含める必要があります。パスが
ファイルを指している場合、プラグインのルートはそのファイルがあるディレクトリとなり、
そのディレクトリ内にマニフェストが存在していなければなりません。

複数のプラグインが同じ id に解決される場合、上記の順序で最初に一致したものが優先され、
優先度の低いものは無視されます。

<div id="package-packs">
  ### パッケージパック
</div>

プラグインディレクトリには、`openclaw.extensions` を含む `package.json` を配置できます。

```json
{
  "name": "my-pack",
  "openclaw": {
    "extensions": ["./src/safety.ts", "./src/tools.ts"]
  }
}
```

各エントリがプラグインになります。パックが複数の拡張機能を列挙している場合、プラグイン ID は
`name/<fileBase>` になります。

プラグインが npm の依存パッケージをインポートする場合は、そのディレクトリ内にインストールして、
`node_modules` が利用可能な状態にしてください（`npm install` / `pnpm install`）。

<div id="channel-catalog-metadata">
  ### チャネルカタログのメタデータ
</div>

チャネルプラグインは、`openclaw.channel` を通じてオンボーディング用のメタデータを、
`openclaw.install` を通じてインストール手順のヒントを公開できます。これにより、コアのカタログ本体はデータフリーな状態のまま維持されます。

例:

```json
{
  "name": "@openclaw/nextcloud-talk",
  "openclaw": {
    "extensions": ["./index.ts"],
    "channel": {
      "id": "nextcloud-talk",
      "label": "Nextcloud Talk",
      "selectionLabel": "Nextcloud Talk (self-hosted)",
      "docsPath": "/channels/nextcloud-talk",
      "docsLabel": "nextcloud-talk",
      "blurb": "Nextcloud Talk webhook ボット経由のセルフホストチャット。",
      "order": 65,
      "aliases": ["nc-talk", "nc"]
    },
    "install": {
      "npmSpec": "@openclaw/nextcloud-talk",
      "localPath": "extensions/nextcloud-talk",
      "defaultChoice": "npm"
    }
  }
}
```

OpenClaw は、**外部チャネルカタログ**（たとえば MPM レジストリのエクスポート）も統合できます。次のいずれかの場所に JSON ファイルを配置します。

* `~/.openclaw/mpm/plugins.json`
* `~/.openclaw/mpm/catalog.json`
* `~/.openclaw/plugins/catalog.json`

または、`OPENCLAW_PLUGIN_CATALOG_PATHS`（または `OPENCLAW_MPM_CATALOG_PATHS`）を、1 つ以上の JSON ファイルを指すように設定します（カンマ／セミコロン／`PATH` 区切り）。各ファイルには `{ "entries": [ { "name": "@scope/pkg", "openclaw": { "channel": {...}, "install": {...} } } ] }` の形式でデータを含める必要があります。

<div id="plugin-ids">
  ## プラグイン ID
</div>

デフォルトのプラグイン ID:

* パッケージ: `package.json` の `name`
* スタンドアロンファイル: 拡張子を除いたファイル名 (`~/.../voice-call.ts` → `voice-call`)

プラグインが `id` を export している場合、OpenClaw はその値を使用しますが、
設定された ID と一致しない場合は警告を出します。

<div id="config">
  ## 設定
</div>

```json5
{
  plugins: {
    enabled: true,
    allow: ["voice-call"],
    deny: ["untrusted-plugin"],
    load: { paths: ["~/Projects/oss/voice-call-extension"] },
    entries: {
      "voice-call": { enabled: true, config: { provider: "twilio" } }
    }
  }
}
```

Fields:

* `enabled`: 全体のオン/オフ切り替え（デフォルト: true）
* `allow`: 許可リスト（任意）
* `deny`: 拒否リスト（任意。`deny` が優先）
* `load.paths`: 追加のプラグインファイル/ディレクトリ
* `entries.<id>`: プラグインごとの有効/無効切り替えと設定

設定を変更した場合は、**Gateway を再起動する必要があります**。

検証ルール（厳格）:

* `entries`、`allow`、`deny`、`slots` に不明なプラグイン ID が含まれている場合は、**エラー**になります。
* プラグインのマニフェストでその channel ID が宣言されていない限り、未知の `channels.<id>` キーは **エラー** になります。
* プラグインの設定は、`openclaw.plugin.json`（`configSchema`）に埋め込まれた JSON Schema を使って検証されます。
* プラグインが無効化されている場合でも、その設定は保持され、**警告**が出力されます。

<div id="plugin-slots-exclusive-categories">
  ## プラグインスロット（排他的なカテゴリ）
</div>

一部のプラグインカテゴリは**排他的**であり、一度に有効にできるのは 1 つだけです。`plugins.slots` を使用して、そのスロットをどのプラグインに割り当てるかを選択します。

```json5
{
  plugins: {
    slots: {
      memory: "memory-core" // または "none" でメモリプラグインを無効化
    }
  }
}
```

複数のプラグインが `kind: "memory"` を宣言している場合、選択されたプラグインだけが読み込まれます。その他のプラグインは、診断出力とともに無効化されます。

<div id="control-ui-schema-labels">
  ## Control UI（スキーマ + ラベル）
</div>

Control UI は、`config.schema`（JSON Schema + `uiHints`）を使って、より使いやすいフォームを表示します。

OpenClaw は、検出されたプラグインに基づいて実行時に `uiHints` を拡張します:

* `plugins.entries.<id>` / `.enabled` / `.config` ごとに、プラグイン単位のラベルを追加
* 次のパス配下に、プラグインが任意で提供する設定フィールド用ヒントをマージ:
  `plugins.entries.<id>.config.<field>`

プラグインの設定フィールドに適切なラベルやプレースホルダを表示し（かつシークレットを機微情報として扱うように）したい場合は、
プラグインマニフェスト内の JSON Schema とあわせて `uiHints` を提供してください。

例:

```json
{
  "id": "my-plugin",
  "configSchema": {
    "type": "object",
    "additionalProperties": false,
    "properties": {
      "apiKey": { "type": "string" },
      "region": { "type": "string" }
    }
  },
  "uiHints": {
    "apiKey": { "label": "API Key", "sensitive": true },
    "region": { "label": "リージョン", "placeholder": "us-east-1" }
  }
}
```

<div id="cli">
  ## CLI
</div>

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins install <path>                 # ローカルファイル/ディレクトリを ~/.openclaw/extensions/<id> にコピー
openclaw plugins install ./extensions/voice-call # relative path ok
openclaw plugins install ./plugin.tgz           # install from a local tarball
openclaw plugins install ./plugin.zip           # install from a local zip
openclaw plugins install -l ./extensions/voice-call # link (no copy) for dev
openclaw plugins install @openclaw/voice-call # install from npm
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins doctor
```

`plugins update` は、npm でインストールされていて `plugins.installs` で管理されているものに対してのみ有効です。

プラグインは、独自のトップレベルコマンドを登録することもできます（例: `openclaw voicecall`）。

<div id="plugin-api-overview">
  ## プラグイン API（概要）
</div>

プラグインは次のいずれかの形式でエクスポートします:

* 関数: `(api) => { ... }`
* オブジェクト: `{ id, name, configSchema, register(api) { ... } }`

<div id="plugin-hooks">
  ## プラグインフック
</div>

プラグインはフックを同梱しておき、実行時に登録できます。これにより、プラグイン単体でイベント駆動の自動化を提供でき、別途フックパックをインストールする必要がありません。

<div id="example">
  ### 例
</div>

```
import { registerPluginHooksFromDir } from "openclaw/plugin-sdk";

export default function register(api) {
  registerPluginHooksFromDir(api, "./hooks");
}
```

注意:

* Hook ディレクトリは通常の Hook 構成（`HOOK.md` + `handler.ts`）に従います。
* Hook の適用可否ルール（OS / バイナリ / 環境変数 / 設定の要件）は引き続き適用されます。
* プラグイン管理の Hook は、`openclaw hooks list` の出力で `plugin:<id>` として表示されます。
* `openclaw hooks` からプラグイン管理の Hook を有効化 / 無効化することはできません。代わりに、該当プラグイン自体を有効化 / 無効化してください。

<div id="provider-plugins-model-auth">
  ## プロバイダー プラグイン（モデル認証）
</div>

プラグインは **モデル プロバイダー認証** フローを登録でき、ユーザーは OpenClaw 内で OAuth や
API キーのセットアップを実行できます（外部スクリプトは不要です）。

`api.registerProvider(...)` を使ってプロバイダーを登録します。各プロバイダーは 1 つ以上の
認証メソッド（OAuth、API キー、デバイスコードなど）を公開します。これらのメソッドは次のコマンドで使用されます:

* `openclaw models auth login --provider <id> [--method <id>]`

例:

```ts
api.registerProvider({
  id: "acme",
  label: "AcmeAI",
  auth: [
    {
      id: "oauth",
      label: "OAuth",
      kind: "oauth",
      run: async (ctx) => {
        // OAuth フローを実行し、認証プロファイルを返します。
        return {
          profiles: [
            {
              profileId: "acme:default",
              credential: {
                type: "oauth",
                provider: "acme",
                access: "...",
                refresh: "...",
                expires: Date.now() + 3600 * 1000,
              },
            },
          ],
          defaultModel: "acme/opus-1",
        };
      },
    },
  ],
});
```

Notes:

* `run` は、`prompter`、`runtime`、`openUrl`、`oauth.createVpsAwareHandlers` といったヘルパーを含む `ProviderAuthContext` を受け取ります。
* デフォルトのモデルやプロバイダー設定を追加する必要がある場合は、`configPatch` を返します。
* `--set-default` でエージェントのデフォルト設定を更新できるようにするには、`defaultModel` を返します。

<div id="register-a-messaging-channel">
  ### メッセージングチャネルを登録する
</div>

プラグインは、組み込みチャネル（WhatsApp、Telegram など）と同様に動作する**チャネルプラグイン**を登録できます。
チャネル設定は `channels.<id>` 配下に定義され、チャネルプラグイン側のコードによって検証されます。

```ts
const myChannel = {
  id: "acmechat",
  meta: {
    id: "acmechat",
    label: "AcmeChat",
    selectionLabel: "AcmeChat (API)",
    docsPath: "/channels/acmechat",
    blurb: "demo channel plugin.",
    aliases: ["acme"],
  },
  capabilities: { chatTypes: ["direct"] },
  config: {
    listAccountIds: (cfg) => Object.keys(cfg.channels?.acmechat?.accounts ?? {}),
    resolveAccount: (cfg, accountId) =>
      (cfg.channels?.acmechat?.accounts?.[accountId ?? "default"] ?? { accountId }),
  },
  outbound: {
    deliveryMode: "direct",
    sendText: async () => ({ ok: true }),
  },
};

export default function (api) {
  api.registerChannel({ plugin: myChannel });
}
```

Notes:

* 設定は（`plugins.entries` ではなく）`channels.<id>` の下に配置します。
* `meta.label` は CLI/UI の一覧で表示されるラベルとして使われます。
* `meta.aliases` は正規化および CLI 入力用の代替 id を追加します。
* `meta.preferOver` は、両方のチャネルが設定されている場合に自動有効化をスキップするチャネル id を列挙します。
* `meta.detailLabel` と `meta.systemImage` により、UI でより豊富なチャネルラベル／アイコンを表示できます。

<div id="write-a-new-messaging-channel-stepbystep">
  ### 新しいメッセージングチャネルを作成する（ステップバイステップ）
</div>

**新しいチャット用インターフェース**（「メッセージングチャネル」）が必要なときに使います。モデルプロバイダーではありません。
モデルプロバイダーに関するドキュメントは `/providers/*` にあります。

1. id と config の構造を決める

* チャネルの設定はすべて `channels.<id>` の下に置きます。
* 複数アカウント構成では、`channels.<id>.accounts.<accountId>` を優先して使います。

2. チャネルのメタデータを定義する

* `meta.label`, `meta.selectionLabel`, `meta.docsPath`, `meta.blurb` は CLI/UI の一覧表示を制御します。
* `meta.docsPath` は `/channels/<id>` のようなドキュメントページを指すようにします。
* `meta.preferOver` を使うと、あるプラグインが別のチャネルを置き換えられます（自動有効化時はこれが優先されます）。
* `meta.detailLabel` と `meta.systemImage` は、詳細テキストやアイコンとして UI によって使用されます。

3. 必須アダプターを実装する

* `config.listAccountIds` と `config.resolveAccount`
* `capabilities`（チャット種別、メディア、スレッドなど）
* `outbound.deliveryMode` と `outbound.sendText`（基本的な送信処理用）

4. 必要に応じてオプションのアダプターを追加する

* `setup`（ウィザード）、`security`（DM ポリシー）、`status`（ヘルスチェック/診断）
* `gateway`（起動/停止/ログイン）、`mentions`、`threading`、`streaming`
* `actions`（メッセージアクション）、`commands`（ネイティブコマンドの動作）

5. プラグイン内でチャネルを登録する

* `api.registerChannel({ plugin })`

最小限の設定例:

```json5
{
  channels: {
    acmechat: {
      accounts: {
        default: { token: "ACME_TOKEN", enabled: true }
      }
    }
  }
}
```

最小構成のチャネルプラグイン（送信専用）:

```ts
const plugin = {
  id: "acmechat",
  meta: {
    id: "acmechat",
    label: "AcmeChat",
    selectionLabel: "AcmeChat (API)",
    docsPath: "/channels/acmechat",
    blurb: "AcmeChat messaging channel.",
    aliases: ["acme"],
  },
  capabilities: { chatTypes: ["direct"] },
  config: {
    listAccountIds: (cfg) => Object.keys(cfg.channels?.acmechat?.accounts ?? {}),
    resolveAccount: (cfg, accountId) =>
      (cfg.channels?.acmechat?.accounts?.[accountId ?? "default"] ?? { accountId }),
  },
  outbound: {
    deliveryMode: "direct",
    sendText: async ({ text }) => {
      // ここで `text` をチャンネルに配信
      return { ok: true };
    },
  },
};

export default function (api) {
  api.registerChannel({ plugin });
}
```

プラグインをロードし（extensions ディレクトリまたは `plugins.load.paths`）、Gateway を再起動してから、設定ファイルの `channels.<id>` を設定します。

<div id="agent-tools">
  ### エージェントツール
</div>

詳しくは専用ガイドを参照してください: [プラグイン用エージェントツール](/ja/plugins/agent-tools)。

<div id="register-a-gateway-rpc-method">
  ### Gateway RPC メソッドを登録する
</div>

```ts
export default function (api) {
  api.registerGatewayMethod("myplugin.status", ({ respond }) => {
    respond(true, { ok: true });
  });
}
```

<div id="register-cli-commands">
  ### CLI コマンドの登録
</div>

```ts
export default function (api) {
  api.registerCli(({ program }) => {
    program.command("mycmd").action(() => {
      console.log("Hello");
    });
  }, { commands: ["mycmd"] });
}
```

<div id="register-auto-reply-commands">
  ### 自動返信コマンドを登録する
</div>

プラグインは、**AI エージェントを呼び出さずに**実行されるカスタムスラッシュコマンドを登録できます。これは、ON/OFF を切り替えるコマンド、ステータスチェック、LLM 処理を必要としない簡易なアクションに便利です。

```ts
export default function (api) {
  api.registerCommand({
    name: "mystatus",
    description: "Show plugin status",
    handler: (ctx) => ({
      text: `Plugin is running! Channel: ${ctx.channel}`,
    }),
  });
}
```

コマンドハンドラーのコンテキスト:

* `senderId`: 送信者の ID（存在する場合）
* `channel`: コマンドが送信されたチャネル
* `isAuthorizedSender`: 送信者が認可済みユーザーかどうか
* `args`: コマンドの後ろに渡された引数（`acceptsArgs: true` の場合）
* `commandBody`: コマンド全文のテキスト
* `config`: 現在の OpenClaw の設定

コマンドオプション:

* `name`: コマンド名（先頭の `/` を除く）
* `description`: コマンド一覧に表示されるヘルプテキスト
* `acceptsArgs`: コマンドが引数を受け付けるかどうか（デフォルト: false）。false にもかかわらず引数が渡された場合、そのコマンドにはマッチせず、メッセージは他のハンドラーの処理に回される
* `requireAuth`: 認可済み送信者を必須とするかどうか（デフォルト: true）
* `handler`: `{ text: string }` を返す関数（非同期でも可）

認可と引数を使った例:

```ts
api.registerCommand({
  name: "setmode",
  description: "プラグインモードを設定",
  acceptsArgs: true,
  requireAuth: true,
  handler: async (ctx) => {
    const mode = ctx.args?.trim() || "default";
    await saveMode(mode);
    return { text: `Mode set to: ${mode}` };
  },
});
```

注意事項:

* プラグインのコマンドは、組み込みコマンドおよび AI エージェントよりも**先に**処理されます
* コマンドはグローバルに登録され、すべてのチャネルで有効になります
* コマンド名は大文字小文字を区別しません（`/MyStatus` は `/mystatus` にマッチします）
* コマンド名は英字で始まり、英字・数字・ハイフン・アンダースコアのみを含める必要があります
* `help`、`status`、`reset` などの予約済みコマンド名は、プラグインで上書きできません
* プラグイン間でコマンド登録が重複している場合は、診断エラーとなり登録に失敗します

<div id="register-background-services">
  ### バックグラウンドサービスの登録
</div>

```ts
export default function (api) {
  api.registerService({
    id: "my-service",
    start: () => api.logger.info("ready"),
    stop: () => api.logger.info("bye"),
  });
}
```

<div id="naming-conventions">
  ## 命名規約
</div>

* Gateway メソッド: `pluginId.action`（例: `voicecall.status`）
* ツール: `snake_case`（例: `voice_call`）
* CLI コマンド: ケバブケースまたはキャメルケース。ただし、コアコマンドと名前が衝突しないようにすること

<div id="skills">
  ## スキル
</div>

プラグインはリポジトリ内にスキルを含めることができます（`skills/<name>/SKILL.md`）。
`plugins.entries.<id>.enabled`（または他の設定ゲート）で有効化し、
ワークスペース／管理対象スキルの配置場所に置かれていることを確認してください。

<div id="distribution-npm">
  ## 配布（npm）
</div>

推奨されるパッケージ構成:

* メインパッケージ: `openclaw`（このリポジトリ）
* プラグイン: `@openclaw/*` 配下の個別 npm パッケージ（例: `@openclaw/voice-call`）

公開時の仕様:

* プラグインの `package.json` には、1 つ以上のエントリファイルを含む `openclaw.extensions` を必ず含めること。
* エントリファイルは `.js` または `.ts` を指定可能（jiti が実行時に TS を読み込む）。
* `openclaw plugins install <npm-spec>` は `npm pack` を使用し、`~/.openclaw/extensions/<id>/` に展開して、設定で有効化する。
* 設定キーの安定性: スコープ付きパッケージは、`plugins.entries.*` では **スコープなし** ID に正規化される。

<div id="example-plugin-voice-call">
  ## プラグイン例: Voice Call
</div>

このリポジトリには音声通話用のプラグイン（Twilio またはログ出力へのフォールバック実装）が含まれています:

* ソース: `extensions/voice-call`
* スキル: `skills/voice-call`
* CLI: `openclaw voicecall start|status`
* ツール: `voice_call`
* RPC: `voicecall.start`, `voicecall.status`
* 設定（Twilio）: `provider: "twilio"` + `twilio.accountSid/authToken/from`（オプションで `statusCallbackUrl`, `twimlUrl`）
* 設定（dev）: `provider: "log"`（外部ネットワーク通信なし）

セットアップと使用方法については、[Voice Call](/ja/plugins/voice-call) および `extensions/voice-call/README.md` を参照してください。

<div id="safety-notes">
  ## セキュリティ上の注意
</div>

プラグインは Gateway と同一プロセスで動作します。信頼できるコードとして扱ってください。

* 信頼できるプラグインのみをインストールしてください。
* `plugins.allow` の許可リストを優先して使用してください。
* 変更後は Gateway を再起動してください。

<div id="testing-plugins">
  ## プラグインのテスト
</div>

プラグインにはテストを含めることができます（推奨されます）：

* リポジトリ内プラグインは、Vitest のテストを `src/**` 配下に配置できます（例：`src/plugins/voice-call.plugin.test.ts`）。
* 別個に公開するプラグインは、自前の CI（lint/build/test）を実行し、`openclaw.extensions` がビルド済みのエントリポイント（`dist/index.js`）を指していることを検証する必要があります。