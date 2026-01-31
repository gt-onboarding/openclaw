---
title: ボイスオーバーレイ
summary: "ウェイクワードとプッシュトゥトークが競合する場合のボイスオーバーレイのライフサイクル"
read_when:
  - ボイスオーバーレイの挙動を調整するとき
---

<div id="voice-overlay-lifecycle-macos">
  # Voice Overlay ライフサイクル (macOS)
</div>

対象: macOS アプリのコントリビューター。目的: ウェイクワードとプッシュトゥトークが重なった場合でも、Voice Overlay の挙動を予測可能なものに保つこと。

<div id="current-intent">
  ### 現在の意図
</div>

- オーバーレイがすでにウェイクワードで表示されている状態でユーザーがホットキーを押した場合、ホットキーのセッションはテキストをリセットせず、既存のテキストを*引き継ぎ*ます。ホットキーを押している間はオーバーレイは表示されたままです。ユーザーがキーを離したとき、トリミング後のテキストがあれば送信し、なければオーバーレイを閉じます。
- ウェイクワードのみの場合は、無音になると自動的に送信されます。一方、プッシュ・トゥ・トークでは、キーを離したタイミングですぐに送信されます。

<div id="implemented-dec-9-2025">
  ### 実装済み (2025年12月9日)
</div>

- オーバーレイ セッションでは、キャプチャごと（ウェイクワードまたはプッシュ・トゥ・トーク）にトークンを保持するようになりました。トークンが一致しない場合、partial/final/send/dismiss/level の更新は破棄され、古いコールバックが呼ばれることを防ぎます。
- プッシュ・トゥ・トークは、表示中のオーバーレイテキストをプレフィックスとして取り込みます（そのため、ウェイクオーバーレイが表示されている間にホットキーを押すと、そのテキストを保持し、新しい音声を後続に追記します）。最大 1.5 秒間、最終的なトランスクリプトを待機し、それが得られない場合は現在のテキストにフォールバックします。
- チャイム／オーバーレイに関するログは、`voicewake.overlay`、`voicewake.ptt`、`voicewake.chime` カテゴリにおいて `info` レベルで出力されます（セッション開始、partial、final、send、dismiss、チャイムの理由）。

<div id="next-steps">
  ### 次のステップ
</div>

1. **VoiceSessionCoordinator (actor)**
   - 常に同時に保持する `VoiceSession` は 1 つだけ。
   - API（トークンベース）: `beginWakeCapture`, `beginPushToTalk`, `updatePartial`, `endCapture`, `cancel`, `applyCooldown`。
   - 古いトークンを持つコールバックを破棄する（古い Recognizer がオーバーレイを再度開くことを防ぐ）。
2. **VoiceSession (model)**
   - フィールド: `token`、`source` (wakeWord|pushToTalk)、コミット済み/未確定テキスト、チャイム用フラグ、タイマー（自動送信・アイドル）、`overlayMode` (display|editing|sending)、クールダウンの期限。
3. **オーバーレイのバインディング**
   - `VoiceSessionPublisher`（`ObservableObject`）がアクティブなセッションを SwiftUI にミラーする。
   - `VoiceWakeOverlayView` は `publisher` 経由でのみ描画し、グローバルシングルトンを直接変更しない。
   - オーバーレイでのユーザー操作（`sendNow`、`dismiss`、`edit`）は、セッショントークンを使って Coordinator にコールバックする。
4. **送信経路の統一**
   - `endCapture` 時: トリム済みテキストが空なら → dismiss。それ以外なら `performSend(session:)`（送信チャイムを 1 回再生し、転送して閉じる）。
   - push-to-talk: 遅延なし。wake-word: 自動送信用のオプションの遅延あり。
   - push-to-talk 完了後、wake-word が即座に再トリガーされないよう、wake ランタイムに短いクールダウンを適用する。
5. **ロギング**
   - Coordinator はサブシステム `bot.molt` で、カテゴリ `voicewake.overlay` および `voicewake.chime` の `.info` ログを出力する。
   - 主なイベント: `session_started`, `adopted_by_push_to_talk`, `partial`, `finalized`, `send`, `dismiss`, `cancel`, `cooldown`。

<div id="debugging-checklist">
  ### デバッグチェックリスト
</div>

- 張り付いたオーバーレイを再現しながらログをストリーミングすること:

  ```bash
  sudo log stream --predicate 'subsystem == "bot.molt" AND category CONTAINS "voicewake"' --level info --style compact
  ```
- アクティブなセッショントークンが 1 つだけであることを確認すること。古いコールバックはコーディネータによって破棄されているはず。
- push-to-talk のリリース時に、必ずアクティブなトークンを指定して `endCapture` を呼び出していることを確認すること。テキストが空の場合は、チャイム音や送信なしで `dismiss` が発生することを想定する。

<div id="migration-steps-suggested">
  ### 移行手順（推奨）
</div>

1. `VoiceSessionCoordinator`、`VoiceSession`、`VoiceSessionPublisher` を追加する。
2. `VoiceWakeRuntime` をリファクタリングして、`VoiceWakeOverlayController` に直接触るのではなく、セッションの作成／更新／終了を行うようにする。
3. `VoicePushToTalk` をリファクタリングして既存のセッションを引き継ぎ、リリース時に `endCapture` を呼び出すようにする。また、ランタイムのクールダウンを適用する。
4. `VoiceWakeOverlayController` を Publisher に接続し、runtime／PTT からの直接呼び出しを削除する。
5. セッションの引き継ぎ、クールダウン、および空テキスト時の破棄に関する統合テストを追加する。