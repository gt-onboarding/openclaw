---
title: カスタムスキルの作成
---

<div id="creating-custom-skills">
  # カスタムスキルの作成 🛠
</div>

OpenClaw は簡単に拡張できるように設計されています。「スキル」は、アシスタントに新しい機能を追加するための主な方法です。

<div id="what-is-a-skill">
  ## Skill とは何か？
</div>

Skill は、`SKILL.md` ファイル（LLM に対する指示とツール定義を記述したもの）と、必要に応じてスクリプトやリソースを含むディレクトリです。

<div id="step-by-step-your-first-skill">
  ## ステップバイステップではじめてのスキルを作成する
</div>

<div id="1-create-the-directory">
  ### 1. ディレクトリを作成する
</div>

スキルはワークスペース内にあり、通常は `~/.openclaw/workspace/skills/` に配置されます。スキル用の新しいディレクトリを作成してください。

```bash
mkdir -p ~/.openclaw/workspace/skills/hello-world
```


<div id="2-define-the-skillmd">
  ### 2. `SKILL.md` を定義する
</div>

そのディレクトリ内に `SKILL.md` ファイルを作成します。このファイルでは、メタデータを YAML フロントマターで、説明や手順を Markdown で記述します。

```markdown
---
name: hello_world
description: A simple skill that says hello.
---

# Hello World Skill
When the user asks for a greeting, use the `echo` tool to say "Hello from your custom skill!".
```


<div id="3-add-tools-optional">
  ### 3. ツールを追加する（任意）
</div>

フロントマターでカスタムツールを定義するか、`bash` や `browser` のような既存のシステムツールを使うようにエージェントに指示できます。

<div id="4-refresh-openclaw">
  ### 4. OpenClaw を更新する
</div>

エージェントに「refresh skills」と指示するか、Gateway を再起動してください。OpenClaw が新しいディレクトリを検出し、`SKILL.md` をインデックスします。

<div id="best-practices">
  ## ベストプラクティス
</div>

- **簡潔に書く**: モデルには「*何を*するか」だけを指示し、「AIとしてどうあるべきか」は書かないようにします。
- **安全性を最優先に**: スキルで `bash` を使う場合、信頼できないユーザー入力から任意のコマンドが注入されないよう、プロンプト内容に注意してください。
- **ローカルでテストする**: `openclaw agent --message "use my new skill"` を使ってローカルでテストしてください。

<div id="shared-skills">
  ## 共有スキル
</div>

[ClawHub](https://clawhub.com) 上でスキルを閲覧したり、自分のスキルを公開したりすることもできます。