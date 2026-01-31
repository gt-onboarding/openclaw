---
title: Bun
summary: "Bun ワークフロー（実験的）: インストールと pnpm との違い・ハマりどころ"
read_when:
  - 最速のローカル開発ループ（bun + watch）を求めているとき
  - Bun の install/patch/lifecycle スクリプト関連の問題に遭遇しているとき
---

<div id="bun-experimental">
  # Bun（実験的）
</div>

目標：pnpm のワークフローから逸脱せずに、このリポジトリを **Bun** で実行できるようにする（任意。WhatsApp/Telegram では非推奨）。

⚠️ **Gateway ランタイムには非推奨**（WhatsApp/Telegram で不具合あり）。本番環境では Node を使用してください。

<div id="status">
  ## ステータス
</div>

- Bun は、TypeScript を直接実行するためのオプションのローカルランタイムです（`bun run …`、`bun --watch …`）。
- ビルドのデフォルトは引き続き `pnpm` であり、完全にサポートされています（一部のドキュメント用ツールでも使用されています）。
- Bun は `pnpm-lock.yaml` を利用できないため、これを無視します。

<div id="install">
  ## インストール
</div>

デフォルト:

```sh
bun install
```

注記: `bun.lock`/`bun.lockb` は `.gitignore` に含まれているため、いずれにせよリポジトリに差分は発生しません。*ロックファイルを一切書き出したくない* 場合は:

```sh
bun install --no-save
```


<div id="build-test-bun">
  ## ビルド / テスト (Bun)
</div>

```sh
bun run build
bun run vitest run
```


<div id="bun-lifecycle-scripts-blocked-by-default">
  ## Bun ライフサイクルスクリプト（デフォルトでブロックされる）
</div>

Bun は、明示的に信頼されていない限り依存関係のライフサイクルスクリプトをブロックする場合があります（`bun pm untrusted` / `bun pm trust`）。
このリポジトリでは、よくブロック対象になるこれらのスクリプトは必須ではありません:

* `@whiskeysockets/baileys` `preinstall`: Node のメジャーバージョンが &gt;= 20 であることをチェックします（こちらは Node 22+ で実行しています）。
* `protobufjs` `postinstall`: 互換性のないバージョンスキームについての警告を出します（ビルド成果物はありません）。

これらのスクリプトが必要となる実際のランタイム問題に遭遇した場合は、それらを明示的に信頼済みに設定してください:

```sh
bun pm trust @whiskeysockets/baileys protobufjs
```


<div id="caveats">
  ## 注意事項
</div>

- 一部のスクリプトでは依然として pnpm がハードコードされています（`docs:build`、`ui:*`、`protocol:check` など）。当面はそれらは pnpm で実行してください。