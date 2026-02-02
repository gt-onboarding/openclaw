---
title: テスト
summary: "ローカル環境でテストを実行する方法（vitest）と、force/coverage モードを使うべきタイミング"
read_when:
  - テストの実行または修正を行うとき
---

<div id="tests">
  # テスト
</div>

* フルテストキット（テストスイート、ライブ、Docker）: [Testing](/ja/testing)

* `pnpm test:force`: 既定の制御ポートをつかんだまま残っている Gateway プロセスを強制終了し、その後、専用の Gateway ポートを使って Vitest のフルスイートを実行し、サーバーテストが起動中のインスタンスと衝突しないようにします。以前の Gateway 実行でポート 18789 が占有されたままの場合に使用します。

* `pnpm test:coverage`: V8 のカバレッジ付きで Vitest を実行します。グローバルなしきい値は、行・分岐・関数・ステートメントがそれぞれ 70% です。カバレッジ対象からは、統合度の高いエントリポイント（CLI 配線、gateway/telegram ブリッジ、webchat の静的サーバー）を除外し、ユニットテスト可能なロジックに焦点を当てています。

* `pnpm test:e2e`: Gateway のエンドツーエンドのスモークテストを実行します（マルチインスタンス WS/HTTP/ノードのペアリング）。

* `pnpm test:live`: プロバイダーのライブテスト（minimax/zai）を実行します。スキップを解除するには API キーと `LIVE=1`（またはプロバイダー固有の `*_LIVE_TEST=1`）が必要です。

<div id="model-latency-bench-local-keys">
  ## モデルレイテンシーベンチマーク（ローカルキー）
</div>

スクリプト: [`scripts/bench-model.ts`](https://github.com/openclaw/openclaw/blob/main/scripts/bench-model.ts)

使い方:

* `source ~/.profile && pnpm tsx scripts/bench-model.ts --runs 10`
* 任意の環境変数: `MINIMAX_API_KEY`, `MINIMAX_BASE_URL`, `MINIMAX_MODEL`, `ANTHROPIC_API_KEY`
* デフォルトプロンプト: 「1語だけで “ok” と返答してください。句読点やそれ以外のテキストは含めないでください。」

直近の実行結果（2025-12-31、20回実行）:

* minimax 中央値 1279ms（最小 1114、最大 2431）
* opus 中央値 2454ms（最小 1224、最大 3170）

<div id="onboarding-e2e-docker">
  ## オンボーディング E2E（Docker）
</div>

Docker はオプションです。これはコンテナ環境でのオンボーディング用スモークテストにのみ必要です。

クリーンな Linux コンテナでのフルコールドスタートフロー:

```bash
scripts/e2e/onboard-docker.sh
```

このスクリプトは疑似TTY経由で対話型ウィザードを操作し、config／ワークスペース／セッションに関連するファイルを検証してから、Gateway を起動して `openclaw health` を実行します。

<div id="qr-import-smoke-docker">
  ## QR インポートのスモークテスト (Docker)
</div>

Docker 上の Node.js 22 以降の環境で `qrcode-terminal` が正常にロードされることを確認します。

```bash
pnpm test:docker:qr
```
