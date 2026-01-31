---
title: Node の問題
summary: Node と tsx による "__name is not a function" クラッシュに関するメモと回避策
read_when:
  - Node 専用の開発スクリプトや watch モードの失敗をデバッグするとき
  - OpenClaw での tsx/esbuild ローダーのクラッシュを調査するとき
---

<div id="node-tsx-__name-is-not-a-function-crash">
  # Node + tsx "__name is not a function" クラッシュ
</div>

<div id="summary">
  ## 概要
</div>

Node.js 上で `tsx` を使用して OpenClaw を実行すると、起動時に次のエラーが発生して失敗します：

```
[openclaw] Failed to start CLI: TypeError: __name is not a function
    at createSubsystemLogger (.../src/logging/subsystem.ts:203:25)
    at .../src/agents/auth-profiles/constants.ts:25:20
```

これは、開発用スクリプトを Bun から `tsx` に切り替えた後（コミット `2871657e`, 2026-01-06）に発生し始めました。同じ実行パスは Bun では問題なく動作していました。


<div id="environment">
  ## 環境
</div>

- Node: v25.x（v25.3.0 で発生を確認）
- tsx: 4.21.0
- OS: macOS（Node 25 が動作している他のプラットフォームでも再現する可能性が高い）

<div id="repro-node-only">
  ## ノードのみでの再現手順
</div>

```bash
# リポジトリルートで
node --version
pnpm install
node --import tsx src/entry.ts status
```


<div id="minimal-repro-in-repo">
  ## リポジトリ内の最小再現例
</div>

```bash
node --import tsx scripts/repro/tsx-name-repro.ts
```


<div id="node-version-check">
  ## Node バージョンチェック
</div>

- Node 25.3.0: 失敗
- Node 22.22.0 (Homebrew `node@22`): 失敗
- Node 24: まだここにはインストールされていないため、要検証

<div id="notes-hypothesis">
  ## メモ / 仮説
</div>

- `tsx` は TS/ESM コードを変換するために esbuild を使用する。esbuild の `keepNames` は `__name` ヘルパーを出力し、関数定義を `__name(...)` でラップする。
- このクラッシュは、実行時に `__name` が存在しているにもかかわらず関数ではないことを示しており、Node 25 のローダーパスにおいて、このモジュール用のヘルパーが欠落しているか、上書きされていることを示唆している。
- ヘルパーが欠落している、または書き換えられているケースについては、他の esbuild の利用者でも同様の `__name` ヘルパーに関する問題が報告されている。

<div id="regression-history">
  ## リグレッション履歴
</div>

- `2871657e` (2026-01-06): script を Bun から tsx に変更し、Bun をオプション扱いにした。
- それ以前（Bun を利用するパス）では、`openclaw status` と `gateway:watch` は動作していた。

<div id="workarounds">
  ## 回避策
</div>

- 開発用スクリプトには Bun を使用する（現状の一時的なロールバック手段）。
- Node + tsc のウォッチを使い、その後コンパイル済みの出力を実行する:
  ```bash
  pnpm exec tsc --watch --preserveWatchOutput
  node --watch openclaw.mjs status
  ```
- ローカルで確認済み: `pnpm exec tsc -p tsconfig.json` と `node openclaw.mjs status` は Node 25 上で動作する。
- 可能であれば、TS ローダーで esbuild の keepNames を無効化する（`__name` ヘルパーの挿入を防ぐ）。tsx は現在、この設定を露出していない。
- Node LTS (22/24) 環境で `tsx` を使ってテストし、この問題が Node 25 固有かどうかを確認する。

<div id="references">
  ## 参考資料
</div>

- https://opennext.js.org/cloudflare/howtos/keep_names
- https://esbuild.github.io/api/#keep-names
- https://github.com/evanw/esbuild/issues/1031

<div id="next-steps">
  ## 次のステップ
</div>

- Node 22/24 上で再現させて、Node 25 でのリグレッションであることを確認する。
- 既知のリグレッションがある場合は、`tsx` の nightly 版を試すか、以前のバージョンにバージョン固定する。
- Node LTS 上で再現する場合は、`__name` スタックトレース付きの最小再現ケースを upstream に報告する。