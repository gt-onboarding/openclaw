---
title: リリース手順
summary: "npm および macOS アプリ向けのステップバイステップなリリースチェックリスト"
read_when:
  - 新しい npm リリースを作成するとき
  - 新しい macOS アプリのリリースを作成するとき
  - 公開前にメタデータを確認するとき
---

<div id="release-checklist-npm-macos">
  # リリースチェックリスト（npm + macOS）
</div>

リポジトリのルートで `pnpm`（Node 22 以上）を使用してください。タグ付け／公開の前に、作業ツリーをクリーンな状態に保ってください。

<div id="operator-trigger">
  ## オペレーター・トリガー
</div>

オペレーターが「release」と言ったら、すぐに次の事前チェックを実行する（ブロックされた場合を除き追加の質問はしない）:

* このドキュメントと `docs/platforms/mac/release.md` を読む。
* `~/.profile` から環境変数を読み込み、`SPARKLE_PRIVATE_KEY_FILE` と App Store Connect 関連の変数が設定されていることを確認する（`SPARKLE_PRIVATE_KEY_FILE` は `~/.profile` に存在している必要がある）。
* 必要に応じて `~/Library/CloudStorage/Dropbox/Backup/Sparkle` の Sparkle の鍵を使用する。

1. **バージョン &amp; メタデータ**

