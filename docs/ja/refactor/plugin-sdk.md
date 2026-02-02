---
title: プラグインSDK
summary: "方針：すべてのメッセージングコネクタ向けに、クリーンな単一のプラグインSDKとランタイムを用意する"
read_when:
  - プラグインアーキテクチャを定義またはリファクタリングするとき
  - チャネルコネクタをプラグインSDK／ランタイムへ移行するとき
---

<div id="plugin-sdk-runtime-refactor-plan">
  # プラグイン SDK とランタイムのリファクタリング計画
</div>

目標：すべてのメッセージングコネクタを、単一の安定した api を使用するプラグイン（バンドル済みまたは外部）にする。
いずれのプラグインも `src/**` を直接 import しない。すべての依存関係は SDK またはランタイム経由とする。

<div id="why-now">
  ## なぜ今なのか
</div>

* 現在のコネクタは、core への直接 import、dist 専用ブリッジ、カスタムヘルパーといった複数のパターンが混在している。
* その結果、アップグレードに弱くなり、クリーンな外部公開用プラグインインターフェースの実現を妨げている。

<div id="target-architecture-two-layers">
  ## ターゲットアーキテクチャ（2レイヤー構成）
</div>

<div id="1-plugin-sdk-compile-time-stable-publishable">
  ### 1) Plugin SDK (コンパイル時専用、安定版、公開可能)
</div>

スコープ: 型、ヘルパー、および設定ユーティリティ。ランタイム状態なし、副作用なし。

内容（例）:

* 型: `ChannelPlugin`、アダプタ、`ChannelMeta`、`ChannelCapabilities`、`ChannelDirectoryEntry`。
* 設定ヘルパー: `buildChannelConfigSchema`、`setAccountEnabledInConfigSection`、`deleteAccountFromConfigSection`、
  `applyAccountNameToChannelSection`。
* ペアリング用ヘルパー: `PAIRING_APPROVED_MESSAGE`、`formatPairingApproveHint`。
* オンボーディング用ヘルパー: `promptChannelAccessConfig`、`addWildcardAllowFrom`、オンボーディング関連の型。
* ツールのパラメータ用ヘルパー: `createActionGate`、`readStringParam`、`readNumberParam`、`readReactionParams`、`jsonResult`。
* ドキュメントリンク用ヘルパー: `formatDocsLink`。

提供:

* `openclaw/plugin-sdk` として公開（またはコアから `openclaw/plugin-sdk` 配下でエクスポート）。
* SemVer を採用し、明示的な安定性保証を行う。

<div id="2-plugin-runtime-execution-surface-injected">
  ### 2) プラグインランタイム（実行サーフェス、インジェクト）
</div>

スコープ: コアのランタイム動作に関わるすべて。
プラグインが決して `src/**` を import しないよう、`OpenClawPluginApi.runtime` 経由でアクセスする。

提案するサーフェス（最小限だが完全）:

```ts
export type PluginRuntime = {
  channel: {
    text: {
      chunkMarkdownText(text: string, limit: number): string[];
      resolveTextChunkLimit(cfg: OpenClawConfig, channel: string, accountId?: string): number;
      hasControlCommand(text: string, cfg: OpenClawConfig): boolean;
    };
    reply: {
      dispatchReplyWithBufferedBlockDispatcher(params: {
        ctx: unknown;
        cfg: unknown;
        dispatcherOptions: {
          deliver: (payload: { text?: string; mediaUrls?: string[]; mediaUrl?: string }) =>
            void | Promise<void>;
          onError?: (err: unknown, info: { kind: string }) => void;
        };
      }): Promise<void>;
      createReplyDispatcherWithTyping?: unknown; // Teams スタイルフロー用のアダプター
    };
    routing: {
      resolveAgentRoute(params: {
        cfg: unknown;
        channel: string;
        accountId: string;
        peer: { kind: "dm" | "group" | "channel"; id: string };
      }): { sessionKey: string; accountId: string };
    };
    pairing: {
      buildPairingReply(params: { channel: string; idLine: string; code: string }): string;
      readAllowFromStore(channel: string): Promise<string[]>;
      upsertPairingRequest(params: {
        channel: string;
        id: string;
        meta?: { name?: string };
      }): Promise<{ code: string; created: boolean }>;
    };
    media: {
      fetchRemoteMedia(params: { url: string }): Promise<{ buffer: Buffer; contentType?: string }>;
      saveMediaBuffer(
        buffer: Uint8Array,
        contentType: string | undefined,
        direction: "inbound" | "outbound",
        maxBytes: number,
      ): Promise<{ path: string; contentType?: string }>;
    };
    mentions: {
      buildMentionRegexes(cfg: OpenClawConfig, agentId?: string): RegExp[];
      matchesMentionPatterns(text: string, regexes: RegExp[]): boolean;
    };
    groups: {
      resolveGroupPolicy(cfg: OpenClawConfig, channel: string, accountId: string, groupId: string): {
        allowlistEnabled: boolean;
        allowed: boolean;
        groupConfig?: unknown;
        defaultConfig?: unknown;
      };
      resolveRequireMention(
        cfg: OpenClawConfig,
        channel: string,
        accountId: string,
        groupId: string,
        override?: boolean,
      ): boolean;
    };
    debounce: {
      createInboundDebouncer<T>(opts: {
        debounceMs: number;
        buildKey: (v: T) => string | null;
        shouldDebounce: (v: T) => boolean;
        onFlush: (entries: T[]) => Promise<void>;
        onError?: (err: unknown) => void;
      }): { push: (v: T) => void; flush: () => Promise<void> };
      resolveInboundDebounceMs(cfg: OpenClawConfig, channel: string): number;
    };
    commands: {
      resolveCommandAuthorizedFromAuthorizers(params: {
        useAccessGroups: boolean;
        authorizers: Array<{ configured: boolean; allowed: boolean }>;
      }): boolean;
    };
  };
  logging: {
    shouldLogVerbose(): boolean;
    getChildLogger(name: string): PluginLogger;
  };
  state: {
    resolveStateDir(cfg: OpenClawConfig): string;
  };
};
```

