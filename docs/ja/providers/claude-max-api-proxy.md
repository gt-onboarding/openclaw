---
title: Claude Max API プロキシ
summary: "Claude Max/Pro サブスクリプションを OpenAI 互換の API エンドポイントとして利用する"
read_when:
  - OpenAI 互換ツールで Claude Max サブスクリプションを利用したい
  - Claude Code CLI をラップするローカルの API サーバーがほしい
  - API キーではなくサブスクリプションを使ってコストを削減したい
---

<div id="claude-max-api-proxy">
  # Claude Max API Proxy
</div>

**claude-max-api-proxy** は、Claude Max/Pro サブスクリプションを OpenAI 互換の API エンドポイントとして公開する、コミュニティが開発したツールです。これにより、OpenAI API 形式をサポートする任意のツールでサブスクリプションを利用できるようになります。

<div id="why-use-this">
  ## なぜこれを使うのか？
</div>

| アプローチ | コスト | 最適な用途 |
|----------|------|----------|
| Anthropic API | トークンごとの従量課金（Opus で入力 100 万トークンあたり約 $15、出力 100 万トークンあたり約 $75） | 本番環境のアプリケーション、大規模トラフィック |
| Claude Max サブスクリプション | 月額 $200 の定額 | 個人利用、開発用途、使用量無制限 |

Claude Max のサブスクリプションをお持ちで、それを OpenAI 互換ツールで使いたい場合、このプロキシを利用することでコストを大幅に削減できます。

<div id="how-it-works">
  ## 動作の仕組み
</div>

```
あなたのアプリ → claude-max-api-proxy → Claude Code CLI → Anthropic (サブスクリプション経由)
     (OpenAI形式)              (形式変換)      (ログイン情報を使用)
```

このプロキシは次の処理を行います:

1. `http://localhost:3456/v1/chat/completions` で OpenAI 形式のリクエストを受け付ける
2. リクエストを Claude Code CLI コマンドに変換する
3. OpenAI 形式でレスポンスを返す（ストリーミング対応）

<div id="installation">
  ## インストール
</div>

```bash
# Node.js 20+ および Claude Code CLI が必要です
npm install -g claude-max-api-proxy

# Verify Claude CLI is authenticated
claude --version
```

<div id="usage">
  ## 使い方
</div>

<div id="start-the-server">
  ### サーバーを起動する
</div>

```bash
claude-max-api
# サーバーは http://localhost:3456 で起動します
```

<div id="test-it">
  ### テストする
</div>

```bash
# ヘルスチェック
curl http://localhost:3456/health

# モデル一覧
curl http://localhost:3456/v1/models

# チャット補完
curl http://localhost:3456/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "claude-opus-4",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

<div id="with-openclaw">
  ### OpenClaw での利用
</div>

OpenClaw で、このプロキシをカスタムの OpenAI 互換エンドポイントとして指定できます：

```json5
{
  env: {
    OPENAI_API_KEY: "not-needed",
    OPENAI_BASE_URL: "http://localhost:3456/v1"
  },
  agents: {
    defaults: {
      model: { primary: "openai/claude-opus-4" }
    }
  }
}
```

<div id="available-models">
  ## 利用可能なモデル
</div>

| モデル ID | 対応先 |
|----------|---------|
| `claude-opus-4` | Claude Opus 4 |
| `claude-sonnet-4` | Claude Sonnet 4 |
| `claude-haiku-4` | Claude Haiku 4 |

<div id="auto-start-on-macos">
  ## macOS での自動起動
</div>

プロキシを自動的に起動する LaunchAgent を作成します：

```bash
cat > ~/Library/LaunchAgents/com.claude-max-api.plist << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key>
  <string>com.claude-max-api</string>
  <key>RunAtLoad</key>
  <true/>
  <key>KeepAlive</key>
  <true/>
  <key>ProgramArguments</key>
  <array>
    <string>/usr/local/bin/node</string>
    <string>/usr/local/lib/node_modules/claude-max-api-proxy/dist/server/standalone.js</string>
  </array>
  <key>EnvironmentVariables</key>
  <dict>
    <key>PATH</key>
    <string>/usr/local/bin:/opt/homebrew/bin:~/.local/bin:/usr/bin:/bin</string>
  </dict>
</dict>
</plist>
EOF

launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.claude-max-api.plist
```

<div id="links">
  ## リンク
</div>

* **npm：** https://www.npmjs.com/package/claude-max-api-proxy
* **GitHub：** https://github.com/atalovesyou/claude-max-api-proxy
* **Issues：** https://github.com/atalovesyou/claude-max-api-proxy/issues

<div id="notes">
  ## 注意事項
</div>

* これは **コミュニティ製ツール** であり、Anthropic や OpenClaw による公式サポートの対象ではありません
* 有効な Claude Max/Pro サブスクリプションと、認証済みの Claude Code CLI が必要です
* プロキシはローカルで動作し、第三者のサーバーへデータを送信しません
* ストリーミングレスポンスに完全対応しています

<div id="see-also">
  ## 関連項目
</div>

* [Anthropic プロバイダー](/ja/providers/anthropic) - Claude 用 setup-token または API キーによる OpenClaw ネイティブ統合
* [OpenAI プロバイダー](/ja/providers/openai) - OpenAI/Codex サブスクリプション用