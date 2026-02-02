---
title: グループメッセージ
summary: "WhatsApp グループメッセージ処理の動作と設定（mentionPatterns は複数のチャネル間で共有されます）"
read_when:
  - グループメッセージのルールやメンションを変更するとき
---

<div id="group-messages-whatsapp-web-channel">
  # グループメッセージ（WhatsApp Web チャンネル）
</div>

目的: Clawd を WhatsApp グループに常駐させ、メンションされたときにだけ反応させ、そのスレッドを個人DMセッションとは分離して保つこと。

注意: `agents.list[].groupChat.mentionPatterns` は、現在 Telegram/Discord/Slack/iMessage でも使用されています。このドキュメントでは、WhatsApp 固有の動作にフォーカスします。マルチエージェント構成では、エージェントごとに `agents.list[].groupChat.mentionPatterns` を設定するか、グローバルなフォールバックとして `messages.groupChat.mentionPatterns` を使用してください。

<div id="whats-implemented-2025-12-03">
  ## 実装済みの内容 (2025-12-03)
</div>

- アクティベーションモード: `mention`（デフォルト）または `always`。`mention` は ping（`mentionedJids` による実際の WhatsApp の @メンション、正規表現パターン、またはテキスト中の任意の場所にあるボットの E.164）のいずれかを必要とします。`always` はすべてのメッセージでエージェントを起動しますが、意味のある付加価値を出せる場合にのみ返信すべきで、それ以外の場合はサイレントトークン `NO_REPLY` を返します。デフォルト値は設定ファイル（`channels.whatsapp.groups`）で指定でき、グループごとに `/activation` で上書きできます。`channels.whatsapp.groups` が設定されている場合、それはグループの許可リストとしても機能します（すべてを許可するには `"*"` を含めます）。
- グループポリシー: `channels.whatsapp.groupPolicy` はグループメッセージを受け付けるかどうかを制御します（`open|disabled|allowlist`）。`allowlist` は `channels.whatsapp.groupAllowFrom` を使用します（フォールバック: 明示的な `channels.whatsapp.allowFrom`）。デフォルトは `allowlist`（送信者を追加するまでブロック）です。
- グループごとのセッション: セッションキーは `agent:<agentId>:whatsapp:group:<jid>` のような形になり、`/verbose on` や `/think high` のようなコマンド（単独のメッセージとして送信）はそのグループにのみ適用されます。個人 DM の状態には影響しません。グループスレッドに対してはハートビートはスキップされます。
- コンテキストインジェクション: 実行をトリガーしなかった **pending-only**（保留中のもののみ）のグループメッセージ（デフォルト 50 件）は、`[Chat messages since your last reply - for context]` の見出しの下にまとめて挿入され、トリガーとなった行は `[Current message - respond to this]` の下に置かれます。すでにセッション内にあるメッセージは再注入されません。
- 送信者の明示: 各グループバッチの末尾には `[from: Sender Name (+E164)]` が付くようになり、Pi が誰が話しているかを認識できます。
- 消えるメッセージ／一度だけ表示メッセージ: テキストやメンションを抽出する前にそれらを展開するため、その中にある ping でも引き続きトリガーされます。
- グループ用システムプロンプト: グループセッションの最初のターン（および `/activation` によってモードが変更されたとき）には、システムプロンプトに短い説明文を注入します。例: `You are replying inside the WhatsApp group "<subject>". Group members: Alice (+44...), Bob (+43...), … Activation: trigger-only … Address the specific sender noted in the message context.` メタデータが利用できない場合でも、エージェントにはグループチャットであることを伝えます。

<div id="config-example-whatsapp">
  ## 設定例 (WhatsApp)
</div>

WhatsApp がテキスト本文から見た目上の `@` 記号を削除してしまう場合でも表示名でのメンションが動作するように、`~/.openclaw/openclaw.json` に `groupChat` ブロックを追加します。

```json5
{
  channels: {
    whatsapp: {
      groups: {
        "*": { requireMention: true }
      }
    }
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          historyLimit: 50,
          mentionPatterns: [
            "@?openclaw",
            "\\+?15555550123"
          ]
        }
      }
    ]
  }
}
```

Notes:

* この正規表現は大文字・小文字を区別せず、`@openclaw` のような表示名でのメンションと、`+` やスペースの有無にかかわらず同じ生の番号の両方をカバーします。
* WhatsApp は、誰かが連絡先をタップした場合には依然として `mentionedJids` 経由で正規のメンションを送信するため、この番号ベースのフォールバックが必要になることは稀ですが、有用なセーフティネットとして機能します。


<div id="activation-command-owner-only">
  ### 有効化コマンド（オーナーのみ）
</div>

グループチャットで次のコマンドを使用します：

- `/activation mention`
- `/activation always`

`channels.whatsapp.allowFrom` に設定されたオーナー番号（未設定の場合はボット自身の E.164）のみがこれを変更できます。現在の有効化モードを確認するには、グループで `/status` を単独のメッセージとして送信してください。

<div id="how-to-use">
  ## 使い方
</div>

1) OpenClaw を実行しているあなたの WhatsApp アカウントを、そのグループに追加します。
2) `@openclaw …` とメンションしてメッセージを送ります（または番号を含めます）。`groupPolicy: "open"`（全ユーザーからのメッセージを無制限に受け付ける設定）にしていない限り、トリガーできるのは許可リストに含まれる送信者だけです。
3) エージェントへのプロンプトには、直近のグループコンテキストに加えて末尾の `[from: …]` マーカーが含まれ、正しい相手を特定して返信できるようになります。
4) セッションレベルのディレクティブ（`/verbose on`, `/think high`, `/new` または `/reset`, `/compact`）は、そのグループのセッションにのみ適用されます。認識されるよう、それらだけを含む単独のメッセージとして送信してください。あなた個人の DM セッションは独立したままです。

<div id="testing-verification">
  ## テスト / 検証
</div>

- 手動スモークテスト:
  - グループ内で `@openclaw` ping を送信し、送信者名を参照した返信が返ってくることを確認する。
  - 2 回目の ping を送信し、履歴ブロックが含まれていることを確認し、その次のターンでクリアされることを確認する。
- Gateway のログを確認する（`--verbose` オプションを付けて実行）ことで、`from: <groupJid>` および末尾の `[from: …]` サフィックスを含む `inbound web message` エントリが表示されることを確認する。

<div id="known-considerations">
  ## 既知の考慮事項
</div>

- グループではノイズの多いブロードキャストを避けるため、ハートビートは意図的にスキップされます。
- エコー抑制は結合されたバッチ文字列を使って行われます。メンションなしで同一のテキストを 2 回送信した場合、最初の 1 回にのみレスポンスが返されます。
- セッションストアのエントリは、セッションストア内で `agent:<agentId>:whatsapp:group:<jid>` という形式で表示されます（デフォルトでは `~/.openclaw/agents/<agentId>/sessions/sessions.json`）。エントリが存在しない場合は、そのグループがまだ一度も実行をトリガーしていないことを意味します。
- グループでの入力中インジケーターは `agents.defaults.typingMode` に従います（デフォルト: メンションされていない場合は `message`）。