---
title: デバイス
summary: "`openclaw devices`（デバイスのペアリングとトークンのローテーション／失効）に関する CLI リファレンス"
read_when:
  - デバイスのペアリング要求を承認するとき
  - デバイスのトークンをローテーションまたは失効させる必要があるとき
---

<div id="openclaw-devices">
  # `openclaw devices`
</div>

デバイスのペアリングリクエストとデバイススコープのトークンを管理します。

<div id="commands">
  ## コマンド
</div>

<div id="openclaw-devices-list">
  ### `openclaw devices list`
</div>

保留中のペアリングリクエストとペアリング済みデバイスを一覧表示します。

```
openclaw devices list
openclaw devices list --json
```


<div id="openclaw-devices-approve-requestid">
  ### `openclaw devices approve <requestId>`
</div>

保留中のデバイスペアリング・リクエストを承認します。

```
openclaw devices approve <requestId>
```


<div id="openclaw-devices-reject-requestid">
  ### `openclaw devices reject <requestId>`
</div>

保留中のデバイスからのペアリング要求を拒否します。

```
openclaw devices reject <requestId>
```


<div id="openclaw-devices-rotate-device-id-role-role-scope-scope">
  ### `openclaw devices rotate --device <id> --role <role> [--scope <scope...>]`
</div>

特定のロール用のデバイストークンをローテーションします（必要に応じてスコープも更新します）。

```
openclaw devices rotate --device <deviceId> --role operator --scope operator.read --scope operator.write
```


<div id="openclaw-devices-revoke-device-id-role-role">
  ### `openclaw devices revoke --device <id> --role <role>`
</div>

特定のロールに紐づくデバイストークンを無効化します。

```
openclaw devices revoke --device <deviceId> --role node
```


<div id="common-options">
  ## 共通オプション
</div>

- `--url <url>`: Gateway の WebSocket URL（設定済みであれば `gateway.remote.url` がデフォルト値）。
- `--token <token>`: Gateway のトークン（必要な場合）。
- `--password <password>`: Gateway のパスワード（パスワード認証）。
- `--timeout <ms>`: RPC のタイムアウト。
- `--json`: JSON 出力（スクリプトからの利用に推奨）。

<div id="notes">
  ## 注意事項
</div>

- トークンローテーションは、新しいトークン（機密性の高い値）を返します。シークレットと同様に扱ってください。
- これらのコマンドを実行するには、`operator.pairing`（または `operator.admin`）スコープが必要です。