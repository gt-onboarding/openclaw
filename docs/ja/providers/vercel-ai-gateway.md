---
title: "Vercel AI Gateway"
summary: "Vercel AI Gateway のセットアップ（認証とモデル選択）"
read_when:
  - OpenClaw で Vercel AI Gateway を使いたい場合
  - API キー用の環境変数や CLI の認証方法を知りたい場合
---

<div id="vercel-ai-gateway">
  # Vercel AI Gateway
</div>

[Vercel AI Gateway](https://vercel.com/ai-gateway) は、単一のエンドポイントから数百におよぶモデルへアクセスできる統合 API を提供します。 

- プロバイダー: `vercel-ai-gateway`
- 認証: `AI_GATEWAY_API_KEY`
- API: Anthropic Messages 互換

<div id="quick-start">
  ## クイックスタート
</div>

1. API キーを設定します（推奨：Gateway 用として保存しておく）：

```bash
openclaw onboard --auth-choice ai-gateway-api-key
```

2. デフォルトのモデルを設定します:

```json5
{
  agents: {
    defaults: {
      model: { primary: "vercel-ai-gateway/anthropic/claude-opus-4.5" }
    }
  }
}
```


<div id="non-interactive-example">
  ## 非インタラクティブな例
</div>

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice ai-gateway-api-key \
  --ai-gateway-api-key "$AI_GATEWAY_API_KEY"
```


<div id="environment-note">
  ## 環境に関する注意
</div>

Gateway をデーモン（launchd/systemd）として実行する場合は、そのプロセスから `AI_GATEWAY_API_KEY`
が利用できるようにしておいてください（たとえば `~/.openclaw/.env` に設定するか、
`env.shellEnv` 経由で渡します）。