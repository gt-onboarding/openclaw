---
title: 設定
summary: "`openclaw config` 用の CLI リファレンス（設定値の取得／設定／解除）"
read_when:
  - 設定を非対話的に表示・編集したいとき
---

<div id="openclaw-config">
  # `openclaw config`
</div>

設定ヘルパー: パスを指定して値の get/set/unset を行います。サブコマンドを指定せずに実行すると、設定ウィザードが起動します（`openclaw configure` と同じです）。

<div id="examples">
  ## 例
</div>

```bash
openclaw config get browser.executablePath
openclaw config set browser.executablePath "/usr/bin/google-chrome"
openclaw config set agents.defaults.heartbeat.every "2h"
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
openclaw config unset tools.web.search.apiKey
```


<div id="paths">
  ## パス
</div>

パスはドット記法またはブラケット記法で指定します。

```bash
openclaw config get agents.defaults.workspace
openclaw config get agents.list[0].id
```

特定のエージェントを指定するには、エージェント一覧のインデックス番号を使用します。

```bash
openclaw config get agents.list
openclaw config set agents.list[1].tools.exec.node "node-id-or-name"
```


<div id="values">
  ## 値
</div>

値は、可能な場合は JSON5 としてパースされ、それ以外の場合は文字列として扱われます。
JSON5 パースを強制するには `--json` を使用します。

```bash
openclaw config set agents.defaults.heartbeat.every "0m"
openclaw config set gateway.port 19001 --json
openclaw config set channels.whatsapp.groups '["*"]' --json
```

編集したら Gateway を再起動してください。
