---
title: ブラウザーログイン
summary: "ブラウザ自動化用の手動ログインと X/Twitter への投稿"
read_when:
  - ブラウザ自動化のためにサイトにログインする必要がある
  - X/Twitter にアップデートを投稿したい
---

<div id="browser-login-xtwitter-posting">
  # ブラウザログインとX/Twitterへの投稿
</div>

<div id="manual-login-recommended">
  ## 手動ログイン（推奨）
</div>

サイトがログインを要求する場合は、**ホスト**ブラウザープロファイル（openclaw ブラウザー）で**手動でサインイン**してください。

モデルにあなたの認証情報を渡さないでください。自動ログインはしばしばボット対策に引っかかり、アカウントがロックされる原因になります。

ブラウザーに関するメインドキュメントに戻る: [Browser](/ja/tools/browser)。

<div id="which-chrome-profile-is-used">
  ## どの Chrome プロファイルが使用されますか？
</div>

OpenClaw は、**専用の Chrome プロファイル**（`openclaw` という名前で、オレンジ色の UI テーマ）を制御します。これは、日常的に使っているブラウザープロファイルとは別物です。

このプロファイルにアクセスする簡単な方法は 2 つあります:

1. **エージェントにブラウザを開くよう依頼し**、自分でログインします。
2. **CLI から開きます**:

```bash
openclaw browser start
openclaw browser open https://x.com
```

複数のブラウザープロファイルがある場合は、`--browser-profile &lt;name&gt;` を指定します（デフォルトは `openclaw` です）。

<div id="xtwitter-recommended-flow">
  ## X/Twitter: 推奨フロー
</div>

* **Read/search/threads:** **bird** CLI スキルを使う（ブラウザ不要で安定）。
  * リポジトリ: https://github.com/steipete/bird
* **Post updates:** **host** ブラウザを使う（手動ログインが必要）。

<div id="sandboxing-host-browser-access">
  ## サンドボックス化 + ホストブラウザへのアクセス
</div>

サンドボックス化されたブラウザセッションは、ボット検知を受ける可能性が**高くなります**。X/Twitter（およびその他の厳格なサイト）では、**ホスト**側のブラウザを優先して使用してください。

エージェントがサンドボックス化されている場合、ブラウザツールはデフォルトでサンドボックス側を使用します。ホスト側で制御できるようにするには、次のようにします：

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        browser: {
          allowHostControl: true
        }
      }
    }
  }
}
```

次に、ホストブラウザを対象にします：

```bash
openclaw browser open https://x.com --browser-profile openclaw --target host
```

または、その更新を投稿するエージェントのサンドボックスを無効にします。
