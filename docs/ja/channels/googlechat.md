---
title: Googlechat
summary: "Google Chat アプリのサポート状況、機能、および設定"
read_when:
  - Google Chat チャネル機能に取り組んでいる場合
---

<div id="google-chat-chat-api">
  # Google Chat（Chat API）
</div>

ステータス：Google Chat API の Webhook（HTTP のみ）経由で DM とスペースに対応済み。

<div id="quick-setup-beginner">
  ## クイックセットアップ（初心者向け）
</div>

1. Google Cloud プロジェクトを作成し、**Google Chat API** を有効化します。
   * 次のページにアクセスします: [Google Chat API Credentials](https://console.cloud.google.com/apis/api/chat.googleapis.com/credentials)
   * まだ有効化されていない場合は API を有効化します。
2. **サービス アカウント** を作成します:
   * **Create Credentials** &gt; **Service Account** をクリックします。
   * 任意の名前を付けます（例: `openclaw-chat`）。
   * 権限は何も設定せず空のままにします（**Continue** を押します）。
   * アクセス権を持つプリンシパルも空のままにします（**Done** を押します）。
3. **JSON キー** を作成してダウンロードします:
   * サービス アカウントの一覧から、今作成したアカウントをクリックします。
   * **Keys** タブを開きます。
   * **Add Key** &gt; **Create new key** をクリックします。
   * **JSON** を選択し、**Create** を押します。
4. ダウンロードした JSON ファイルを Gateway ホスト上に保存します（例: `~/.openclaw/googlechat-service-account.json`）。
5. [Google Cloud Console Chat Configuration](https://console.cloud.google.com/apis/api/chat.googleapis.com/hangouts-chat) で Google Chat アプリを作成します:
   * **Application info** を入力します:
     * **App name**: （例: `OpenClaw`）
     * **Avatar URL**: （例: `https://openclaw.ai/logo.png`）
     * **Description**: （例: `Personal AI Assistant`）
   * **Interactive features** を有効化します。
   * **Functionality** の下で **Join spaces and group conversations** にチェックを入れます。
   * **Connection settings** の下で **HTTP endpoint URL** を選択します。
   * **Triggers** の下で **Use a common HTTP endpoint URL for all triggers** を選択し、Gateway のパブリック URL の末尾に `/googlechat` を付けたものを設定します。
     * *ヒント: `openclaw status` を実行すると Gateway のパブリック URL を確認できます。*
   * **Visibility** の下で **Make this Chat app available to specific people and groups in &lt;Your Domain&gt;** にチェックを入れます。
   * テキストボックスに自分のメールアドレス（例: `user@example.com`）を入力します。
   * 画面下部の **Save** をクリックします。
6. **アプリのステータスを有効化します**:
   * 保存後、**ページを再読み込み** します。
   * **App status** セクション（通常は保存後にページの上部または下部付近に表示されます）を探します。
   * ステータスを **Live - available to users** に変更します。
   * 再度 **Save** をクリックします。
7. サービス アカウントのパスと webhook audience を指定して OpenClaw を構成します:
   * 環境変数: `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE=/path/to/service-account.json`
   * または設定: `channels.googlechat.serviceAccountFile: "/path/to/service-account.json"`。
8. webhook audience の種類と値を設定します（Chat アプリの設定と一致させます）。
9. Gateway を起動します。Google Chat はあなたの webhook パスに対して POST リクエストを送信するようになります。

<div id="add-to-google-chat">
  ## Google Chat に追加する
</div>

Gateway が動作していて、メールアドレスが可視性リストに追加されていることを確認したら、次の手順を実行します:

1. [Google Chat](https://chat.google.com/) にアクセスします。
2. **Direct Messages** の横にある **+**（プラス）アイコンをクリックします。
3. 通常ユーザーを追加するときに使用する検索バーに、Google Cloud Console で設定した **App name** を入力します。
   * **Note**: このボットはプライベートアプリのため、&quot;Marketplace&quot; の閲覧リストには *表示されません*。名前で検索する必要があります。
4. 検索結果からボットを選択します。
5. **Add** または **Chat** をクリックして、1:1 の会話を開始します。
6. 「Hello」と送信してアシスタントを起動します。

<div id="public-url-webhook-only">
  ## 公開URL（Webhook専用）
</div>

Google Chat の Webhook には、インターネットから到達可能な HTTPS エンドポイントが必要です。セキュリティのため、インターネットに公開するのは **`/googlechat` パスのみ** にしてください。OpenClaw のダッシュボードやその他の機微なエンドポイントは、プライベートネットワーク内にとどめておいてください。

<div id="option-a-tailscale-funnel-recommended">
  ### オプション A: Tailscale Funnel（推奨）
</div>

プライベートなダッシュボードには Tailscale Serve を、公開用の webhook パスには Funnel を使用します。これにより `/` はプライベートのままにしつつ、`/googlechat` のみを公開できます。

1. **Gateway がバインドされているアドレスを確認する:**
   ```bash
   ss -tlnp | grep 18789
   ```
   IP アドレス（例: `127.0.0.1`、`0.0.0.0`、または `100.x.x.x` のような Tailscale IP）を控えておきます。

2. **ダッシュボードを tailnet 内限定で公開する（ポート 8443）:**
   ```bash
   # localhost（127.0.0.1 または 0.0.0.0）にバインドされている場合:
   tailscale serve --bg --https 8443 http://127.0.0.1:18789

   # Tailscale IP のみにバインドされている場合（例: 100.106.161.80）:
   tailscale serve --bg --https 8443 http://100.106.161.80:18789
   ```

3. **webhook パスのみをパブリックに公開する:**
   ```bash
   # localhost（127.0.0.1 または 0.0.0.0）にバインドされている場合:
   tailscale funnel --bg --set-path /googlechat http://127.0.0.1:18789/googlechat

   # Tailscale IP のみにバインドされている場合（例: 100.106.161.80）:
   tailscale funnel --bg --set-path /googlechat http://100.106.161.80:18789/googlechat
   ```

4. **Funnel アクセス用にノードを承認する:**
   プロンプトが表示されたら、出力に示される認可 URL にアクセスし、tailnet ポリシーでこのノードに対して Funnel を有効化します。

5. **設定を確認する:**
   ```bash
   tailscale serve status
   tailscale funnel status
   ```

パブリック webhook URL は次のようになります:
`https://<node-name>.<tailnet>.ts.net/googlechat`

プライベートなダッシュボードは引き続き tailnet 限定です:
`https://<node-name>.<tailnet>.ts.net:8443/`

Google Chat アプリの設定には、パブリック URL（`:8443` なし）を使用してください。

> 注意: この設定は再起動後も維持されます。後で削除する場合は、`tailscale funnel reset` と `tailscale serve reset` を実行してください。

<div id="option-b-reverse-proxy-caddy">
  ### オプションB: リバースプロキシ (Caddy)
</div>

Caddy のようなリバースプロキシを使用する場合は、特定のパスだけをプロキシ対象にしてください:

```caddy
your-domain.com {
    reverse_proxy /googlechat* localhost:18789
}
```

この設定では、`your-domain.com/` へのリクエストはすべて無視されるか 404 を返し、その一方で `your-domain.com/googlechat` へのリクエストだけが安全に OpenClaw にルーティングされます。

<div id="option-c-cloudflare-tunnel">
  ### オプション C: Cloudflare Tunnel
</div>

トンネルの ingress ルールを設定し、Webhook のパスのみがルーティングされるようにします：

* **パス**: `/googlechat` -&gt; `http://localhost:18789/googlechat`
* **デフォルトルール**: HTTP 404（Not Found）

<div id="how-it-works">
  ## 仕組み
</div>

1. Google Chat は webhook の POST を Gateway に送信します。各リクエストには `Authorization: Bearer <token>` ヘッダーが含まれます。
2. OpenClaw は設定済みの `audienceType` と `audience` に対してトークンを検証します:
   * `audienceType: "app-url"` → audience はあなたの HTTPS webhook URL です。
   * `audienceType: "project-number"` → audience は Cloud プロジェクト番号です。
3. メッセージはスペースごとにルーティングされます:
   * DM はセッションキー `agent:<agentId>:googlechat:dm:<spaceId>` を使用します。
   * スペースはセッションキー `agent:<agentId>:googlechat:group:<spaceId>` を使用します。
4. DM へのアクセスはデフォルトでペアリングです。不明な送信者にはペアリングコードが送信されます。次のコマンドで承認します:
   * `openclaw pairing approve googlechat <code>`
5. グループスペースはデフォルトで @ メンションが必要です。メンション検出にアプリのユーザー名が必要な場合は `botUser` を使用します。

<div id="targets">
  ## ターゲット
</div>

配信および許可リストには、次の識別子を使用します。

* ダイレクトメッセージ: `users/<userId>` または `users/<email>`（メールアドレスも指定できます）。
* スペース: `spaces/<spaceId>`。

<div id="config-highlights">
  ## 設定の要点
</div>

```json5
{
  channels: {
    "googlechat": {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      audienceType: "app-url",
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890", // オプション; メンション検出を支援
      dm: {
        policy: "pairing",
        allowFrom: ["users/1234567890", "name@example.com"]
      },
      groupPolicy: "allowlist",
      groups: {
        "spaces/AAAA": {
          allow: true,
          requireMention: true,
          users: ["users/1234567890"],
          systemPrompt: "Short answers only."
        }
      },
      actions: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20
    }
  }
}
```

Notes:

* サービス アカウントの認証情報は、`serviceAccount`（JSON 文字列）でインライン指定することもできます。
* `webhookPath` が設定されていない場合、Webhook のデフォルトパスは `/googlechat` です。
* リアクションは、`actions.reactions` が有効な場合、`reactions` ツールおよび `channels action` から利用できます。
* `typingIndicator` は、`none`、`message`（デフォルト）、`reaction` をサポートします（`reaction` にはユーザー OAuth が必要です）。
* 添付ファイルは Chat API 経由でダウンロードされ、メディアパイプラインに保存されます（サイズの上限は `mediaMaxMb` で制御されます）。

<div id="troubleshooting">
  ## トラブルシューティング
</div>

<div id="405-method-not-allowed">
  ### 405 Method Not Allowed
</div>

Google Cloud Logs Explorer に次のようなエラーが表示される場合があります：

```
status code: 405, reason phrase: HTTP error response: HTTP/1.1 405 Method Not Allowed
```

これは webhook ハンドラが登録されていないことを意味します。よくある原因は次のとおりです。

1. **チャネルが設定されていない**: 設定ファイルに `channels.googlechat` セクションがありません。次のコマンドで確認します:
   ```bash
   openclaw config get channels.googlechat
   ```
   もし「Config path not found」と表示された場合は、設定を追加してください（[Config highlights](#config-highlights) を参照）。

2. **プラグインが有効化されていない**: プラグインの状態を確認します:
   ```bash
   openclaw plugins list | grep googlechat
   ```
   もし「disabled」と表示された場合は、設定ファイルに `plugins.entries.googlechat.enabled: true` を追加してください。

3. **Gateway が再起動されていない**: 設定を追加した後、Gateway を再起動してください:
   ```bash
   openclaw gateway restart
   ```

チャネルが稼働していることを確認してください:

```bash
openclaw channels status
# 表示例: Google Chat default: enabled, configured, ...
```

<div id="other-issues">
  ### その他の問題
</div>

* `openclaw channels status --probe` を実行し、認証エラーや audience 設定の不足・誤りがないか確認します。
* メッセージが届かない場合は、Chat アプリの webhook URL とイベントサブスクリプションを確認します。
* メンション必須のゲート設定で返信がブロックされている場合は、`botUser` をアプリの user リソース名に設定し、`requireMention` を確認します。
* テストメッセージを送信しながら `openclaw logs --follow` を使用し、リクエストが Gateway に到達しているか確認します。

関連ドキュメント:

* [Gateway の設定](/ja/gateway/configuration)
* [セキュリティ](/ja/gateway/security)
* [Reactions](/ja/tools/reactions)