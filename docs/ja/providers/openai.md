---
title: OpenAI
summary: "OpenClaw から API キーまたは Codex サブスクリプション経由で OpenAI を利用する"
read_when:
  - OpenClaw で OpenAI のモデルを利用したい場合
  - API キーの代わりに Codex サブスクリプションによる認証を使いたい場合
---

<div id="openai">
  # OpenAI
</div>

OpenAI は GPT モデル用の開発者向け API を提供しています。Codex は、サブスクリプション
アクセス向けの **ChatGPT サインイン**、もしくは従量課金アクセス向けの **API キー** でのサインインの両方をサポートしています。Codex クラウドでは ChatGPT サインインが必須です。

<div id="option-a-openai-api-key-openai-platform">
  ## オプション A: OpenAI APIキー (OpenAI Platform)
</div>

**最適な用途:** 直接APIアクセスおよび従量課金での利用。
OpenAI ダッシュボードから自分のAPIキーを取得してください。

### CLI の設定

```bash
openclaw onboard --auth-choice openai-api-key
# または非対話モード
openclaw onboard --openai-api-key "$OPENAI_API_KEY"
```

<div id="config-snippet">
  ### 設定例
</div>

```json5
{
  env: { OPENAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "openai/gpt-5.2" } } }
}
```

<div id="option-b-openai-code-codex-subscription">
  ## オプションB: OpenAI Code (Codex) サブスクリプション
</div>

**最適な用途:** APIキーではなく、ChatGPT/Codex のサブスクリプション経由のアクセスを利用したい場合。
Codex クラウドでは ChatGPT でのサインインが必須ですが、Codex CLI は ChatGPT または APIキーによるサインインの両方をサポートします。

<div id="cli-setup">
  ### CLI のセットアップ
</div>

```bash
# ウィザードでCodex OAuthを実行する
openclaw onboard --auth-choice openai-codex

# Or run OAuth directly
openclaw models auth login --provider openai-codex
```

<div id="config-snippet">
  ### 設定例
</div>

```json5
{
  agents: { defaults: { model: { primary: "openai-codex/gpt-5.2" } } }
}
```

<div id="notes">
  ## 注意事項
</div>

* モデル参照は常に `provider/model` 形式を使用します（[/concepts/models](/ja/concepts/models) を参照）。
* 認証の詳細や再利用ルールについては [/concepts/oauth](/ja/concepts/oauth) を参照してください。