メモ:

* Runtime はコアの挙動にアクセスするための唯一の手段です。
* SDK は意図的に小さく、安定したものに保たれています。
* 各 Runtime メソッドは既存のコア実装に 1 対 1 で対応します（重複はありません）。

<div id="migration-plan-phased-safe">
  ## 移行計画（段階的かつ安全）
</div>

<div id="phase-0-scaffolding">
  ### フェーズ 0: 準備 (scaffolding)
</div>

* `openclaw/plugin-sdk` を導入する。
* 上記のインターフェースを持つ `api.runtime` を `OpenClawPluginApi` に追加する。
* 既存の import は移行期間中も維持しつつ、非推奨警告を出す。

<div id="phase-1-bridge-cleanup-low-risk">
  ### フェーズ 1: ブリッジのクリーンアップ（低リスク）
</div>

* 拡張機能単位の `core-bridge.ts` を `api.runtime` に置き換える。
* BlueBubbles、Zalo、Zalo Personal から先に移行する（すでにほぼ対応済み）。
* 重複しているブリッジコードを削除する。

<div id="phase-2-light-direct-import-plugins">
  ### フェーズ 2: 軽量な直接インポート型プラグイン
</div>

* Matrix を SDK とランタイムに移行する。
* オンボーディング、ディレクトリ、グループメンションのロジックを検証する。

<div id="phase-3-heavy-direct-import-plugins">
  ### フェーズ 3: ヘビーな direct-import プラグイン
</div>

* MS Teams を移行する（runtime helper を最も多く含む）。
* reply/typing のセマンティクスが現行の挙動と一致していることを保証する。

<div id="phase-4-imessage-pluginization">
  ### フェーズ 4: iMessage のプラグイン化
</div>

* iMessage を `extensions/imessage` に移動する。
* コアへの直接呼び出しを `api.runtime` に置き換える。
* 設定キー、CLI の動作、ドキュメントは既存どおりに保つ。

<div id="phase-5-enforcement">
  ### フェーズ 5: ルール適用
</div>

* Lint ルール / CI チェックを追加する: `src/**` からの `extensions/**` インポートを禁止する。
* プラグイン SDK / バージョン互換性チェックを追加する（ランタイムおよび SDK の semver）。

<div id="compatibility-and-versioning">
  ## 互換性とバージョニング
</div>

* SDK: SemVer によるバージョン管理で公開し、変更点を文書化する。
* Runtime: コアの各リリースごとにバージョン付けする。`api.runtime.version` を追加する。
* プラグインは必要な runtime のバージョン範囲を宣言する（例: `openclawRuntime: ">=2026.2.0"`）。

<div id="testing-strategy">
  ## テスト戦略
</div>

* アダプターレベルのユニットテスト（実際のコア実装を用いてランタイム関数を実行）。
* プラグインごとのゴールデンテスト：挙動に乖離がないことを確認（ルーティング、ペアリング、許可リスト、メンションによるゲート制御）。
* CI で使用するエンドツーエンドのプラグインサンプルを 1 つ用意（インストール + 実行 + スモークテスト）。

<div id="open-questions">
  ## 未解決の検討事項
</div>

* SDK の型定義をどこで提供するか：別パッケージか、core からのエクスポートか？
* Runtime 用の型をどこで配布するか：SDK（型のみ）か、core か？
* バンドル済みプラグインと外部プラグイン向けに、ドキュメントへのリンクをどのように提供するか？
* 移行期間中、リポジトリ内のプラグインに対して、限定的に core を直接 import することを許可するか？

<div id="success-criteria">
  ## 成功条件
</div>

* すべてのチャネルコネクタが、SDK とランタイムを利用するプラグインであること。
* `src/**` から `extensions/**` へのインポート文が存在しないこと。
* 新しいコネクタテンプレートは SDK とランタイムのみに依存すること。
* 外部プラグインは、コアソースコードへのアクセスなしで開発・更新可能であること。

関連ドキュメント: [Plugins](/ja/plugin), [Channels](/ja/channels/index), [Configuration](/ja/gateway/configuration).