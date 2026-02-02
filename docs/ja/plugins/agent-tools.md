---
title: エージェントツール
summary: "プラグイン内でエージェントツールを実装する（スキーマ、オプション扱いのツール、許可リスト）"
read_when:
  - プラグインに新しいエージェントツールを追加したい場合
  - 許可リストでツールをオプトイン制にしたい場合
---

<div id="plugin-agent-tools">
  # プラグインエージェントツール
</div>

OpenClaw のプラグインは、エージェントの実行中に LLM から利用される **エージェントツール**（JSON Schema 関数）を登録できます。ツールは **必須**（常に利用可能）または
**任意**（オプトイン）のいずれかにできます。

エージェントツールは、メイン設定では `tools` の下に、エージェントごとでは
`agents.list[].tools` の下に定義します。allowlist/denylist（許可リスト/拒否リスト）ポリシーによって、そのエージェントがどのツールを呼び出せるかが制御されます。

<div id="basic-tool">
  ## 基本ツール
</div>

```ts
import { Type } from "@sinclair/typebox";

export default function (api) {
  api.registerTool({
    name: "my_tool",
    description: "Do a thing",
    parameters: Type.Object({
      input: Type.String(),
    }),
    async execute(_id, params) {
      return { content: [{ type: "text", text: params.input }] };
    },
  });
}
```


<div id="optional-tool-optin">
  ## オプションツール（任意利用）
</div>

オプションツールが自動で有効化されることは**決してありません**。ユーザーが明示的に、それらをエージェントの許可リストに追加する必要があります。

```ts
export default function (api) {
  api.registerTool(
    {
      name: "workflow_tool",
      description: "Run a local workflow",
      parameters: {
        type: "object",
        properties: {
          pipeline: { type: "string" },
        },
        required: ["pipeline"],
      },
      async execute(_id, params) {
        return { content: [{ type: "text", text: params.pipeline }] };
      },
    },
    { optional: true },
  );
}
```

オプションのツールを `agents.list[].tools.allow`（またはグローバルな `tools.allow`）で有効にします。

```json5
{
  agents: {
    list: [
      {
        id: "main",
        tools: {
          allow: [
            "workflow_tool",  // 特定のツール名
            "workflow",       // プラグインID(そのプラグインの全ツールを有効化)
            "group:plugins"   // 全プラグインツール
          ]
        }
      }
    ]
  }
}
```

ツールの利用可否に影響するその他の設定項目:

* プラグインツールだけが記載された許可リストは、プラグインのオプトインとして扱われます。コアツールは、
  許可リストにコアツールまたはグループも含めない限り有効なままです。
* `tools.profile` / `agents.list[].tools.profile` (基本の許可リスト)
* `tools.byProvider` / `agents.list[].tools.byProvider` (プロバイダー固有の許可/拒否)
* `tools.sandbox.tools.*` (サンドボックス化されている場合のサンドボックスツールポリシー)


<div id="rules-tips">
  ## ルールとヒント
</div>

- ツール名はコアツールの名前と衝突しては**いけません**。衝突したツールはスキップされます。
- 許可リストで使用するプラグインIDは、コアツールの名前と衝突してはなりません。
- 副作用を伴ったり、追加のバイナリや認証情報を必要とするツールでは、`optional: true` の設定を優先して使用してください。