* [ ] `package.json` のバージョンを更新する（例: `2026.1.29`）。
* [ ] `pnpm plugins:sync` を実行し、拡張パッケージのバージョンと changelog を同期する。
* [ ] CLI/バージョン文字列を更新する: [`src/cli/program.ts`](https://github.com/openclaw/openclaw/blob/main/src/cli/program.ts) と、[`src/provider-web.ts`](https://github.com/openclaw/openclaw/blob/main/src/provider-web.ts) 内の Baileys user agent。
* [ ] パッケージのメタデータ（name, description, repository, keywords, license）を確認し、`bin` マップが `openclaw` に対して [`openclaw.mjs`](https://github.com/openclaw/openclaw/blob/main/openclaw.mjs) を指していることを確認する。
* [ ] 依存関係を変更した場合は `pnpm install` を実行し、`pnpm-lock.yaml` を最新にする。

2. **ビルド &amp; アーティファクト**

* [ ] A2UI の入力が変わった場合、`pnpm canvas:a2ui:bundle` を実行し、更新された [`src/canvas-host/a2ui/a2ui.bundle.js`](https://github.com/openclaw/openclaw/blob/main/src/canvas-host/a2ui/a2ui.bundle.js) をコミットする。
* [ ] `pnpm run build`（`dist/` を再生成する）。
* [ ] npm パッケージの `files` に必要な `dist/*` フォルダ（特にヘッドレスノード + ACP CLI 用の `dist/node-host/**` と `dist/acp/**`）がすべて含まれていることを確認する。
* [ ] `dist/build-info.json` が存在し、期待される `commit` ハッシュを含んでいることを確認する（CLI バナーは npm インストール時にこれを使用する）。
* [ ] 任意: ビルド後に `npm pack --pack-destination /tmp` を実行し、生成された tarball の内容を確認して GitHub リリース用に手元に保持する（**コミットはしないこと**）。

3. **Changelog &amp; ドキュメント**

* [ ] `CHANGELOG.md` をユーザー向けのハイライトで更新する（ファイルがない場合は作成する）。エントリはバージョンの降順のみで並べる。
* [ ] README の例やフラグが現在の CLI の挙動と一致していることを確認する（特に新しいコマンドやオプション）。

4. **検証**

* [ ] `pnpm lint`
* [ ] `pnpm test`（カバレッジ出力が必要な場合は `pnpm test:coverage`）
* [ ] `pnpm run build`（テスト後の最終サニティチェック）
* [ ] `pnpm release:check`（`npm pack` の内容を検証）
* [ ] `OPENCLAW_INSTALL_SMOKE_SKIP_NONROOT=1 pnpm test:install:smoke`（Docker インストールのスモークテスト／高速パス。リリース前に必須）
  * 直前の npm リリースが壊れていることが分かっている場合は、プリインストールステップ用に `OPENCLAW_INSTALL_SMOKE_PREVIOUS=<last-good-version>` を設定するか、`OPENCLAW_INSTALL_SMOKE_SKIP_PREVIOUS=1` を設定する。
* [ ] （任意）フルインストーラーのスモークテスト（非 root + CLI カバレッジを追加）: `pnpm test:install:smoke`
* [ ] （任意）インストーラー E2E（Docker 上で `curl -fsSL https://openclaw.bot/install.sh | bash` を実行し、オンボードしてから実際のツール呼び出しを実行）:
  * `pnpm test:install:e2e:openai`（`OPENAI_API_KEY` が必要）
  * `pnpm test:install:e2e:anthropic`（`ANTHROPIC_API_KEY` が必要）
  * `pnpm test:install:e2e`（両方のキーが必要。両方のプロバイダーを実行）
* [ ] （任意）変更内容が送受信パスに影響する場合は、Web Gateway をスポットチェックする。

5. **macOS アプリ（Sparkle）**

* [ ] macOS アプリをビルドして署名し、配布用に zip します。
* [ ] Sparkle 用の appcast を生成し（[`scripts/make_appcast.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/make_appcast.sh) 経由で HTML のリリースノートを作成）、`appcast.xml` を更新します。
* [ ] アプリの zip（および任意で dSYM の zip）を GitHub リリースに添付できるよう準備しておきます。
* [ ] 正確なコマンドと必要な環境変数については、[macOS release](/ja/platforms/mac/release) に従います。
  * `APP_BUILD` は数値かつ単調増加である必要があります（`-beta` は不可）。そうしないと Sparkle がバージョンを正しく比較できません。
  * 公証（notarization）を行う場合は、App Store Connect API の環境変数から作成した `openclaw-notary` キーチェーンプロファイルを使用します（[macOS release](/ja/platforms/mac/release) を参照）。

6. **Publish (npm)**

* [ ] git のステータスがクリーンであることを確認し、必要に応じてコミットとプッシュを行います。
* [ ] 必要に応じて `npm login` を実行し、2FA を含めログイン状態を確認します。
* [ ] `npm publish --access public` を実行します（プレリリースには `--tag beta` を使用します）。
* [ ] 次を使ってレジストリを確認します: `npm view openclaw version`, `npm view openclaw dist-tags`, および `npx -y openclaw@X.Y.Z --version`（または `--help`）。

<div id="troubleshooting-notes-from-200-beta2-release">
  ### トラブルシューティング（2.0.0-beta2 リリース時のメモ）
</div>

* **`npm pack/publish` がハングする、または巨大な tarball が生成される**: `dist/OpenClaw.app`（およびリリース zip）の macOS アプリバンドルがパッケージに含まれてしまいます。`package.json` の `files` で公開対象をホワイトリスト指定することで修正します（`dist` のサブディレクトリ、docs、スキルを含め、アプリバンドルを除外する）。`npm pack --dry-run` を実行し、`dist/OpenClaw.app` が一覧に含まれていないことを確認します。
* **dist-tag 用の npm 認証が Web 認証ループになる**: レガシー認証を使って OTP プロンプトを表示させます:
  * `NPM_CONFIG_AUTH_TYPE=legacy npm dist-tag add openclaw@X.Y.Z latest`
* **`npx` 検証が `ECOMPROMISED: Lock compromised` で失敗する**: 新しいキャッシュで再試行します:
  * `NPM_CONFIG_CACHE=/tmp/npm-cache-$(date +%s) npx -y openclaw@X.Y.Z --version`
* **遅れて入れた修正のあとにタグを指し直す必要がある**: タグを強制更新して push し、その後も GitHub リリースのアセットが一致していることを確認します:
  * `git tag -f vX.Y.Z && git push -f origin vX.Y.Z`

7. **GitHub リリース + appcast**

* [ ] タグ付けと push を行う: `git tag vX.Y.Z && git push origin vX.Y.Z`（または `git push --tags`）。
* [ ] `vX.Y.Z` 用の GitHub リリースを作成／更新し、**タイトルを `openclaw X.Y.Z`**（タグ名だけではなく）とする。本文にはそのバージョンの **完全な** changelog セクション（Highlights + Changes + Fixes）をインラインで含め（素のリンクだけにはせず）、**本文中でタイトルを繰り返さないこと**。
* [ ] アーティファクトを添付する: `npm pack` の tarball（任意）、`OpenClaw-X.Y.Z.zip`、および `OpenClaw-X.Y.Z.dSYM.zip`（生成されていれば）。
* [ ] 更新した `appcast.xml` をコミットして push する（Sparkle は main からフィードされる）。
* [ ] クリーンな一時ディレクトリ（`package.json` がない状態）から `npx -y openclaw@X.Y.Z send --help` を実行し、インストール／CLI エントリポイントが動作することを確認する。
* [ ] リリースノートをアナウンス／共有する。

<div id="plugin-publish-scope-npm">
  ## プラグイン公開スコープ (npm)
</div>

`@openclaw/*` スコープの **既存の npm プラグイン** のみを公開します。npm 上にない
同梱プラグインは公開せず、**ディスクツリーのみ** に残します（引き続き
`extensions/**` に同梱されます）。

リストを作成する手順:

1. `npm search @openclaw --json` を実行し、パッケージ名を取得します。
2. `extensions/*/package.json` の name と比較します。
3. **共通部分**（すでに npm にあるもの）だけを公開します。

現在の npm プラグイン一覧（必要に応じて更新すること）:

* @openclaw/bluebubbles
* @openclaw/diagnostics-otel
* @openclaw/discord
* @openclaw/lobster
* @openclaw/matrix
* @openclaw/msteams
* @openclaw/nextcloud-talk
* @openclaw/nostr
* @openclaw/voice-call
* @openclaw/zalo
* @openclaw/zalouser

リリースノートでは、**新しいオプション同梱プラグイン** のうち、**デフォルトで有効ではないもの**
（例: `tlon`）も必ず記載してください。