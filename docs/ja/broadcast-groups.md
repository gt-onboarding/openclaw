---
title: ブロードキャストグループ
summary: "WhatsApp メッセージを複数のエージェント群にブロードキャストする"
read_when:
  - ブロードキャストグループの設定時
  - WhatsApp におけるマルチエージェント返信のデバッグ時
status: experimental
---

<div id="broadcast-groups">
  # ブロードキャストグループ
</div>

**ステータス:** 実験的機能\
**バージョン:** 2026.1.9 で追加

<div id="overview">
  ## 概要
</div>

Broadcast Groups は、複数のエージェントが同じメッセージを同時に処理し、応答できるようにする機能です。これにより、1 つの電話番号だけで、1 つの WhatsApp グループまたは DM 内で連携して動作する専門的なエージェントチームを構築できます。

現在の対応範囲: **WhatsApp のみ**（Web チャンネル）。

Broadcast groups は、チャネルの許可リストとグループの有効化ルールの後に評価されます。WhatsApp グループでは、これは OpenClaw が通常返信するタイミング（例: メンションされたときなど。グループ設定による）でブロードキャストが行われることを意味します。

<div id="use-cases">
  ## ユースケース
</div>

<div id="1-specialized-agent-teams">
  ### 1. 特化型エージェントチーム
</div>

責務を細かく分割し、明確にフォーカスされた複数のエージェントをデプロイします：

```
Group: "Development Team"
Agents:
  - CodeReviewer (reviews code snippets)
  - DocumentationBot (generates docs)
  - SecurityAuditor (checks for vulnerabilities)
  - TestGenerator (suggests test cases)
```

各エージェントが同じメッセージを処理し、それぞれの専門分野に即した視点を提示します。

<div id="2-multi-language-support">
  ### 2. 多言語対応
</div>

```
Group: "International Support"
Agents:
  - Agent_EN (responds in English)
  - Agent_DE (responds in German)
  - Agent_ES (responds in Spanish)
```

<div id="3-quality-assurance-workflows">
  ### 3. 品質保証のワークフロー
</div>

```
Group: "Customer Support"
Agents:
  - SupportAgent (provides answer)
  - QAAgent (reviews quality, only responds if issues found)
```

<div id="4-task-automation">
  ### 4. タスクの自動化
</div>

```
Group: "Project Management"
Agents:
  - TaskTracker (タスクデータベースを更新)
  - TimeLogger (費やした時間を記録)
  - ReportGenerator (要約を作成)
```

<div id="configuration">
  ## 設定
</div>

<div id="basic-setup">
  ### 基本セットアップ
</div>

トップレベルに `broadcast` セクション（`bindings` の隣）を追加します。キーには WhatsApp のピア ID を使用します：

* グループチャット：グループ JID（例：`120363403215116621@g.us`）
* DM：E.164 形式の電話番号（例：`+15551234567`）

```json
{
  "broadcast": {
    "120363403215116621@g.us": ["alfred", "baerbel", "assistant3"]
  }
}
```

**結果:** このチャットで OpenClaw が返信する際には、3 つすべてのエージェントが実行されます。

<div id="processing-strategy">
  ### 処理戦略
</div>

エージェントによるメッセージ処理の方法を制御します。

<div id="parallel-default">
  #### 並列（デフォルト）
</div>

すべてのエージェントが同時に処理します：

