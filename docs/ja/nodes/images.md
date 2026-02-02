---
title: 画像
summary: "send、Gateway、およびエージェント応答における画像とメディアの取り扱いルール"
read_when:
  - メディアパイプラインや添付ファイルを変更するとき
---

<div id="image-media-support-2025-12-05">
  # 画像とメディアのサポート — 2025-12-05
</div>

WhatsApp チャンネルは **Baileys Web** 経由で動作します。本ドキュメントでは、送信、Gateway、およびエージェントからの返信に対する現在のメディア処理ルールをまとめています。

<div id="goals">
  ## 目標
</div>

- `openclaw message send --media` を使用して、任意のキャプション付きでメディアを送信できるようにする。
- Web インボックスからの自動返信で、テキストに加えてメディアも含められるようにする。
- 種類ごとの制限を、合理的で予測しやすい水準に保つ。

<div id="cli-surface">
  ## CLI サーフェイス
</div>

- `openclaw message send --media <path-or-url> [--message <caption>]`
  - `--media` は省略可能。キャプションは空でもよく、メディアのみを送信してもよい。
  - `--dry-run` は解決済み（最終）的なペイロードを表示する。`--json` は `{ channel, to, messageId, mediaUrl, caption }` を出力する。

<div id="whatsapp-web-channel-behavior">
  ## WhatsApp Web チャンネルの挙動
</div>

- 入力: ローカルファイルパス **または** HTTP(S) URL。
- フロー: Buffer に読み込み、メディア種別を検出し、適切なペイロードを構築:
  - **画像:** JPEG にリサイズ＆再圧縮（最大辺 2048px）、`agents.defaults.mediaMaxMb`（デフォルト 5MB）を目標としつつ、上限 6MB。
  - **音声/ボイス/動画:** 16MB までパススルーで送信。音声はボイスメッセージ（`ptt: true`）として送信される。
  - **ドキュメント:** 上記以外のすべてについて、最大 100MB まで対応し、可能な場合はファイル名を保持。
- WhatsApp の GIF 風再生: `gifPlayback: true`（CLI: `--gif-playback`）付きの MP4 を送信し、モバイルクライアントでインラインループ再生されるようにする。
- MIME 検出は、マジックバイト列を最優先とし、その次にヘッダー、その次にファイル拡張子を使用。
- キャプションは `--message` または `reply.text` から取得。空のキャプションも許可される。
- ログ出力: 通常ログでは `↩️`/`✅` を表示し、verbose モードではサイズとソースのパス/URL も含める。

<div id="auto-reply-pipeline">
  ## 自動返信パイプライン
</div>

- `getReplyFromConfig` は `{ text?, mediaUrl?, mediaUrls? }` を返します。
- メディアが含まれている場合、Web 送信コンポーネントは `openclaw message send` と同じパイプラインを使用してローカルパスまたは URL を解決します。
- 複数のメディア項目が指定されている場合は、順番に送信されます。

<div id="inbound-media-to-commands-pi">
  ## コマンドへのインバウンドメディア（Pi）
</div>

- 受信した Web メッセージにメディアが含まれている場合、OpenClaw はメディアを一時ファイルとしてダウンロードし、以下のテンプレート変数を提供します:
  - 受信メディア用の擬似 URL `{{MediaUrl}}`
  - コマンド実行前に書き出されるローカル一時パス `{{MediaPath}}`
- セッションごとの Docker サンドボックスが有効な場合、受信メディアはサンドボックスのワークスペース内にコピーされ、`MediaPath`/`MediaUrl` は `media/inbound/<filename>` のような相対パスに書き換えられます。
- メディア理解処理（`tools.media.*` または共有の `tools.media.models` で設定されている場合）はテンプレート処理の前に実行され、`Body` に `[Image]`、`[Audio]`、`[Video]` ブロックを挿入できます。
  - 音声メディアの場合は `{{Transcript}}` が設定され、その書き起こしがコマンド解析に使用されるため、スラッシュコマンドは引き続き動作します。
  - 動画および画像の説明は、コマンド解析のためにキャプションテキストを保持します。
- デフォルトでは、条件に一致した最初の image/audio/video 添付ファイルのみが処理されます。複数の添付ファイルを処理するには、`tools.media.<cap>.attachments` を設定します。

<div id="limits-errors">
  ## 制限とエラー
</div>

**送信上限（WhatsApp Web 送信）**

- 画像：再圧縮後に約 6 MB が上限。
- 音声 / ボイス / 動画：16 MB 上限、ドキュメント：100 MB 上限。
- サイズ超過または読み取り不能なメディア → ログに明確なエラーが記録され、その返信はスキップされる。

**メディア解析の上限（文字起こし / 説明生成）**

- 画像のデフォルト：10 MB（`tools.media.image.maxBytes`）。
- 音声のデフォルト：20 MB（`tools.media.audio.maxBytes`）。
- 動画のデフォルト：50 MB（`tools.media.video.maxBytes`）。
- サイズ超過メディアは解析処理をスキップするが、返信自体は元の本文のまま送信される。

<div id="notes-for-tests">
  ## テスト向けメモ
</div>

- 画像／音声／ドキュメントのケースについて、送信＋返信フローを一通りカバーすること。
- 画像の再圧縮（サイズ上限の適用）と、音声に対するボイスメモ用フラグを検証すること。
- 複数メディアを含む返信が、連続した送信として順次ファンアウトされることを確認すること。