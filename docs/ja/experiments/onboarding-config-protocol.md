---
title: Onboarding Config プロトコル
summary: "オンボーディングウィザードおよび config スキーマ向け RPC プロトコルに関するメモ"
read_when: "オンボーディングウィザードのステップまたは config スキーマのエンドポイントを変更する場合"
---

<div id="onboarding-config-protocol">
  # オンボーディングと設定プロトコル
</div>

目的: CLI、macOS アプリ、Web UI にまたがって共通のオンボーディングおよび設定用インターフェースを提供すること。

<div id="components">
  ## コンポーネント
</div>

- Wizard エンジン（共有セッション + プロンプト + オンボーディング状態で構成される）。
- CLI のオンボーディングでは、UI クライアントと同じ Wizard フローを使用する。
- Gateway RPC は、Wizard および設定スキーマのエンドポイントを公開する。
- macOS のオンボーディングでは、Wizard のステップモデルを使用する。
- Web UI は、JSON Schema + UI ヒントから設定フォームをレンダリングする。

<div id="gateway-rpc">
  ## Gateway RPC
</div>

- `wizard.start` のパラメータ: `{ mode?: "local"|"remote", workspace?: string }`
- `wizard.next` のパラメータ: `{ sessionId, answer?: { stepId, value? } }`
- `wizard.cancel` のパラメータ: `{ sessionId }`
- `wizard.status` のパラメータ: `{ sessionId }`
- `config.schema` のパラメータ: `{}`

レスポンス（形式）

- Wizard: `{ sessionId, done, step?, status?, error? }`
- 設定スキーマ: `{ schema, uiHints, version, generatedAt }`

<div id="ui-hints">
  ## UI ヒント
</div>

- パスをキーとした `uiHints`。任意のメタデータ（label/help/group/order/advanced/sensitive/placeholder）を指定できる。
- Sensitive なフィールドはパスワード入力としてレンダリングされるが、別途のマスキングレイヤーは存在しない。
- サポートされていないスキーマのノードは、生の JSON エディタにフォールバックする。

<div id="notes">
  ## メモ
</div>

- このドキュメントは、オンボーディングおよび設定に関するプロトコルのリファクタリング内容を一元的に管理・追跡するための唯一の場所です。