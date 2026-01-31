---
title: Github Copilot
summary: "OpenClaw からデバイスフローを使って GitHub Copilot にサインインする"
read_when:
  - "GitHub Copilot をモデルプロバイダーとして使用したい場合"
  - "`openclaw models auth login-github-copilot` フローが必要な場合"
---

<div id="github-copilot">
  # GitHub Copilot
</div>

<div id="what-is-github-copilot">
  ## GitHub Copilot とは？
</div>

GitHub Copilot は、GitHub が提供する AI コーディングアシスタントです。GitHub アカウントとプランに応じて Copilot
モデルへアクセスできます。OpenClaw は、Copilot をモデルプロバイダーとして 2 通りの方法で利用できます。

<div id="two-ways-to-use-copilot-in-openclaw">
  ## OpenClaw で Copilot を使う 2 通りの方法
</div>

<div id="1-built-in-github-copilot-provider-github-copilot">
  ### 1) 組み込み GitHub Copilot プロバイダー (`github-copilot`)
</div>

ネイティブな device-login フローを使用して GitHub トークンを取得し、OpenClaw 実行時に Copilot API トークンと交換します。これは VS Code を必要としない **デフォルト** かつ最も簡単な方法です。

<div id="2-copilot-proxy-plugin-copilot-proxy">
  ### 2) Copilot Proxy プラグイン (`copilot-proxy`)
</div>

**Copilot Proxy** VS Code 拡張機能をローカルブリッジとして使用します。OpenClaw は
プロキシの `/v1` エンドポイントと通信し、そこで設定したモデル一覧を使用します。
すでに VS Code で Copilot Proxy を動かしている場合や、Copilot Proxy 経由でルーティングする必要がある場合にこれを選択してください。
プラグインを有効化し、VS Code 拡張機能を起動したままにしておく必要があります。

GitHub Copilot をモデルプロバイダー (`github-copilot`) として使用します。`login` コマンドは
GitHub のデバイスフローを実行し、認証プロファイルを保存し、そのプロファイルを使用するように
設定ファイルを更新します。

<div id="cli-setup">
  ## CLI のセットアップ
</div>

```bash
openclaw models auth login-github-copilot
```

指定された URL を開き、ワンタイムコードを入力するように求められます。処理が完了するまで、ターミナルはそのまま開いておいてください。


<div id="optional-flags">
  ### オプションフラグ
</div>

```bash
openclaw models auth login-github-copilot --profile-id github-copilot:work
openclaw models auth login-github-copilot --yes
```


<div id="set-a-default-model">
  ## 既定のモデルを設定する
</div>

```bash
openclaw models set github-copilot/gpt-4o
```


<div id="config-snippet">
  ### 設定スニペット
</div>

```json5
{
  agents: { defaults: { model: { primary: "github-copilot/gpt-4o" } } }
}
```


<div id="notes">
  ## 注意事項
</div>

- インタラクティブな TTY が必要です。端末から直接実行してください。
- 利用可能な Copilot モデルはあなたのプランに依存します。あるモデルが拒否された場合は、
  別の ID（例: `github-copilot/gpt-4.1`）を試してください。
- ログイン時には auth プロファイルストアに GitHub トークンを保存し、OpenClaw の実行時にそれを用いて
  Copilot API トークンを取得します。