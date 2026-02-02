---
title: 昇格モード
summary: "昇格実行モードと /elevated ディレクティブ"
read_when:
  - 昇格モードのデフォルト設定、許可リスト、またはスラッシュコマンドの挙動を調整したいとき
---

<div id="elevated-mode-elevated-directives">
  # Elevated モード（/elevated ディレクティブ）
</div>

<div id="what-it-does">
  ## 何を行うか
</div>

- `/elevated on` は Gateway ホスト上で実行され、exec 承認を維持します（`/elevated ask` と同じ）。
- `/elevated full` は Gateway ホスト上で実行され、かつ exec を自動承認します（exec 承認をスキップします）。
- `/elevated ask` は Gateway ホスト上で実行されますが、exec 承認を維持します（`/elevated on` と同じ）。
- `on`/`ask` は `exec.security=full` を強制 **しません**。設定された security/ask ポリシーがそのまま適用されます。
- エージェントが**サンドボックス**化されている場合のみ挙動を変更します（そうでない場合、exec はすでにホスト上で実行されています）。
- ディレクティブの書式: `/elevated on|off|ask|full`, `/elev on|off|ask|full`。
- `on|off|ask|full` のみが受け付けられます。それ以外が指定された場合はヒントメッセージを返すだけで、状態は変更されません。

<div id="what-it-controls-and-what-it-doesnt">
  ## 何を制御し、何を制御しないか
</div>

- **利用ゲート**: `tools.elevated` がグローバルなベースラインです。`agents.list[].tools.elevated` で、エージェントごとに elevated レベルをさらに制限できます（両方で許可されている必要があります）。
- **セッション単位の状態**: `/elevated on|off|ask|full` は、現在のセッションキーに対する elevated レベルを設定します。
- **インラインディレクティブ**: メッセージ内の `/elevated on|ask|full` は、そのメッセージにのみ適用されます。
- **グループ**: グループチャットでは、エージェントがメンションされた場合にのみ、elevated ディレクティブが有効になります。メンション要件をバイパスするコマンドのみのメッセージは、メンションされたものとして扱われます。
- **ホスト上での実行**: elevated は `exec` を Gateway ホスト上での実行に強制します。`full` はさらに `security=full` も設定します。
- **承認**: `full` は exec 承認をスキップします。`on`／`ask` は、許可リスト／ask ルールが要求する場合は承認を行います。
- **非サンドボックス化されたエージェント**: 実行場所には影響せず、ゲート処理、ログ、ステータスにのみ影響します。
- **ツールポリシーは依然として有効**: ツールポリシーによって `exec` が拒否されている場合、elevated は使用できません。
- **`/exec` とは別物**: `/exec` は、認可済み送信者に対するセッション単位のデフォルトを調整するものであり、elevated を必要としません。

<div id="resolution-order">
  ## 解決の優先順位
</div>

1. メッセージ内のインラインディレクティブ（そのメッセージにのみ適用）。
2. セッションのオーバーライド（ディレクティブだけを含むメッセージを送信して設定）。
3. グローバルなデフォルト値（設定内の `agents.defaults.elevatedDefault`）。

<div id="setting-a-session-default">
  ## セッションのデフォルト値を設定する
</div>

- **ディレクティブのみ**を含むメッセージを送信します（空白は含んでいても可）、例: `/elevated full`。
- 確認の返信が返されます（`Elevated mode set to full...` / `Elevated mode disabled.`）。
- elevated アクセスが無効化されている場合、または送信者が承認済みの許可リストに含まれていない場合、ディレクティブは対処可能なエラーで返信し、セッション状態は変更しません。
- 現在の elevated レベルを確認するには、引数なしで `/elevated`（または `/elevated:`）を送信します。

<div id="availability-allowlists">
  ## 可用性と許可リスト
</div>

- 機能ゲート: `tools.elevated.enabled`（コードが対応していても、設定でデフォルトをオフにできる）。
- 送信者許可リスト: `tools.elevated.allowFrom` にプロバイダーごとの許可リスト（例: `discord`、`whatsapp`）。
- エージェント単位のゲート: `agents.list[].tools.elevated.enabled`（任意。ここでは制限を強めることしかできない）。
- エージェント単位の許可リスト: `agents.list[].tools.elevated.allowFrom`（任意。設定されている場合、送信者はグローバルとエージェント単位の**両方**の許可リストに一致する必要がある）。
- Discord のフォールバック: `tools.elevated.allowFrom.discord` が省略された場合、フォールバックとして `channels.discord.dm.allowFrom` のリストが使用される。上書きするには `tools.elevated.allowFrom.discord`（`[]` でも可）を設定する。エージェント単位の許可リストではこのフォールバックは**使用されない**。
- すべてのゲートを通過する必要があり、通過できない場合は elevated は利用不可として扱われる。

<div id="logging-status">
  ## ログ記録とステータス
</div>

- 権限昇格された exec 呼び出しは info レベルでログ出力されます。
- セッションステータスには Elevated モードが含まれます（例: `elevated=ask`, `elevated=full`）。