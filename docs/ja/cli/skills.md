---
title: スキル
summary: "`openclaw skills`（list/info/check）とスキルの利用条件に関する CLI リファレンス"
read_when:
  - 利用可能で実行準備が整っているスキルを確認したいとき
  - スキル用の不足しているバイナリ／環境変数／設定ファイルをデバッグしたいとき
---

<div id="openclaw-skills">
  # `openclaw skills`
</div>

スキル（バンドル済み + ワークスペース + 管理されたオーバーライド）を確認し、要件を満たして利用可能なものと、要件を満たさず利用できないものを見分けます。

関連項目:

* スキルシステム: [スキル](/ja/tools/skills)
* スキル設定: [スキル設定](/ja/tools/skills-config)
* ClawHub のインストール: [ClawHub](/ja/tools/clawhub)

<div id="commands">
  ## コマンド
</div>

```bash
openclaw skills list
openclaw skills list --eligible
openclaw skills info <name>
openclaw skills check
```
