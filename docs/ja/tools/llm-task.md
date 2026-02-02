---
title: LLM タスク
summary: "ワークフロー向けの JSON 専用 LLM タスク（オプションのプラグインツール）"
read_when:
  - ワークフロー内に JSON 専用の LLM ステップを追加したい場合
  - 自動化のためにスキーマ検証済みの LLM 出力が必要な場合
---

<div id="llm-task">
  # LLMタスク
</div>

`llm-task` は、JSON 専用の LLM タスクを実行し、
構造化された出力（必要に応じて JSON Schema による検証付き）を返す **オプションのプラグインツール** です。

これは Lobster のようなワークフローエンジンに最適です。各ワークフローごとにカスタムの OpenClaw コードを書くことなく、1つの LLM ステップを追加できます。

<div id="enable-the-plugin">
  ## プラグインを有効にする
</div>

1. プラグインを有効にします。

```json
{
  "plugins": {
    "entries": {
      "llm-task": { "enabled": true }
    }
  }
}
```

2. ツールを許可リストに追加します（これは `optional: true` で登録されています）:

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "tools": { "allow": ["llm-task"] }
      }
    ]
  }
}
```


<div id="config-optional">
  ## 設定（任意）
</div>

```json
{
  "plugins": {
    "entries": {
      "llm-task": {
        "enabled": true,
        "config": {
          "defaultProvider": "openai-codex",
          "defaultModel": "gpt-5.2",
          "defaultAuthProfileId": "main",
          "allowedModels": ["openai-codex/gpt-5.2"],
          "maxTokens": 800,
          "timeoutMs": 30000
        }
      }
    }
  }
}
```

`allowedModels` は `provider/model` 文字列の許可リストです。設定されている場合、このリストに含まれないリクエストはすべて拒否されます。


<div id="tool-parameters">
  ## ツールのパラメータ
</div>

- `prompt` (string、必須)
- `input` (any、任意)
- `schema` (object、任意の JSON Schema)
- `provider` (string、任意)
- `model` (string、任意)
- `authProfileId` (string、任意)
- `temperature` (number、任意)
- `maxTokens` (number、任意)
- `timeoutMs` (number、任意)

<div id="output">
  ## 出力
</div>

解析済みの JSON を含む `details.json` を返します（`schema` が指定されている場合は、それに対して検証も行います）。

<div id="example-lobster-workflow-step">
  ## 例: Lobster ワークフロー ステップ
</div>

```lobster
openclaw.invoke --tool llm-task --action json --args-json '{
  "prompt": "Given the input email, return intent and draft.",
  "input": {
    "subject": "Hello",
    "body": "Can you help?"
  },
  "schema": {
    "type": "object",
    "properties": {
      "intent": { "type": "string" },
      "draft": { "type": "string" }
    },
    "required": ["intent", "draft"],
    "additionalProperties": false
  }
}'
```


<div id="safety-notes">
  ## 安全上の注意事項
</div>

- このツールは **JSON のみ** を扱い、モデルには JSON 以外を出力しないよう指示します（コードフェンスなし、コメントなし）。
- この実行では、モデルにはいかなるツールも利用可能にされません。
- `schema` で検証しない限り、出力は信頼できないものとして扱ってください。
- 副作用を伴うあらゆるステップ（送信、post、exec）の前に承認を取得してください。