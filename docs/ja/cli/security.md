---
title: セキュリティ
summary: "`openclaw security` の CLI リファレンス（ありがちなセキュリティ上の危険な設定ミスの検査と修正）"
read_when:
  - 設定や状態に対して簡易なセキュリティ監査を実行したい
  - 安全な「修正」提案（chmod やデフォルト設定の厳格化）を適用したい
---

<div id="openclaw-security">
  # `openclaw security`
</div>

セキュリティツール（監査 + オプションの修復）。

関連項目:

* セキュリティガイド: [セキュリティ](/ja/gateway/security)

<div id="audit">
  ## 監査
</div>

```bash
openclaw security audit
openclaw security audit --deep
openclaw security audit --fix
```

この監査は、複数の DM 送信者がメインのセッションを共有している場合に警告を出し、共有インボックスでは `session.dmScope="per-channel-peer"`（マルチアカウントのチャネルでは `per-account-channel-peer`）を使用することを推奨します。
また、小規模なモデル（`<=300B`）がサンドボックス化なしで、かつ web/browser ツールを有効にした状態で使用されている場合にも警告を出します。
