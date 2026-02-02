---
title: Canvas
summary: "WKWebView とカスタム URL スキームを用いて埋め込まれた、エージェント制御の Canvas パネル"
read_when:
  - macOS の Canvas パネルを実装するとき
  - ビジュアルなワークスペース向けのエージェント制御を追加するとき
  - WKWebView における Canvas 読み込みをデバッグするとき
---

<div id="canvas-macos-app">
  # Canvas (macOS app)
</div>

macOS アプリは、`WKWebView` を使用してエージェントが制御する **Canvas パネル** を埋め込みます。これは、HTML/CSS/JS、A2UI、および小規模な対話型 UI サーフェス向けの軽量なビジュアルワークスペースです。

<div id="where-canvas-lives">
  ## Canvas の保存場所
</div>

Canvas の状態は Application Support 配下に保存されます：

- `~/Library/Application Support/OpenClaw/canvas/<session>/...`

Canvas パネルは、これらのファイルを **カスタム URL スキーム** 経由で提供します：

- `openclaw-canvas://<session>/<path>`

例：

- `openclaw-canvas://main/` → `<canvasRoot>/main/index.html`
- `openclaw-canvas://main/assets/app.css` → `<canvasRoot>/main/assets/app.css`
- `openclaw-canvas://main/widgets/todo/` → `<canvasRoot>/main/widgets/todo/index.html`

ルートに `index.html` が存在しない場合、アプリは **組み込みのスキャフォルドページ** を表示します。

<div id="panel-behavior">
  ## パネルの挙動
</div>

- メニューバー（またはマウスカーソル）の近くに表示される、枠のないサイズ変更可能なパネル。
- セッションごとにサイズと位置を記憶する。
- ローカルの Canvas ファイルが変更されると自動的に再読み込みされる。
- 一度に表示される Canvas パネルは 1 つだけ（必要に応じてセッションが切り替わる）。

Canvas は、Settings → **Allow Canvas** から無効化できる。無効化されている場合、canvas
ノードのコマンドは `CANVAS_DISABLED` を返す。

<div id="agent-api-surface">
  ## エージェント API サーフェス
</div>

Canvas は **Gateway WebSocket** を介して公開されているため、エージェントは次のことができます:

* パネルの表示/非表示を切り替える
* パスまたは URL に移動する
* JavaScript を評価する
* スナップショット画像を取得する

CLI の例:

```bash
openclaw nodes canvas present --node <id>
openclaw nodes canvas navigate --node <id> --url "/"
openclaw nodes canvas eval --node <id> --js "document.title"
openclaw nodes canvas snapshot --node <id>
```

Notes:

* `canvas.navigate` は **ローカル Canvas パス**、`http(s)` URL、`file://` URL を受け付けます。
* `"/"` を渡すと、Canvas はローカルの scaffold または `index.html` を表示します。


<div id="a2ui-in-canvas">
  ## Canvas における A2UI
</div>

A2UI は Gateway の Canvas ホストによってホストされ、Canvas パネル内にレンダリングされます。
Gateway が Canvas ホストとして公開されている場合、macOS アプリは初回起動時に
自動的に A2UI ホストページへ遷移します。

デフォルトの A2UI ホスト URL:

```
http://<gateway-host>:18793/__openclaw__/a2ui/
```


<div id="a2ui-commands-v08">
  ### A2UI コマンド (v0.8)
</div>

Canvas は現在、**A2UI v0.8** のサーバーからクライアントへのメッセージを受け付けます：

* `beginRendering`
* `surfaceUpdate`
* `dataModelUpdate`
* `deleteSurface`

`createSurface` (v0.9) には対応していません。

CLI の例：

```bash
cat > /tmp/a2ui-v0.8.jsonl <<'EOFA2'
{"surfaceUpdate":{"surfaceId":"main","components":[{"id":"root","component":{"Column":{"children":{"explicitList":["title","content"]}}}},{"id":"title","component":{"Text":{"text":{"literalString":"Canvas (A2UI v0.8)"},"usageHint":"h1"}}},{"id":"content","component":{"Text":{"text":{"literalString":"If you can read this, A2UI push works."},"usageHint":"body"}}}]}}
{"beginRendering":{"surfaceId":"main","root":"root"}}
EOFA2

openclaw nodes canvas a2ui push --jsonl /tmp/a2ui-v0.8.jsonl --node <id>
```

簡易スモークテスト：

```bash
openclaw nodes canvas a2ui push --node <id> --text "Hello from A2UI"
```


<div id="triggering-agent-runs-from-canvas">
  ## Canvas からエージェント実行をトリガーする
</div>

Canvas はディープリンク経由で新しいエージェント実行をトリガーできます:

* `openclaw://agent?...`

例（JavaScript の場合）:

```js
window.location.href = "openclaw://agent?message=Review%20this%20design";
```

有効なキーが指定されていない場合、アプリは確認を求めます。


<div id="security-notes">
  ## セキュリティに関する注意事項
</div>

- Canvas スキームはディレクトリトラバーサルを防止し、ファイルはセッションのルートディレクトリ配下に存在している必要があります。
- ローカル Canvas コンテンツはカスタムスキームを使用します（ループバックサーバーは不要です）。
- 外部の `http(s)` URL は、ユーザーが明示的に遷移した場合にのみ許可されます。