---
title: 位置情報コマンド
summary: "ノード向け位置情報コマンド（location.get）、権限モード、およびバックグラウンド時の動作"
read_when:
  - 位置情報ノードのサポートや権限UIを追加するとき
  - バックグラウンド位置情報取得とプッシュフローを設計するとき
---

<div id="location-command-nodes">
  # location コマンド（ノード向け）
</div>

<div id="tldr">
  ## TL;DR
</div>

- `location.get` はノードコマンドです（`node.invoke` 経由）。
- デフォルトでは Off。
- 設定はセレクターを使用します: Off / While Using / Always。
- 「Precise Location」は別のトグルです。

<div id="why-a-selector-not-just-a-switch">
  ## なぜ単なるスイッチではなくセレクタなのか
</div>

OS の権限は多層的です。アプリ内でセレクタは表示できますが、実際にどの権限が付与されるかを決めるのは OS 側です。

- iOS/macOS: ユーザーはシステムのプロンプトや「設定」で **使用中のみ** または **常に許可** を選択できます。アプリ側から権限の昇格をリクエストできますが、OS が「設定」アプリでの操作を要求する場合があります。
- Android: バックグラウンド位置情報は別の権限であり、Android 10 以降では多くの場合、設定画面でのフローが必要です。
- 高精度な位置情報も別の権限です（iOS 14+ の「正確な位置情報」、Android の「fine」対「coarse」など）。

UI 上のセレクタは、こちらが要求するモードを決めるだけで、実際の付与状態は OS の設定側で管理されます。

<div id="settings-model">
  ## 設定モデル
</div>

ノード（デバイス）ごとに:

- `location.enabledMode`: `off | whileUsing | always`
- `location.preciseEnabled`: bool

UI の動作:

- `whileUsing` を選択すると、フォアグラウンド権限を要求します。
- `always` を選択すると、まず `whileUsing` の権限を確保したうえでバックグラウンド権限を要求します（必要に応じてユーザーを設定画面に案内します）。
- OS が要求されたレベルを拒否した場合は、これまでに付与された中で最も高いレベルに戻し、その状態を表示します。

<div id="permissions-mapping-nodepermissions">
  ## 権限マッピング (node.permissions)
</div>

この項目は任意です。macOS ノードは権限マップ経由で `location` を報告しますが、iOS/Android では省略される場合があります。

<div id="command-locationget">
  ## コマンド: `location.get`
</div>

`node.invoke` で呼び出されます。

推奨パラメータ:

```json
{
  "timeoutMs": 10000,
  "maxAgeMs": 15000,
  "desiredAccuracy": "coarse|balanced|precise"
}
```

レスポンスペイロード：

```json
{
  "lat": 48.20849,
  "lon": 16.37208,
  "accuracyMeters": 12.5,
  "altitudeMeters": 182.0,
  "speedMps": 0.0,
  "headingDeg": 270.0,
  "timestamp": "2026-01-03T12:34:56.000Z",
  "isPrecise": true,
  "source": "gps|wifi|cell|unknown"
}
```

Errors (stable codes):

* `LOCATION_DISABLED`: セレクターがオフになっています。
* `LOCATION_PERMISSION_REQUIRED`: 要求されたモードに必要な権限がありません。
* `LOCATION_BACKGROUND_UNAVAILABLE`: アプリがバックグラウンドにありますが、「使用中のみ」の権限しか付与されていません。
* `LOCATION_TIMEOUT`: 所定時間内に位置を特定できませんでした。
* `LOCATION_UNAVAILABLE`: システム障害 / 利用可能なプロバイダーがありません。


<div id="background-behavior-future">
  ## バックグラウンド動作（将来予定）
</div>

目的：次の条件をすべて満たす場合に限り、ノードがバックグラウンド状態でもモデルが位置情報を要求できるようにする。

- ユーザーが **Always** を選択している。
- OS がバックグラウンドでの位置情報取得を許可している。
- アプリが位置情報取得のためにバックグラウンド実行を許可されている（iOS のバックグラウンドモード / Android のフォアグラウンドサービスまたは特別な許可）。

プッシュトリガーのフロー（将来予定）:

1) Gateway がノードにプッシュを送信する（サイレントプッシュまたは FCM データ）。
2) ノードが短時間復帰し、デバイスに位置情報を要求する。
3) ノードがペイロードを Gateway に転送する。

補足:

- iOS: 「Always」権限とバックグラウンド位置情報モードが必要。サイレントプッシュはスロットルされる可能性があり、断続的な失敗が発生することを想定する。
- Android: バックグラウンド位置情報にはフォアグラウンドサービスが必要となる場合がある。これがない場合は拒否されることを想定する。

<div id="modeltooling-integration">
  ## モデル／ツール連携
</div>

- ツールの提供範囲: `nodes` ツールに `location_get` アクションが追加されます（ノードが必要）。
- CLI: `openclaw nodes location get --node <id>`。
- エージェント向けガイドライン: ユーザーが位置情報を有効にし、そのスコープを理解している場合にのみ呼び出してください。

<div id="ux-copy-suggested">
  ## UX コピー（推奨）
</div>

- Off: 「位置情報の共有は無効です。」
- While Using: 「OpenClaw 使用中のみ。」
- Always: 「バックグラウンドでの位置情報利用を許可します（システムの権限が必要です）。」
- Precise: 「正確な GPS 位置情報を使用します。オフにするとおおよその位置のみを共有します。」