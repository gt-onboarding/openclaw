---
title: Dns
summary: "`openclaw dns`（広域ディスカバリ用ヘルパー）の CLI リファレンス"
read_when:
  - Tailscale + CoreDNS を使って広域ディスカバリ（DNS-SD）を行いたいとき
  - カスタムディスカバリドメイン（例: openclaw.internal）のスプリット DNS を設定したいとき
---

<div id="openclaw-dns">
  # `openclaw dns`
</div>

広域ディスカバリー向けの DNS ヘルパー（Tailscale + CoreDNS）。現在は主に macOS + Homebrew CoreDNS を対象としています。

関連項目:

* Gateway ディスカバリー: [Discovery](/ja/gateway/discovery)
* 広域ディスカバリー設定: [Configuration](/ja/gateway/configuration)

<div id="setup">
  ## セットアップ
</div>

```bash
openclaw dns setup
openclaw dns setup --apply
```
