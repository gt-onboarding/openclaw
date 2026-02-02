---
title: Zalouser
summary: "Zalo Personal プラグイン: zca-cli 経由の QR ログイン + メッセージング（プラグインのインストール + チャンネル設定 + CLI + ツール）"
read_when:
  - OpenClaw で Zalo Personal（非公式）サポートを利用したい
  - zalouser プラグインを設定または開発している
---

<div id="zalo-personal-plugin">
  # Zalo Personal (プラグイン)
</div>

Zalo Personal を OpenClaw で利用するためのプラグインで、`zca-cli` を使用して通常の Zalo 個人アカウントを自動化します。

> **警告:** 非公式な自動化は、アカウントの一時停止や利用停止につながる可能性があります。自己責任で使用してください。

<div id="naming">
  ## 命名
</div>

Channel ID は `zalouser` です。これは、このチャンネルが **個人の Zalo ユーザーアカウント**（非公式）を自動化するものであることを明示するためです。`zalo` は、将来の公式な Zalo api 連携用として予約してあります。

<div id="where-it-runs">
  ## 実行場所
</div>

このプラグインは **Gateway プロセス内** で実行されます。

リモート Gateway を使用している場合は、**Gateway を実行しているマシン** 上でプラグインをインストールおよび設定し、その後 Gateway を再起動してください。

<div id="install">
  ## インストール
</div>

<div id="option-a-install-from-npm">
  ### オプションA: npm からインストール
</div>

```bash
openclaw plugins install @openclaw/zalouser
```

その後、Gateway を再起動してください。


<div id="option-b-install-from-a-local-folder-dev">
  ### オプションB: ローカルフォルダからインストール（開発用）
</div>

```bash
openclaw plugins install ./extensions/zalouser
cd ./extensions/zalouser && pnpm install
```

その後、Gateway を再起動してください。


<div id="prerequisite-zca-cli">
  ## 前提条件: zca-cli
</div>

Gateway を実行するマシンの `PATH` 環境変数に `zca` が含まれている必要があります。

```bash
zca --version
```


<div id="config">
  ## 設定
</div>

チャンネルの設定は `channels.zalouser` 以下に配置します（`plugins.entries.*` ではありません）:

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


<div id="cli">
  ## CLI
</div>

```bash
openclaw channels login --channel zalouser
openclaw channels logout --channel zalouser
openclaw channels status --probe
openclaw message send --channel zalouser --target <threadId> --message "Hello from OpenClaw"
openclaw directory peers list --channel zalouser --query "name"
```


<div id="agent-tool">
  ## エージェントツール
</div>

ツール名: `zalouser`

アクション: `send`, `image`, `link`, `friends`, `groups`, `me`, `status`