---
title: 権限
summary: "macOS 権限の永続化（TCC）と署名要件"
read_when:
  - macOS の権限ダイアログが表示されない／固まってしまう問題をデバッグするとき
  - macOS アプリをパッケージングまたは署名するとき
  - バンドル ID やアプリのインストールパスを変更するとき
---

<div id="macos-permissions-tcc">
  # macOS の権限 (TCC)
</div>

macOS の権限付与は非常に壊れやすい仕組みです。TCC は権限付与を、
アプリのコード署名、バンドル識別子、ディスク上のパスにひも付けます。これらのいずれかが変わると、
macOS はそのアプリを新しいものとして扱い、権限プロンプトをリセットしたり表示しなくなったりする場合があります。

<div id="requirements-for-stable-permissions">
  ## 安定した権限のための要件
</div>

- 同じパス: アプリは常に同じ場所から実行すること（OpenClaw の場合は `dist/OpenClaw.app`）。
- 同じバンドル識別子: バンドル ID を変更すると、新しい権限識別情報が作成される。
- 署名付きアプリ: 未署名またはアドホック署名のビルドでは権限が永続化されない。
- 一貫した署名: 再ビルドしても署名が安定するように、正式な Apple Development または Developer ID 証明書を使用すること。

アドホック署名はビルドごとに新しい識別情報を生成する。その結果、macOS は以前に付与された権限を忘れ、古いエントリがクリアされるまでプロンプトがまったく表示されなくなることがある。

<div id="recovery-checklist-when-prompts-disappear">
  ## プロンプトが表示されなくなったときの復旧チェックリスト
</div>

1. アプリを終了します。
2. 「システム設定」&gt;「プライバシーとセキュリティ」で当該アプリの項目を削除します。
3. 同じパスからアプリを再起動し、もう一度権限を付与します。
4. それでもプロンプトが表示されない場合は、`tccutil` で TCC エントリをリセットしてから、再度試してください。
5. 一部の権限は、macOS を完全に再起動した後でないと再表示されません。

リセットの例（必要に応じて bundle ID を置き換えてください）:

```bash
sudo tccutil reset Accessibility bot.molt.mac
sudo tccutil reset ScreenCapture bot.molt.mac
sudo tccutil reset AppleEvents
```

権限をテストする場合は、必ず実際の有効な証明書で署名してください。権限が問題にならないローカル環境での簡易な実行に限り、アドホックビルドを使用してもかまいません。