```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

<div id="sequential">
  #### 順次
</div>

エージェントは順番に処理されます（前の処理が完了するまで次のエージェントは待機します）:

```json
{
  "broadcast": {
    "strategy": "sequential",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

<div id="complete-example">
  ### 完全なサンプル
</div>

```json
{
  "agents": {
    "list": [
      {
        "id": "code-reviewer",
        "name": "Code Reviewer",
        "workspace": "/path/to/code-reviewer",
        "sandbox": { "mode": "all" }
      },
      {
        "id": "security-auditor",
        "name": "Security Auditor",
        "workspace": "/path/to/security-auditor",
        "sandbox": { "mode": "all" }
      },
      {
        "id": "docs-generator",
        "name": "Documentation Generator",
        "workspace": "/path/to/docs-generator",
        "sandbox": { "mode": "all" }
      }
    ]
  },
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["code-reviewer", "security-auditor", "docs-generator"],
    "120363424282127706@g.us": ["support-en", "support-de"],
    "+15555550123": ["assistant", "logger"]
  }
}
```

<div id="how-it-works">
  ## 動作の仕組み
</div>

<div id="message-flow">
  ### メッセージフロー
</div>

1. **受信メッセージ** が WhatsApp グループに届く
2. **ブロードキャスト判定**: システムが peer ID が `broadcast` に含まれているかを確認する
3. **broadcast リストに含まれている場合**:
   * 記載されているすべてのエージェントがメッセージを処理する
   * 各エージェントは独自のセッションキーと分離されたコンテキストを持つ
   * エージェントは並列（デフォルト）または逐次で処理する
4. **broadcast リストに含まれていない場合**:
   * 通常のルーティングが適用される（最初にマッチしたバインディング）

Note: broadcast グループは、チャンネルの許可リストやグループの有効化ルール（メンション／コマンド／その他）をバイパスしません。メッセージが処理対象になった際に、「どのエージェントが実行されるか」だけを変更します。

<div id="session-isolation">
  ### セッションの分離
</div>

ブロードキャストグループ内の各エージェントは、完全に分離された次のものを保持します:

* **セッションキー**（`agent:alfred:whatsapp:group:120363...` と `agent:baerbel:whatsapp:group:120363...`）
* **会話履歴**（エージェントは他のエージェントのメッセージを閲覧できない）
* **ワークスペース**（設定されていれば個別のサンドボックス）
* **ツールアクセス**（異なる allow/deny リスト）
* **メモリ / コンテキスト**（個別の IDENTITY.md や SOUL.md など）
* **グループコンテキストバッファ**（コンテキストとして使用される直近のグループメッセージ）は各ピアで共有されるため、ブロードキャスト対象のすべてのエージェントは、トリガー時に同じコンテキストを参照します

これにより、各エージェントは次のような違いを持たせることができます:

* 異なる人格
* 異なるツールアクセス（例: 読み取り専用 vs 読み書き可能）
* 異なるモデル（例: opus vs. sonnet）
* インストールされるスキルを変えること

<div id="example-isolated-sessions">
  ### 例: 分離されたセッション
</div>

エージェント `["alfred", "baerbel"]` を含むグループ `120363403215116621@g.us` では、

**Alfred のコンテキスト：**

```
Session: agent:alfred:whatsapp:group:120363403215116621@g.us
History: [ユーザーメッセージ、alfredの過去の応答]
Workspace: /Users/pascal/openclaw-alfred/
Tools: read, write, exec
```

**Bärbel の状況:**

```
Session: agent:baerbel:whatsapp:group:120363403215116621@g.us  
History: [user message, baerbel's previous responses]
Workspace: /Users/pascal/openclaw-baerbel/
Tools: read only
```

<div id="best-practices">
  ## ベストプラクティス
</div>

<div id="1-keep-agents-focused">
  ### 1. エージェントを特化させる
</div>

各エージェントは、単一かつ明確な責務を一つだけ持つように設計してください。

```json
{
  "broadcast": {
    "DEV_GROUP": ["formatter", "linter", "tester"]
  }
}
```

✅ **良い例:** 各エージェントには 1 つの役割だけを与える
❌ **悪い例:** 1 つの汎用的な &quot;dev-helper&quot; エージェント

<div id="2-use-descriptive-names">
  ### 2. わかりやすい名前を付ける
</div>

各エージェントが何をするものかを明確にします：

```json
{
  "agents": {
    "security-scanner": { "name": "Security Scanner" },
    "code-formatter": { "name": "Code Formatter" },
    "test-generator": { "name": "Test Generator" }
  }
}
```

<div id="3-configure-different-tool-access">
  ### 3. ツールへのアクセスを個別に設定する
</div>

エージェントには、必要なツールだけを割り当てます。

```json
{
  "agents": {
    "reviewer": {
      "tools": { "allow": ["read", "exec"] }  // Read-only
    },
    "fixer": {
      "tools": { "allow": ["read", "write", "edit", "exec"] }  // 読み取り・書き込み
    }
  }
}
```

<div id="4-monitor-performance">
  ### 4. パフォーマンスを監視する
</div>

多くのエージェントがいる場合は、次の点を検討してください:

* 高速化のために `"strategy": "parallel"`（デフォルト）を使用する
* ブロードキャストグループ内のエージェント数を 5〜10 に制限する
* より単純なエージェントには高速なモデルを使用する

<div id="5-handle-failures-gracefully">
  ### 5. 障害を適切に処理する
</div>

エージェントは互いに独立して動作します。1つのエージェントでエラーが発生しても、他のエージェントの処理がブロックされることはありません。

```
Message → [Agent A ✓, Agent B ✗ error, Agent C ✓]
Result: Agent A and C respond, Agent B logs error
```

<div id="compatibility">
  ## 互換性
</div>

<div id="providers">
  ### プロバイダー
</div>

ブロードキャストグループは現在、以下のプロバイダーに対応しています:

* ✅ WhatsApp（実装済み）
* 🚧 Telegram（計画中）
* 🚧 Discord（計画中）
* 🚧 Slack（計画中）

<div id="routing">
  ### ルーティング
</div>

ブロードキャストグループは、既存のルーティングと連携して機能します。

```json
{
  "bindings": [
    { "match": { "channel": "whatsapp", "peer": { "kind": "group", "id": "GROUP_A" } }, "agentId": "alfred" }
  ],
  "broadcast": {
    "GROUP_B": ["agent1", "agent2"]
  }
}
```

* `GROUP_A`: alfred のみが応答する（通常のルーティング）
* `GROUP_B`: agent1 と agent2 の両方が応答する（ブロードキャスト）

**優先順位:** `broadcast` は `bindings` よりも優先されます。

<div id="troubleshooting">
  ## トラブルシューティング
</div>

<div id="agents-not-responding">
  ### エージェントが応答しない
</div>

**確認:**

1. エージェント ID が `agents.list` に存在していること
2. Peer ID の形式が正しいこと（例: `120363403215116621@g.us`）
3. エージェントが deny リストに含まれていないこと

**デバッグ:**

```bash
tail -f ~/.openclaw/logs/gateway.log | grep broadcast
```

<div id="only-one-agent-responding">
  ### 1 つのエージェントしか応答しない
</div>

**原因:** Peer ID が `bindings` にはあるが、`broadcast` にはない可能性があります。

**対処:** `broadcast` の設定に追加するか、`bindings` から削除します。

<div id="performance-issues">
  ### パフォーマンスの問題
</div>

**エージェント数が多いと遅くなる場合:**

* グループあたりのエージェント数を減らす
* より軽量なモデルを使用する（opus ではなく sonnet など）
* サンドボックスの起動にかかる時間を確認する

<div id="examples">
  ## 使用例
</div>

<div id="example-1-code-review-team">
  ### 例 1: コードレビュー チーム
</div>

```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": [
      "code-formatter",
      "security-scanner",
      "test-coverage",
      "docs-checker"
    ]
  },
  "agents": {
    "list": [
      { "id": "code-formatter", "workspace": "~/agents/formatter", "tools": { "allow": ["read", "write"] } },
      { "id": "security-scanner", "workspace": "~/agents/security", "tools": { "allow": ["read", "exec"] } },
      { "id": "test-coverage", "workspace": "~/agents/testing", "tools": { "allow": ["read", "exec"] } },
      { "id": "docs-checker", "workspace": "~/agents/docs", "tools": { "allow": ["read"] } }
    ]
  }
}
```

**ユーザー送信:** コードスニペット
**レスポンス:**

* code-formatter: &quot;インデントを修正し、型ヒントを追加しました&quot;
* security-scanner: &quot;⚠️ 12行目に SQL インジェクションの脆弱性があります&quot;
* test-coverage: &quot;カバレッジは 45% で、エラーケースのテストが不足しています&quot;
* docs-checker: &quot;関数 `process_data` に docstring がありません&quot;

<div id="example-2-multi-language-support">
  ### 例 2：多言語対応
</div>

```json
{
  "broadcast": {
    "strategy": "sequential",
    "+15555550123": ["detect-language", "translator-en", "translator-de"]
  },
  "agents": {
    "list": [
      { "id": "detect-language", "workspace": "~/agents/lang-detect" },
      { "id": "translator-en", "workspace": "~/agents/translate-en" },
      { "id": "translator-de", "workspace": "~/agents/translate-de" }
    ]
  }
}
```

<div id="api-reference">
  ## API リファレンス
</div>

<div id="config-schema">
  ### 設定スキーマ
</div>

```typescript
interface OpenClawConfig {
  broadcast?: {
    strategy?: "parallel" | "sequential";
    [peerId: string]: string[];
  };
}
```

<div id="fields">
  ### フィールド
</div>

* `strategy` (オプション): エージェントの処理方法
  * `"parallel"` (デフォルト): すべてのエージェントが同時に処理する
  * `"sequential"`: エージェントが配列の順序で処理する

* `[peerId]`: WhatsApp グループ JID、E.164 番号、またはその他のピア ID
  * 値: メッセージを処理するエージェント ID の配列

<div id="limitations">
  ## 制約事項
</div>

1. **最大エージェント数:** 厳密な上限はないが、エージェントが 10 体を超えると動作が遅くなる可能性がある
2. **共有コンテキスト:** エージェント同士は互いの応答を閲覧できない（仕様どおりの挙動）
3. **メッセージ順序:** 並列応答は任意の順序で到着する可能性がある
4. **レート制限:** すべてのエージェントによる利用分が WhatsApp のレート制限に合算される

<div id="future-enhancements">
  ## 将来の拡張
</div>

予定されている機能：

* [ ] 共有コンテキストモード（エージェントが互いの応答を参照できる）
* [ ] エージェント間の協調（エージェント同士がシグナルを送信できる）
* [ ] 動的なエージェント選択（メッセージ内容に基づいてエージェント群を選択）
* [ ] エージェントの優先度（特定のエージェントが他より先に応答する）

<div id="see-also">
  ## 関連項目
</div>

* [マルチエージェント構成](/ja/multi-agent-sandbox-tools)
* [ルーティング構成](/ja/concepts/channel-routing)
* [セッション管理](/ja/concepts/sessions)