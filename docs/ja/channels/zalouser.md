---
title: Zalouser
summary: "zca-cli（QR ログイン）を使った Zalo 個人アカウントのサポート、その機能と設定"
read_when:
  - OpenClaw 用に Zalo Personal をセットアップするとき
  - Zalo Personal のログインまたはメッセージフローをデバッグするとき
---

<div id="zalo-personal-unofficial">
  # Zalo Personal（非公式）
</div>

ステータス: 実験的。`zca-cli` を通じて**個人用 Zalo アカウント**を自動操作するインテグレーションです。

> **警告:** これは非公式なインテグレーションであり、アカウントの一時停止や凍結につながる可能性があります。利用は自己責任で行ってください。

<div id="plugin-required">
  ## 必要なプラグイン
</div>

Zalo Personal はプラグインとして提供されており、コアの標準インストールには含まれていません。

* CLI からインストール: `openclaw plugins install @openclaw/zalouser`
* もしくはソースコードのチェックアウトからインストール: `openclaw plugins install ./extensions/zalouser`
* 詳細: [プラグイン](/ja/plugin)

<div id="prerequisite-zca-cli">
  ## 前提条件：zca-cli
</div>

Gateway を実行しているマシンの `PATH` に `zca` バイナリが存在している必要があります。

* 確認: `zca --version`
* 存在しない場合は、zca-cli をインストールします（`extensions/zalouser/README.md` または upstream の zca-cli ドキュメントを参照してください）。

<div id="quick-setup-beginner">
  ## クイックセットアップ（初心者向け）
</div>

1. プラグインをインストールします（上記を参照）。
2. ログインします（QR、Gateway を実行しているマシン上で）:
   * `openclaw channels login --channel zalouser`
   * ターミナルに表示される QR コードを Zalo モバイルアプリでスキャンします。
3. チャンネルを有効化します：

```json5
{
  channels: {
    zalouser: {
      enabled: true,
      dmPolicy: "pairing"
    }
  }
}
```

4. Gateway を再起動します（またはオンボーディングを完了します）。
5. DM アクセスはデフォルトでペアリングになります。最初のやり取り時にペアリングコードを承認します。

<div id="what-it-is">
  ## 概要
</div>

* 受信メッセージの受信には `zca listen` を使用します。
* 返信（テキスト／メディア／リンク）の送信には `zca msg ...` を使用します。
* Zalo Bot API を利用できない「個人アカウント」用途向けに設計されています。

<div id="naming">
  ## 命名
</div>

このチャンネル ID は `zalouser` になっており、**個人の Zalo ユーザーアカウント**（非公式）を自動化するものであることを明示しています。`zalo` は、将来の公式な Zalo api 連携用として予約しています。

<div id="finding-ids-directory">
  ## ID の取得方法（ディレクトリ）
</div>

ディレクトリ CLI を使用して、ピアやグループとその ID を特定します。

```bash
openclaw directory self --channel zalouser
openclaw directory peers list --channel zalouser --query "name"
openclaw directory groups list --channel zalouser --query "work"
```

<div id="limits">
  ## 制限
</div>

* 送信テキストは約 2000 文字ごとに分割されます（Zalo クライアントの制限による）。
* ストリーミングはデフォルトでは無効です。

<div id="access-control-dms">
  ## アクセス制御（DM）
</div>

`channels.zalouser.dmPolicy` では `pairing | allowlist | open | disabled` を指定できます（デフォルト: `pairing`）。ここで `open` は、任意のユーザーからのメッセージを無制限に受け付ける設定を意味します。
`channels.zalouser.allowFrom` にはユーザー ID または名前を指定します。ウィザードは、利用可能な場合は `zca friend find` を使って名前から ID を解決します。

承認は次のコマンドで行います:

* `openclaw pairing list zalouser`
* `openclaw pairing approve zalouser <code>`

<div id="group-access-optional">
  ## グループアクセス（任意）
</div>

* デフォルト: `channels.zalouser.groupPolicy = "open"`（制限なしでグループを許可）。未設定時のデフォルトを上書きするには `channels.defaults.groupPolicy` を使用します。
* 許可リストに制限するには:
  * `channels.zalouser.groupPolicy = "allowlist"`
  * `channels.zalouser.groups`（キーはグループ ID または名前）
* すべてのグループをブロック: `channels.zalouser.groupPolicy = "disabled"`。
* 設定ウィザードでは、グループの許可リストを対話的に入力できます。
* 起動時に OpenClaw は許可リスト内のグループ名/ユーザー名を ID に解決してマッピングをログに記録し、解決できなかったエントリは入力された文字列のまま保持します。

Example:

```json5
{
  channels: {
    zalouser: {
      groupPolicy: "allowlist",
      groups: {
        "123456789": { allow: true },
        "Work Chat": { allow: true }
      }
    }
  }
}
```

<div id="multi-account">
  ## マルチアカウント
</div>

アカウントは zca のプロファイルに対応します。例:

```json5
{
  channels: {
    zalouser: {
      enabled: true,
      defaultAccount: "default",
      accounts: {
        work: { enabled: true, profile: "work" }
      }
    }
  }
}
```

<div id="troubleshooting">
  ## トラブルシューティング
</div>

**`zca` が見つからない場合:**

* zca-cli をインストールし、Gateway プロセスの `PATH` に含まれていることを確認してください。

**ログイン状態が保持されない場合:**

* `openclaw channels status --probe`
* 再ログイン: `openclaw channels logout --channel zalouser && openclaw channels login --channel zalouser`