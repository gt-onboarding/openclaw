---
title: Gateway ロック
summary: "WebSocket リスナーのバインドを利用した Gateway のシングルトンガード機構"
read_when:
  - Gateway プロセスの実行またはデバッグ時
  - 単一インスタンス強制の仕組みを調査しているとき
---

<div id="gateway-lock">
  # Gateway ロック
</div>

最終更新: 2025年12月11日

<div id="why">
  ## 理由
</div>

- 同一ホスト上の同一ベースポートにつき、動作する Gateway インスタンスは 1 つだけに制限する。追加の Gateway は分離されたプロファイルと一意のポートを使用しなければならない。
- クラッシュや SIGKILL が発生しても、古いロックファイルを残さずに復旧できるようにする。
- 制御ポートがすでに占有されている場合は、明確なエラーを出して即座に失敗させる。

<div id="mechanism">
  ## メカニズム
</div>

- Gateway は起動時に、排他的な TCP リスナーを使って WebSocket リスナー（デフォルト `ws://127.0.0.1:18789`）を即座にバインドします。
- バインドが `EADDRINUSE` で失敗した場合、起動処理は `GatewayLockError("another gateway instance is already listening on ws://127.0.0.1:<port>")` をスローします。
- OS は、クラッシュや SIGKILL を含むあらゆるプロセス終了時にリスナーを自動的に解放するため、別途ロックファイルやクリーンアップ手順は不要です。
- シャットダウン時には、Gateway は WebSocket サーバーと基盤となる HTTP サーバーをクローズして、ポートを速やかに解放します。

<div id="error-surface">
  ## エラーの出方
</div>

- ほかのプロセスがポートを占有している場合、起動時に `GatewayLockError("another gateway instance is already listening on ws://127.0.0.1:<port>")` がスローされます。
- それ以外のバインド失敗は、`GatewayLockError("failed to bind gateway socket on ws://127.0.0.1:<port>: …")` としてスローされます。

<div id="operational-notes">
  ## 運用上の注意
</div>

- ポートが*別の*プロセスによって使用中の場合も、発生するエラーは同じです。その場合はポートを解放するか、`openclaw gateway --port <port>` で別のポートを指定してください。
- macOS アプリ側でも、Gateway を起動する前に独自の軽量な PID ガードを実装していますが、実行時ロック自体は WebSocket のバインドによって保証されます。