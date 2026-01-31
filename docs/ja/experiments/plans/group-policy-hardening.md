---
title: グループポリシー強化
summary: "Telegram 許可リストの強化: プレフィックスと空白の正規化"
read_when:
  - 過去の Telegram 許可リスト変更をレビューするとき
---

<div id="telegram-allowlist-hardening">
  # Telegram 許可リストの強化
</div>

**日付**: 2026-01-05\
**ステータス**: 完了\
**PR**: #216

<div id="summary">
  ## 概要
</div>

Telegram 許可リストは、`telegram:` と `tg:` のプレフィックスを大文字・小文字を区別せずに受け付け、誤って含まれた空白も許容するようになりました。これにより、受信時の許可リストチェックが送信時の正規化処理と整合するようになりました。

<div id="what-changed">
  ## 変更点
</div>

* `telegram:` と `tg:` のプレフィックスは同一のものとして扱われます（大文字・小文字は区別されません）。
* 許可リストのエントリは前後の空白が削除され、空のエントリは無視されます。

<div id="examples">
  ## 例
</div>

以下の表記はいずれも同じ ID として扱われます:

* `telegram:123456`
* `TG:123456`
* `tg:123456`

<div id="why-it-matters">
  ## なぜ重要か
</div>

ログやチャット ID をコピー＆ペーストすると、多くの場合プレフィックスや余分な空白が含まれます。これを正規化しておくことで、DM やグループのどちらに返信するかを判定するときの偽陰性を防げます。

<div id="related-docs">
  ## 関連ドキュメント
</div>

* [グループチャット](/ja/concepts/groups)
* [Telegram プロバイダー](/ja/channels/telegram)